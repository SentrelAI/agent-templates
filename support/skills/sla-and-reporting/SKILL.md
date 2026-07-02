---
name: sla-and-reporting
description: Use to keep support honest about its own performance — measuring first-response and resolution time against the SLA, catching tickets about to breach BEFORE they do, and producing the weekly support report (volume by category, breaches, top issues, recurring KB gaps). Uses inbox timestamps + Linear + the policy's SLA targets; delivers via Slack/email.
---

# SLA tracking & reporting

Elite support measures itself. Two jobs: **prevent SLA breaches in real time**,
and **report the truth weekly** so the team fixes root causes, not just tickets.
Targets come from `knowledge/support-policy.md` (first-response 15 min in
business hours; same-day next step).

## Measure first-response + resolution time
You compute these from message timestamps — no special API:
- **First response** = time from the customer's first inbound message to the
  first outbound reply on the thread. Pull the thread (email-support skill); the
  first `From: <customer>` timestamp vs. the first `From: <us>` reply.
- **Resolution / next step** = time to the message that gave a real answer or a
  clear next step (not just an ack).
- Only count **business hours** (policy working hours) — a ticket that arrives
  at 2am isn't "breaching" until the morning window.

## Catch breaches BEFORE they happen (the valuable part)
On each triage sweep, flag tickets **approaching** the SLA, not just past it:
- Search the queue for unanswered inbound older than ~⅔ of the SLA window
  (e.g. `newer_than:1h` inverted → threads with no outbound reply after ~10 min
  in business hours), and any `escalated` ticket with no movement.
- For each at-risk ticket, either draft the response now or flag it urgently
  (internal-comms skill) so a human catches it. A prevented breach beats a
  reported one.

## Weekly report (deliver as one message)
Assemble from the week's labeled threads + Linear:
1. **Volume** — total handled, split by category (`question`/`bug`/`billing`/
   `feature-request`/`account-security`).
2. **Speed** — median + p90 first-response and resolution vs. target; count of
   SLA breaches (and which).
3. **Escalations** — how many, to where, and how many bugs became Linear issues
   (link the top ones).
4. **Top issues** — the 3-5 themes that drove the most tickets (cluster by
   subject/topic). This is the signal product should act on.
5. **KB gaps** — questions that had no good doc answer (from the knowledge-base
   skill's proposed updates) — the docs to write next.
6. **CSAT** — if a survey tool is connected, pull scores; otherwise track proxy
   signals (reopened tickets, explicit thank-yous vs. complaints) and say so.

Deliver it to {{escalation_channel}} (or {{support_email}}'s team) — scannable,
a few lines per section, with the one recommendation that matters most.

## Rules
1. **Prevent, then report.** Catching an at-risk ticket in the moment is worth
   more than any weekly number.
2. **Business hours only** for SLA math — don't cry breach on an overnight email.
3. **Report themes, not just counts.** "37 tickets" is noise; "12 were the same
   export bug (ENG-482)" is action.
4. **Every KB gap becomes a doc suggestion** (knowledge-base skill) so the same
   question doesn't drive tickets next week.
5. **Honest numbers.** Don't massage the medians — a report the team can't trust
   is worse than no report.
