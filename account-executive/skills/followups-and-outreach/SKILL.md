---
name: followups-and-outreach
description: Use to read deal threads and draft follow-ups, call recaps, and next-step emails that actually advance the deal — works with EITHER Gmail or Outlook/Microsoft 365, never sending. Covers both APIs (search, thread reads, draft creation with correct threading) and what makes a follow-up land vs. get ignored.
---

# Follow-ups & outreach (Gmail **or** Outlook)

You keep deals moving by reading the thread and drafting the right next touch —
always as a draft for {{rep_name}} to approve, on whichever mail provider is
connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"gmail"`** → base `https://gmail.googleapis.com`, prefix
  `/gmail/v1/users/me`.
- **`provider:"outlook"`** → base `https://graph.microsoft.com/v1.0`, prefix
  `/me`.
- Auth injected for both. Result `{ status, body }`. Detect which is connected
  and stay on it.

## Gmail path

## Find the deal thread

```
request({ provider:"gmail", method:"GET", path:"/gmail/v1/users/me/messages",
  query:{ q:"from:prospect@x.com OR to:prospect@x.com newer_than:30d", maxResults:10 } })
```

Read the **thread** in full (`GET /gmail/v1/users/me/threads/<threadId>?format=full`)
so the follow-up references the real last exchange. Pull `Message-ID` +
`References` from `payload.headers` for threading; body text is base64url in
the `text/plain` part.

## Draft a follow-up that advances the deal (never send)

A good AE follow-up: reference something specific from the last conversation →
add value (a resource, an answer, a relevant proof point) → propose one clear
next step. No "just checking in", no guilt, no fake urgency.

Create a **draft** on the thread — you never call `messages/send`; sending is
gated on {{rep_name}}'s approval:

```
// build RFC 2822 MIME (To / Subject Re: / In-Reply-To / References / body),
// base64url-encode → <raw>
request({ provider:"gmail", method:"POST", path:"/gmail/v1/users/me/drafts",
  body:{ message:{ raw:"<base64url MIME>", threadId:"<threadId>" } } })
```

Carry `In-Reply-To` + `References` so it threads as a reply.

## Outlook / Microsoft 365 (Microsoft Graph)

Same idea, Graph endpoints:
- **Find the thread:** `GET /me/messages` ·
  `query:{ $search:"\"prospect@x.com\"", $top:10, $select:"subject,from,conversationId,receivedDateTime" }`,
  then read it by `$filter:"conversationId eq '<id>'"` (`$orderby:"receivedDateTime asc"`).
  `body.value[].body.content` is the text directly — no base64 decode.
- **Draft a reply (never send):** `POST /me/messages/<id>/createReply` →
  `body.id` (a threaded draft in Drafts), then
  `PATCH /me/messages/<draftId>` · `body:{ body:{ contentType:"HTML", content:"<reply>" } }`.
  Leave it as a draft — do **not** POST `/send`.

## Types of touch (pick the right one)
- **Post-call recap** — summarize what was agreed, restate the next step + date,
  attach anything promised. Send within a day of the call.
- **Re-engagement** — for a quiet thread, lead with a *reason to reply now* (new
  info, a relevant update), not "circling back."
- **Next-step nudge** — a specific ask tied to the agreed plan ("ready to get
  the security review on the calendar — does Tue or Thu work?").

## Rules

1. **Read the thread first.** A follow-up that ignores the last message reads
   like a blast and kills trust.
2. **Every follow-up advances the deal.** Reference specifics, add value, ask
   for one concrete next step. Never "just checking in."
3. **Draft, never send.** {{rep_name}} approves anything a prospect sees.
4. **Thread correctly** (`Re:` + `In-Reply-To`/`References`) and log the touch
   (and any new commitment → a CRM task) via the crm-hubspot skill.
5. **No commercials.** Never quote price, discount, or terms in a draft — flag
   those to {{rep_name}} instead.
