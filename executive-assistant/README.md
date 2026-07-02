# Vera — executive assistant bundle

A chief-of-staff-grade assistant. Triages the inbox, guards and arranges the
calendar, prepares the principal for every meeting, and chases down follow-ups
so nothing slips — drafting anything that goes out in their name and waiting
for a yes before it sends, books, or moves.

## How it works

- **Inbox** via the apps tool — Vera reads, sorts (needs-you / draft / delegate
  / archive), labels, and **drafts** replies in the principal's voice through
  the Gmail REST API. It never sends; a human approves.
- **Calendar** via the Google Calendar API — reads the day/week, finds open
  time with **free/busy** across attendees, holds tentative slots, and (after
  approval) books/moves meetings. Protects working hours + focus blocks.
- **Prep** — a scannable morning brief and per-meeting prep, pulling context
  from the calendar, inbox, past **Notion** notes, and a quick web check on
  external people.
- **Follow-through** — every commitment is tracked with an owner + due date and
  nudged on time over **Slack**; nothing goes cold.

The four skills (`inbox-triage`, `calendar-management`, `briefings-and-prep`,
`coordination-and-followups`) are **knowledge** — real endpoints, the order to
call them, and the gotchas (base64url drafts, `singleEvents:true`, timezones,
free/busy, Slack `body.ok`).

## Connect at /integrations
- **Gmail** (required) — the inbox.
- **Google Calendar** (required) — scheduling + protection.
- **Notion** — notes, briefs, and the follow-up list.
- **Slack** — team coordination + delivering the brief.

## Guardrails (baked in)
- **Nothing sends in the principal's name without approval** (draft-only).
- **No confirming, moving, or canceling** meetings with others until the
  principal says yes.
- Vera holds, drafts, arranges, and prepares — the principal decides.

Edit `knowledge/operating-manual.md` to set scheduling rules, VIPs, priorities,
communication style, and approval boundaries without touching the persona.
