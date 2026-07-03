---
name: followup-tracking
description: Use to close the loop after meetings — drafting a same-day recap (decided / action items / open questions), sending it after approval, and chasing action items to done (nudge owners on time, flag stuck ones). Covers mail (Gmail or Outlook) through the mcp__apps__request proxy plus Slack (native slack.post) for nudges.
---

# Follow-up & tracking

A meeting's value is realized *after* it. Two jobs: get a crisp recap out
same-day, and make sure every action item actually happens.

## Draft the recap (send only after approval)
Structure: **Decided · Action items (owner + date) · Open questions.** Scannable,
same-day. Draft it on the mail provider that's connected:
- **Gmail:** base64url RFC 2822 MIME → `POST /gmail/v1/users/me/drafts { message:{ raw, threadId } }` (base `https://gmail.googleapis.com`).
- **Outlook:** `POST /me/messages` (a draft) or `createReply` on the invite thread
  → `PATCH /me/messages/<draftId> { body }` (base `https://graph.microsoft.com/v1.0`).
Don't send — a human approves anything attendees see. (For an internal team, a
Slack recap via `slack.post` is fine to post.)

Recap shape:
```
Recap — Product sync (7/6)
Decided: ship v2 on 7/20; cut export to hit the date.
Action items:
  • Dana — pull churn cohort by 7/11
  • Milo — update the roadmap doc by 7/9
Open questions: pricing for the SMB tier (owner: Ace, next week).
```

## Chase action items to done
Read the action-item tracker (notes-and-actions skill). For each open item:
- **Due soon / overdue** → nudge the owner (Slack, native `slack.post`):
  `slack.post({ text: "<@U123> reminder: you own *pull churn cohort* from Tue's product sync — due 7/11. On track?" })`
- **Done** → mark it complete in the tracker.
- **Stuck** → flag it (to the owner + the meeting's lead) with what's blocking.
Never reassign someone's action item; nudge + surface.

## Rules
1. **Recap same-day** — decided / actions (owner+date) / open questions; draft,
   human sends to attendees.
2. **Nudge on time** — a reminder before the due date prevents the miss.
3. **Nudge, never reassign** — @mention the owner; don't move their item.
4. **Close the loop** — mark done, flag stuck; no action item silently dies.
5. **Internal nudges over Slack; attendee-facing recaps need approval.**
