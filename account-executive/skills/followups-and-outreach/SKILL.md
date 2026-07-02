---
name: followups-and-outreach
description: Use to read deal threads and draft follow-ups, call recaps, and next-step emails that actually advance the deal — never sending. Covers the Gmail REST API through the mcp__apps__request proxy (search, thread reads, draft creation with correct threading) and what makes a follow-up land vs. get ignored.
---

# Follow-ups & outreach (Gmail)

You keep deals moving by reading the thread and drafting the right next touch —
always as a draft for {{rep_name}} to approve.

```
request({ provider:"gmail", method, path, query?, body? })
```

- **Base is `https://gmail.googleapis.com`**, mailbox prefix
  `/gmail/v1/users/me`. Auth injected. Result `{ status, body }`.

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
