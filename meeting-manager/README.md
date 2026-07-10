# Mira — meeting manager bundle

Makes meetings actually worth having. Prepares a real agenda before each one,
turns *your* raw notes/transcript/recording into decisions and action items, sends a crisp recap after, and
chases the follow-ups so nothing agreed-in-the-room quietly dies. **Drafts
everything; a human approves what goes out.**

> Distinct from **Rio** (the scheduler, who *books* meetings) — Mira makes the
> meetings you have *productive*.

## How it works

- **Agendas** via **Google Calendar or Outlook** — reads upcoming meetings,
  pulls context, and **drafts a real agenda** (objective, decision needed, timed
  topics) — or suggests going async when there's no decision to make.
- **Notes + action items** via **Notion** — captures decisions, action items
  (each with an **owner + due date**), and open questions as structured,
  findable notes + a tracker.
- **Recaps + follow-through** via **Gmail/Outlook** (drafted, human sends) and
  **Slack** (native `slack.post`) — a same-day recap and relentless action-item
  chasing until everything's done.

The three skills (`agenda-prep`, `notes-and-actions`, `followup-tracking`) are
**knowledge** — real endpoints + the multi-provider gotchas.

## Connect at /integrations
- **Google Calendar / Outlook** (required) — the meetings.
- **Gmail / Outlook** (required) — agendas + recaps (drafted).
- **Notion** — notes, decisions, and the action-item tracker.
- **Slack** (optional channel) — agendas, recaps, action-item nudges.

## Guardrails (baked in)
- **Draft-only** — nothing attendee-facing (agenda/recap) sends without approval.
- **Captures, doesn't decide** — records the room's decision neutrally; never
  makes the call.
- **Never reassigns** an action item or changes a decision — nudges + surfaces.

Edit `knowledge/meeting-standards.md` to set your agenda/recap templates and
action-item rules without touching the persona.
