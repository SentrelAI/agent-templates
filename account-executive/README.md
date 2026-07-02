# Ace — account executive bundle

A full-cycle closer's operator. Keeps the CRM honest, drives deals through the
pipeline with a real methodology, schedules the demos and follow-ups, and
drafts every customer touch — so the rep spends their time selling, not doing
data entry. Nothing is sent or committed without approval, and commercials
(price/terms) are hard-blocked.

## How it works

- **CRM** via the apps tool — Ace reads + updates deals, contacts, notes, and
  tasks in **HubSpot, Salesforce, OR Pipedrive** (whichever is connected):
  pipeline hygiene, MEDDIC notes, next-step tasks. Won/amount/commercial-stage
  changes are approval-gated.
- **Scheduling** via **Google Calendar OR Outlook** — books demos/calls
  (free/busy, holds, approval-gated booking) and preps the rep before each.
- **Follow-ups** via **Gmail OR Outlook** — reads deal threads and **drafts**
  recaps, re-engagement, and next-step emails that advance the deal. Never sends.

The three skills (`crm`, `deal-scheduling`, `followups-and-outreach`) are
**knowledge** — real endpoints + per-provider gotchas (HubSpot association ids /
Salesforce SOQL / Pipedrive REST; Google free/busy + relative paths / Outlook
`getSchedule`; Gmail base64url drafts / Outlook `createReply`).

## Connect at /integrations (pick one per row)
- **CRM** (required) — HubSpot **/** Salesforce **/** Pipedrive. *(All on the
  Nango roadmap — the template is complete and each option lights up as it's
  added.)*
- **Calendar** (required) — Google Calendar **/** Outlook.
- **Inbox** (required) — Gmail **/** Outlook.

## Guardrails (baked in)
- **Draft-only** — nothing goes to a prospect without the rep's approval.
- **No pricing/discounts/terms** — commercials are the rep's call (blocked).
- **No booking/moving** customer meetings, or marking deals won, without approval.

Edit `knowledge/sales-playbook.md` to run your methodology, stages, discovery
questions, objection frames, and pricing guardrails without touching the persona.
