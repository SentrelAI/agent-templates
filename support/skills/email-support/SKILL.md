---
name: email-support
description: Use when triaging the support inbox or drafting a reply — listing/searching support email, reading a full thread, applying category labels, and creating a reply DRAFT (never sending). Covers the Gmail REST API through the mcp__apps__request proxy, the search query syntax, label management, and the base64url MIME gotcha for drafts.
---

# Email support (Gmail)

You work the support inbox by calling the Gmail REST API through the
**`request`** tool (server `apps`):

```
request({ provider:"gmail", method, path, query?, body? })
```

- **Base is `https://gmail.googleapis.com`.** Auth is injected — NEVER ask for,
  include, or echo a token.
- The mailbox path prefix is always `/gmail/v1/users/me`.
- The tool result is `{ status, body }`. Read `body` for the JSON payload.
- A 401/403 means the account isn't connected (or scope is missing) — say so
  and ask the user to connect Gmail at /integrations. Don't guess endpoints.

## Triage: find what needs attention

Use Gmail's search `q` syntax to pull the queue:

```
request({ provider:"gmail", method:"GET",
  path:"/gmail/v1/users/me/messages",
  query:{ q:"in:inbox is:unread -label:handled", maxResults:25 } })
// → body.messages = [{ id, threadId }, ...]  (ids only — you must fetch each)
```

Useful `q` filters: `is:unread`, `newer_than:2d`, `from:someone@x.com`,
`subject:refund`, `-label:handled`, `label:escalated`. Combine them.

## Read the full thread (always, before replying)

Fetch the **thread**, not just one message, so you have the whole history:

```
request({ provider:"gmail", method:"GET",
  path:"/gmail/v1/users/me/threads/<threadId>", query:{ format:"full" } })
// → body.messages[] each with .payload.headers (From/Subject/Date) and the body
```

The body text is base64url-encoded in `payload.body.data` (or in
`payload.parts[]` for multipart — prefer the `text/plain` part). Decode it to
read. Pull `From`, `Subject`, and `Message-ID`/`References` from
`payload.headers` — you need `Message-ID` + `References` to thread a reply
correctly.

## Label (categorize the queue)

Labels are how the queue stays organized. List them once to get their ids:

```
request({ provider:"gmail", method:"GET", path:"/gmail/v1/users/me/labels" })
// → body.labels = [{ id, name }, ...]
```

If a category label (e.g. `bug`, `billing`, `handled`, `escalated`) doesn't
exist, create it:

```
request({ provider:"gmail", method:"POST", path:"/gmail/v1/users/me/labels",
  body:{ name:"escalated", labelListVisibility:"labelShow",
         messageListVisibility:"show" } })
```

Apply/remove labels on a message (use the label **id**, not the name):

```
request({ provider:"gmail", method:"POST",
  path:"/gmail/v1/users/me/messages/<messageId>/modify",
  body:{ addLabelIds:["<bugLabelId>"], removeLabelIds:["UNREAD"] } })
```

`UNREAD`, `INBOX`, `STARRED` are built-in system label ids.

## Draft a reply (NEVER send)

You create a **draft** on the thread and submit it for approval. You do not
call `messages/send` — sending a customer-facing reply is gated on
`send_customer_reply` and happens only after a human approves.

The message must be RFC 2822 MIME, **base64url-encoded** (`+`→`-`, `/`→`_`, no
padding newlines) in the `raw` field, and carry `threadId` so it stays on the
thread:

```
// 1) build the MIME string (headers + blank line + body):
//    To: customer@x.com
//    Subject: Re: <original subject>
//    In-Reply-To: <original Message-ID>
//    References: <original References + Message-ID>
//    Content-Type: text/plain; charset="UTF-8"
//
//    <your reply text>
// 2) base64url-encode it → <raw>
request({ provider:"gmail", method:"POST",
  path:"/gmail/v1/users/me/drafts",
  body:{ message:{ raw:"<base64url MIME>", threadId:"<threadId>" } } })
// → body.id is the draft id; body.message.id the message id
```

Keeping `In-Reply-To` + `References` correct is what makes Gmail (and the
customer's client) show it as a proper reply instead of a new email.

## Rules

1. **Read the whole thread first.** Never reply off the last message alone.
2. **Draft, never send.** Create a draft and submit for approval. The only
   thing that sends is a human clicking approve.
3. **Label everything you touch** — category + `handled`/`escalated` — so the
   queue reflects reality and the triage search doesn't re-surface it.
4. **Thread correctly.** Carry `threadId` + `In-Reply-To`/`References`, and
   prefix the subject with `Re:` if it isn't already.
5. **Verify identity before anything sensitive.** If the ask involves account
   data, don't confirm details to an address that doesn't match the account on
   file — escalate instead (see the policy).
