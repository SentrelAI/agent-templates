# Remy — customer support specialist bundle

A frontline support teammate. Triages the support inbox, drafts accurate,
on-brand replies grounded in your knowledge base, escalates real bugs to
engineering with a clean repro, and keeps the team in the loop — with human
approval before any reply leaves and a hard stop on refunds and account
changes.

## How it works

- **Email** via the apps tool — Remy reads, searches, labels, and **drafts**
  replies through the Gmail REST API (`mcp__apps__request({ provider:"gmail",
  … })`). It never calls `messages/send`; a human approves, and only then does
  a reply go out. OAuth tokens are injected by Rails — no credentials in the
  agent.
- **Grounding** via **Notion** — before answering, Remy searches the knowledge
  base and reads the actual page content, then cites the doc. If the answer
  isn't there, it escalates instead of inventing one.
- **Escalation** via **Linear** — real bugs / feature requests get a clean,
  de-duplicated issue with a repro, linked back to the customer thread.
- **Coordination** via **Slack** — escalation handoffs, urgent flags, and a
  daily queue summary go to the team channel. Internal only; customers are
  answered by email.

The four skills (`email-support`, `ticketing-and-escalation`,
`knowledge-base`, `internal-comms`) are **knowledge** — they teach Remy the
real endpoints, the order to call them, and the support-specific gotchas
(threading a reply, avoiding duplicate issues, grounding before answering).

## Connect at /integrations
- **Gmail** (required) — the support inbox.
- **Notion** — the knowledge base Remy answers from.
- **Linear** — where escalated bugs/feature requests are tracked.
- **Slack** — the team channel for escalations + summaries.

## Guardrails (baked in)
- Every customer-facing reply is a **draft** pending human approval.
- **Refunds, credits, plan changes, account/data edits, access resets are
  blocked** — Remy hands them to a human, always.
- Answers are **grounded in the knowledge base**; unverifiable questions
  escalate rather than getting a guessed answer.

Edit `knowledge/support-policy.md` to change SLAs, the escalation matrix, the
hard rules, and the canned-response library without touching the persona.
