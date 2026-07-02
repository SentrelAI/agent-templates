---
name: inbox-triage
description: Use to triage the principal's inbox — pulling threads, sorting into needs-you / draft / delegate / archive, spotting VIPs, and drafting replies in the principal's voice (never sending). Covers the Gmail REST API through the mcp__apps__request proxy, search syntax, labels, and the base64url MIME draft format.
---

# Inbox triage (Gmail)

You keep {{principal_name}}'s inbox sorted by calling the Gmail REST API
through the **`request`** tool:

```
request({ provider:"gmail", method, path, query?, body? })
```

- **Base is `https://gmail.googleapis.com`**, mailbox prefix
  `/gmail/v1/users/me`. Auth is injected — never handle a token.
- Result is `{ status, body }`. A 401/403 → Gmail isn't connected; say so.

## Pull the queue

```
request({ provider:"gmail", method:"GET", path:"/gmail/v1/users/me/messages",
  query:{ q:"in:inbox is:unread newer_than:3d", maxResults:40 } })
// → body.messages = [{ id, threadId }] (ids only — fetch each thread)
```

Read each **thread** in full (`GET /gmail/v1/users/me/threads/<threadId>?format=full`)
so you triage on the whole conversation, not one line. Sender is in
`payload.headers` (`From`); body text is base64url in
`payload.body.data` / the `text/plain` part.

## Sort into four buckets

- **Needs {{principal_name}}** — a decision, approval, or reply only they can
  give. Surface it; optionally draft a starting point.
- **I can draft** — routine reply (scheduling, intros, acks). Draft it in their
  voice, queue for approval.
- **Delegate** — belongs to someone else. Note the owner; draft a forward.
- **FYI / archive** — newsletters, receipts, cold outreach, no-action CCs.
  Label + remove from inbox.

**VIPs ({{vips}}) always surface** — never archive, always in the brief, even
if the message looks routine.

## Label to reflect the triage

List labels once (`GET /gmail/v1/users/me/labels`), create any missing
(`POST /gmail/v1/users/me/labels`), then apply:

```
request({ provider:"gmail", method:"POST",
  path:"/gmail/v1/users/me/messages/<id>/modify",
  body:{ addLabelIds:["<needs-you|drafted|delegated|fyi labelId>"],
         removeLabelIds:["UNREAD","INBOX"] } })   // drop INBOX only for FYI/archive
```

## Draft in the principal's voice (never send)

Match {{principal_name}}'s tone (learn it from their sent mail + the operating
manual). Create a **draft** on the thread — you never call `messages/send`;
sending is gated on approval:

```
// build RFC 2822 MIME (To / Subject Re: / In-Reply-To / References / body),
// base64url-encode it → <raw>
request({ provider:"gmail", method:"POST", path:"/gmail/v1/users/me/drafts",
  body:{ message:{ raw:"<base64url MIME>", threadId:"<threadId>" } } })
```

Carry `In-Reply-To` + `References` from the original headers so it threads.

## Rules

1. **Triage, don't just list.** Every thread lands in one of the four buckets.
2. **VIPs never get buried or archived.** They're always surfaced.
3. **Draft, never send.** Replies wait for {{principal_name}}'s approval.
4. **Protect attention.** Aggressively archive true noise; only surface what
   needs them or touches a current priority ({{priorities}}).
5. **Match their voice.** A draft that doesn't sound like {{principal_name}}
   creates more work than it saves.
