---
name: email-support
description: Use when triaging the support inbox or drafting a reply — works with EITHER Gmail or Outlook/Microsoft 365. Covers listing/searching support email, reading a full thread, categorizing (labels/categories), and creating a reply DRAFT (never sending) on whichever provider is connected. Includes provider detection and the per-provider gotchas (Gmail base64url MIME vs. Outlook createReply drafts).
---

# Email support (Gmail **or** Outlook)

You work the support inbox through the **`request`** tool. This skill supports
two providers — use whichever the workspace connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"gmail"`** → base `https://gmail.googleapis.com`, mailbox prefix
  `/gmail/v1/users/me`.
- **`provider:"outlook"`** → base `https://graph.microsoft.com/v1.0` (Microsoft
  Graph), mailbox prefix `/me`.
- Auth is injected for both — NEVER ask for, include, or echo a token. Result is
  `{ status, body }`. A 401/403 means that provider isn't connected — say so and
  point the user to /integrations.

**Pick the provider once:** whichever of `gmail` / `outlook` is connected is the
one you use for the whole session. Don't mix. If both are connected, prefer the
one the support address ({{support_email}}) belongs to.

---

## Gmail

### Triage — find what needs attention
```
request({ provider:"gmail", method:"GET", path:"/gmail/v1/users/me/messages",
  query:{ q:"in:inbox is:unread -label:handled", maxResults:25 } })
// → body.messages = [{ id, threadId }] (ids only — fetch each)
```
`q` syntax: `is:unread`, `newer_than:2d`, `from:x@y.com`, `subject:refund`,
`-label:handled`.

### Read the full thread
```
request({ provider:"gmail", method:"GET",
  path:"/gmail/v1/users/me/threads/<threadId>", query:{ format:"full" } })
```
Body text is base64url in `payload.body.data` / the `text/plain` part. Pull
`From`, `Subject`, `Message-ID`, `References` from `payload.headers`.

### Label (categorize)
List (`GET /gmail/v1/users/me/labels`), create missing
(`POST /gmail/v1/users/me/labels {name}`), apply by id:
```
request({ provider:"gmail", method:"POST",
  path:"/gmail/v1/users/me/messages/<id>/modify",
  body:{ addLabelIds:["<labelId>"], removeLabelIds:["UNREAD"] } })
```

### Draft a reply (NEVER send)
Build RFC 2822 MIME (`To` / `Subject: Re:` / `In-Reply-To` / `References` /
blank line / body), **base64url-encode** it, and create a draft on the thread:
```
request({ provider:"gmail", method:"POST", path:"/gmail/v1/users/me/drafts",
  body:{ message:{ raw:"<base64url MIME>", threadId:"<threadId>" } } })
```
Never call `messages/send` — sending is gated on approval.

---

## Outlook / Microsoft 365 (Microsoft Graph)

### Triage — find what needs attention
```
request({ provider:"outlook", method:"GET", path:"/me/messages",
  query:{ $filter:"isRead eq false", $top:25,
          $select:"subject,from,receivedDateTime,conversationId,isRead" } })
// → body.value = [{ id, conversationId, subject, from, ... }]  (full objects)
```
Full-text search instead of a filter: `$search:"\"refund\""` (quotes required).

### Read the full thread (conversation)
Graph groups a thread by `conversationId`:
```
request({ provider:"outlook", method:"GET", path:"/me/messages",
  query:{ $filter:"conversationId eq '<conversationId>'",
          $orderby:"receivedDateTime asc",
          $select:"subject,from,toRecipients,body,receivedDateTime" } })
```
`body.value[].body.content` is the message body (HTML or text per
`body.contentType`) — no base64 decode needed.

### Categorize (labels ≈ categories, or folders)
Outlook uses **categories** (colored tags) and **folders**. Tag a message:
```
request({ provider:"outlook", method:"PATCH", path:"/me/messages/<id>",
  body:{ categories:["Escalated"], isRead:true } })
```
(Master list: `GET /me/outlook/masterCategories`; create with `POST`.) To file
it, move to a folder: `POST /me/messages/<id>/move { destinationId:"<folderId>" }`
(well-known ids like `archive` work).

### Draft a reply (NEVER send) — cleaner than Gmail
Graph creates the draft *for you* on the thread, then you fill the body — no MIME
assembly:
```
// 1) create the reply draft (lands in Drafts, correctly threaded):
request({ provider:"outlook", method:"POST",
  path:"/me/messages/<id>/createReply" })          // → body.id = draft id
// 2) set the body:
request({ provider:"outlook", method:"PATCH", path:"/me/messages/<draftId>",
  body:{ body:{ contentType:"HTML", content:"<your reply>" } } })
```
Leave it as a draft — do **not** POST `/me/messages/<draftId>/send`. Use
`createReplyAll` when the thread has multiple recipients who should stay looped.

---

## Rules (both providers)

1. **Read the whole thread first** — never reply off the last message alone.
2. **Draft, never send.** Gmail: create a draft, don't `send`. Outlook: create
   the reply draft + set body, don't `/send`. A human approving is the only
   thing that sends.
3. **Categorize everything you touch** (Gmail labels / Outlook categories) so the
   queue reflects reality — `handled` / `escalated` + the category.
4. **Thread correctly.** Gmail: carry `threadId` + `In-Reply-To`/`References`.
   Outlook: `createReply` handles threading for you.
5. **One provider per session** — detect which is connected and stay on it.
6. **Verify identity before anything sensitive** — don't confirm account details
   to an address that doesn't match the account on file; escalate (see policy).
