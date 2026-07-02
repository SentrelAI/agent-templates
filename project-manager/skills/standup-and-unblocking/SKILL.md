---
name: standup-and-unblocking
description: Use to run the async standup and chase blockers over Slack — posting a concise standup, flagging blockers with an owner, and nudging the right person (via @mention) when something's stuck. Uses the native slack.post tool (the agent's Slack channel); nudges are mentions, never orders.
---

# Standup & unblocking (Slack)

You run the team's async standup and chase blockers over Slack. Slack is a
**native channel** for this agent (installed once via "Install in Slack" on the
Channels tab). You post with the built-in **`slack.post`** tool — no token, no
apps-proxy call:

```
slack.post({ text, channel?, thread_ts? })
```

- **`text`** — Slack mrkdwn works (`*bold*`, `<@U…>` to @mention, `<url|label>`,
  `\n`).
- **`channel`** — omit it; posts to the agent's own bound channel
  ({{team_channel}}). Only pass it to reply in another channel; `thread_ts` to
  thread.
- Posts on success, throws on failure (if Slack isn't installed for this agent,
  you'll get a clear error → tell the user to install it on the Channels tab).

## The async standup (blocker-first)
Pull the board (sprint-management skill), then post one scannable message:
```
slack.post({ text:
  "*Standup — {{workspace}}*\n" +
  "🟢 *Moving:* auth revamp (ENG-410) in review · onboarding polish (ENG-433)\n" +
  "🎯 *Today:* wrap billing migration; kick off ENG-440\n" +
  "🔴 *Blockers:*\n" +
  "  • ENG-421 billing migration — waiting on staging DB creds. <@U123> can you unblock today?\n" +
  "  • ENG-419 — no update in 3 days. <@U456> still on track?" })
```
Lead with blockers — they're what need action. Keep the green stuff to one line.

## Nudge an owner (mention, don't order)
When something's stuck or stale, @mention the owner with a specific, friendly
check-in — ask, don't command:
```
slack.post({ text: "<@U456> quick check — ENG-419 (search indexing) has been quiet 3 days and it's on the launch path. Still on track, or is something in the way?" })
```

## Escalate (when it warrants)
For a blocker stuck past the policy window or a date about to slip, post a clear
flag to the channel (and, per `pm-playbook.md`, ping the lead) with the impact +
options — not just "it's blocked."

## Rules
1. **Blocker-first.** Lead the standup with what's stuck + an owner; green work
   gets one line.
2. **@mention the owner** so the nudge reaches the right person — one message,
   not a thread of pings.
3. **Nudge, never order.** Ask if it's on track / what's in the way; never tell
   people what to do or move their work.
4. **Default to your own channel.** Omit `channel` unless replying in a specific
   thread.
5. **Internal only.** Standups/nudges are for the team; anything outward is a
   human's call.
