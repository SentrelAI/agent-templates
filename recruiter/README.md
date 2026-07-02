# Reese — recruiter bundle

A structured, candidate-obsessed recruiter. Drafts personalized outreach and
handles inbound applications, coordinates interviews across the panel, and runs
a clean pipeline with structured scorecards — so hiring is fast, fair, and every
candidate gets a real experience. **Drafts everything; a human approves anything
a candidate sees and every hire/no-hire call.**

## How it works

- **Hiring inbox** via the apps tool — Reese reads applications/replies and
  **drafts** outreach, screening replies, and kind prompt declines through
  **Gmail or Outlook** (never sends).
- **Interviews** via **Google Calendar or Outlook** — finds times across the
  whole panel with free/busy, holds tentatively, and (after approval) books with
  the candidate + panel, armed with the structured questions.
- **Pipeline** via **Notion** — an ATS-lite: candidates by stage with structured
  **scorecards** (evidence per competency, not a vibe), sorted so nobody waits.
- **Coordination** via **Slack** (native `slack.post`) — shares candidates +
  pipeline pulse with the hiring team.

The three skills (`candidate-outreach`, `interview-scheduling`,
`pipeline-and-scorecards`) are **knowledge** — real endpoints + gotchas
(multi-provider mail drafts, panel free/busy, Notion scorecards).

## Connect at /integrations
- **Gmail / Outlook** (required) — the hiring inbox.
- **Google Calendar / Outlook** (required) — interview coordination.
- **Notion** — the candidate pipeline + scorecards.
- **LinkedIn** (optional) — post a role / read a public profile. *(Candidate
  search + InMail need LinkedIn's gated Recruiter API, not available here — a
  dedicated ATS like Greenhouse/Lever is the roadmap upgrade.)*

## Guardrails (baked in)
- **Draft-only** — nothing a candidate sees sends without approval.
- **No advance/reject/offer decisions** — Reese recommends; a human decides;
  offers are blocked.
- **Fair-hiring** — screens on evidence of the work only; never touches a
  protected characteristic.

Edit `knowledge/hiring-playbook.md` to set your scorecards, screening rubric,
structured questions, and fair-hiring rules without touching the persona.
