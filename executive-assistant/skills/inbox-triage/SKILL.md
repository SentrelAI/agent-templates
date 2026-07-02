---
name: inbox-triage
description: Use to triage the principal's inbox — works with EITHER Gmail or Outlook/Microsoft 365. Pull threads, sort into needs-you / draft / delegate / archive, spot VIPs, and draft replies in the principal's voice (never sending), on whichever provider is connected. Covers both APIs, provider detection, and the per-provider draft gotchas.
---

# Inbox triage (Gmail **or** Outlook)

You keep {{principal_name}}'s inbox sorted through the **`request`** tool, on
whichever mail provider is connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"gmail"`** → base `https://gmail.googleapis.com`, prefix
  `/gmail/v1/users/me`.
- **`provider:"outlook"`** → base `https://graph.microsoft.com/v1.0`, prefix
  `/me`.
- Auth injected for both — never handle a token. Result `{ status, body }`.
  401/403 = that provider isn't connected; say so. **Detect which is connected
  and stay on it** for the session.

## Sort into four buckets (both providers)
Read each thread in full, then sort:
- **Needs {{principal_name}}** — a decision/approval/reply only they can give.
- **I can draft** — routine reply; draft in their voice, queue for approval.
- **Delegate** — belongs to someone else; note the owner, draft a forward.
- **FYI / archive** — newsletters, receipts, cold outreach, no-action CCs.

**VIPs ({{vips}}) always surface** — never archive, always in the brief.

---

## Gmail

Pull the queue → read the thread → label → draft:
```
// queue
request({ provider:"gmail", method:"GET", path:"/gmail/v1/users/me/messages",
  query:{ q:"in:inbox is:unread newer_than:3d", maxResults:40 } })
// thread (full history)
request({ provider:"gmail", method:"GET",
  path:"/gmail/v1/users/me/threads/<threadId>", query:{ format:"full" } })
// label (create via POST /labels; ids from GET /labels)
request({ provider:"gmail", method:"POST",
  path:"/gmail/v1/users/me/messages/<id>/modify",
  body:{ addLabelIds:["<needs-you|drafted|delegated|fyi>"],
         removeLabelIds:["UNREAD","INBOX"] } })   // drop INBOX only for FYI/archive
// draft (base64url RFC 2822 MIME with In-Reply-To/References)
request({ provider:"gmail", method:"POST", path:"/gmail/v1/users/me/drafts",
  body:{ message:{ raw:"<base64url MIME>", threadId:"<threadId>" } } })
```
Body text is base64url in the `text/plain` part; headers in `payload.headers`.

---

## Outlook / Microsoft 365 (Microsoft Graph)

```
// queue
request({ provider:"outlook", method:"GET", path:"/me/messages",
  query:{ $filter:"isRead eq false", $top:40,
          $select:"subject,from,receivedDateTime,conversationId,isRead" } })
// thread (by conversationId)
request({ provider:"outlook", method:"GET", path:"/me/messages",
  query:{ $filter:"conversationId eq '<conversationId>'",
          $orderby:"receivedDateTime asc", $select:"subject,from,body,receivedDateTime" } })
// categorize + read state (categories are the label equivalent)
request({ provider:"outlook", method:"PATCH", path:"/me/messages/<id>",
  body:{ categories:["Needs you"], isRead:true } })
// archive: move to a folder
request({ provider:"outlook", method:"POST", path:"/me/messages/<id>/move",
  body:{ destinationId:"archive" } })
// draft a reply (Graph builds + threads it; then set the body — no MIME)
request({ provider:"outlook", method:"POST", path:"/me/messages/<id>/createReply" })
request({ provider:"outlook", method:"PATCH", path:"/me/messages/<draftId>",
  body:{ body:{ contentType:"HTML", content:"<reply in the principal's voice>" } } })
```
`body.value[].body.content` is the message body directly (HTML/text) — no decode.

---

## Draft in the principal's voice (never send)
Match {{principal_name}}'s tone (learn it from their sent mail + the operating
manual). On **both** providers you create a *draft* and stop — never Gmail
`messages/send` or Outlook `/send`. A human approving is the only thing that
sends.

## Rules
1. **Triage, don't just list** — every thread lands in one of the four buckets.
2. **VIPs never get buried or archived** — always surfaced.
3. **Draft, never send** — replies wait for {{principal_name}}'s approval.
4. **Protect attention** — aggressively archive true noise; only surface what
   needs them or touches a priority ({{priorities}}).
5. **One provider per session** — detect Gmail vs. Outlook and stay on it.
6. **Match their voice** — a draft that doesn't sound like {{principal_name}}
   makes more work than it saves.
