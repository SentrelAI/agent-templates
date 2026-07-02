# Ace — account executive bundle

A full-cycle closer's operator. Keeps the CRM honest, drives deals through the
pipeline with a real methodology, schedules the demos and follow-ups, and
drafts every customer touch — so the rep spends their time selling, not doing
data entry. Nothing is sent or committed without approval, and commercials
(price/terms) are hard-blocked.

## How it works

- **CRM** via the apps tool — Ace reads + updates deals, contacts, notes, and
  tasks in **HubSpot** (`mcp__apps__request({ provider:"hubspot", … })`):
  pipeline hygiene, MEDDIC notes, next-step tasks. Won/amount/commercial-stage
  changes are approval-gated.
- **Scheduling** via **Google Calendar** — books demos/calls (free/busy, holds,
  approval-gated booking) and preps the rep before each.
- **Follow-ups** via **Gmail** — reads deal threads and **drafts** recaps,
  re-engagement, and next-step emails that advance the deal. Never sends.

The three skills (`crm-hubspot`, `deal-scheduling`, `followups-and-outreach`)
are **knowledge** — real endpoints + the gotchas (HubSpot search filters +
pagination + association type ids, calendar free/busy + relative paths, Gmail
base64url drafts + threading).

## Connect at /integrations
- **HubSpot** (required) — the system of record. *(On the Nango roadmap — this
  template is complete and becomes deployable the moment HubSpot is wired in.)*
- **Google Calendar** (required) — scheduling + prep.
- **Gmail** (required) — deal threads + drafted follow-ups.

## Guardrails (baked in)
- **Draft-only** — nothing goes to a prospect without the rep's approval.
- **No pricing/discounts/terms** — commercials are the rep's call (blocked).
- **No booking/moving** customer meetings, or marking deals won, without approval.

Edit `knowledge/sales-playbook.md` to run your methodology, stages, discovery
questions, objection frames, and pricing guardrails without touching the persona.
