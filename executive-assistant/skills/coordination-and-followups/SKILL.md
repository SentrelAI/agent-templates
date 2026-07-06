---
name: coordination-and-followups
description: Use to coordinate with the principal's team and make sure nothing slips — tracking every open commitment with an owner and due date, nudging on time, and posting the daily brief/summary to Slack. Uses the native slack.post tool (the agent's Slack channel); follow-ups are held as tasks/reminders.
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

## Slack — coordinate, nudge, deliver
Slack is a **native channel** for this agent (installed once via "Install in
Slack" on the Channels tab). You post with the built-in **`slack.post`** tool —
no token, no channel lookup, no apps-proxy call:

```
slack.post({ text, channel?, thread_ts? })
```

- **`text`** — the message; Slack **mrkdwn** works (`*bold*`, `<url|label>`,
  `<@U…>` to @mention someone, `\n` for line breaks).
- **`channel`** — omit it; posts to the agent's **own bound channel** (the
  team/coordination channel). Only pass `channel` to reply in a different
  channel where {{principal_name}} was mentioned; `thread_ts` to reply in-thread.
- The tool posts on success and throws on failure (if Slack isn't installed for
  this agent, you'll get a clear error → tell the user to install it on the
  Channels tab).

**Nudge** — post to the team channel and @mention the owner (the native channel
posts to a shared channel, so a nudge is a mention, not a private DM):
```
slack.post({ text: "<@U123> quick nudge for {{principal_name}}: the deck for Thursday's board prep — still on track for tomorrow AM?" })
```

**Deliver the brief / summary** — one scannable message to the channel:
```
slack.post({ text: "📋 *EOD* — 3 replies awaiting your OK · 2 follow-ups moved · 1 overdue (vendor contract) · tomorrow starts 9:30 (board prep, brief attached)" })
```

## Rules

1. **Everything gets an owner + a due date.** An untracked commitment is a
   dropped one.
2. **Nudge before it's due, not after.** A timely reminder prevents the miss; a
   late one just documents it.
3. **@mention the owner** in the channel so the nudge reaches the right person;
   keep it to one message, not a thread of pings.
4. **Internal only over Slack.** Coordinating with the team is fine; anything to
   an outside party is outbound → draft for approval (inbox-triage skill).
5. **Default to your own channel.** Omit `channel` unless you're replying to a
   specific thread where {{principal_name}} was mentioned.
