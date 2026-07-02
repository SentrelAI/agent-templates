---
name: coordination-and-followups
description: Use to coordinate with the principal's team and make sure nothing slips — tracking every open commitment with an owner and due date, nudging the right person on time, and posting the daily brief/summary to Slack. Covers the Slack Web API through the mcp__apps__request proxy; follow-ups are held as tasks/reminders.
---

# Coordination & follow-ups

Two jobs: keep {{principal_name}}'s commitments from going cold, and coordinate
with the team over Slack.

## Track every commitment
Whenever {{principal_name}} (or someone to them) says "I'll send X", "let's
reconnect next week", "can you get me Y" — capture it as a tracked follow-up
with **owner + due date + next step**. Use the built-in task/reminder capability
so it resurfaces on time. The end-of-day sweep and morning brief both read this
list, so nothing has to live in {{principal_name}}'s head.

For each open item, decide who moves it:
- **{{principal_name}} owes it** → draft the thing (email/inbox-triage skill) or
  remind them, on the due date.
- **Someone else owes it** → nudge that person (Slack, below) a touch before
  it's due.
- **Waiting on an outsider** → send/queue a gentle chase.

## Slack — coordinate & nudge
```
request({ provider:"slack", method:"POST", path:"/<method>", body:{ ... } })
```

- **Base `https://slack.com/api`**; methods are path segments. Auth injected.
- Slack returns HTTP 200 even on errors — **check `body.ok`**, read
  `body.error` (`channel_not_found`, `not_in_channel`, `user_not_found`).

Resolve a channel id once (`POST /conversations.list` → match `.name`). To nudge
a teammate directly, open a DM then post to it:

```
request({ provider:"slack", method:"POST", path:"/conversations.open",
  body:{ users:"<memberId>" } })            // → body.channel.id (a DM channel)
request({ provider:"slack", method:"POST", path:"/chat.postMessage",
  body:{ channel:"<DM or channel id>",
         text:"Quick nudge for {{principal_name}}: the deck for Thursday's board prep — still on track for tomorrow AM?" } })
```

Get a member id from an email with `POST /users.lookupByEmail`
(`body:{ email }` → `body.user.id`).

## Post the brief / summary
Deliver the morning brief or end-of-day summary to {{principal_name}}'s DM or a
private channel — scannable, one message:

```
"📋 EOD: 3 replies awaiting your OK · 2 follow-ups moved · 1 overdue (vendor contract) · tomorrow starts 9:30 (board prep, brief attached)"
```

## Rules

1. **Everything gets an owner + a due date.** An untracked commitment is a
   dropped one.
2. **Nudge before it's due, not after.** A timely reminder prevents the miss; a
   late one just documents it.
3. **Right person, right nudge.** DM the individual who owes the thing; don't
   broadcast to a channel for a one-person ask.
4. **Internal only over Slack.** Coordinating with the team is fine; anything to
   an outside party is outbound → draft for approval.
5. **Check `body.ok`.** Handle `not_in_channel` / `user_not_found` explicitly
   instead of assuming the message landed.
