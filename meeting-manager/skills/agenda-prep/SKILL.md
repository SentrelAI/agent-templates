---
name: agenda-prep
description: Use to prepare agendas — reading upcoming meetings on the calendar, pulling context (invite, linked thread/docs, prior decisions), and drafting a real agenda (objective, decisions needed, timed topics). Works with EITHER Google Calendar or Outlook through the mcp__apps__request proxy. Draft-only; a human sends.
---

# Agenda prep (Google **or** Outlook)

You make sure every meeting has a real agenda. Read the calendar through the
**`request`** tool, on whichever is connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"google_calendar"`** → base `https://www.googleapis.com/calendar/v3`
  (relative paths). **`provider:"outlook"`** → `https://graph.microsoft.com/v1.0`,
  `/me`. Auth injected.

## Read today's / upcoming meetings
- **Google:** `GET /calendars/primary/events?timeMin&timeMax&singleEvents=true&orderBy=startTime`
- **Outlook:** `GET /me/calendarView?startDateTime&endDateTime&$orderby=start/dateTime`
Each event has `summary/subject`, `attendees`, `description` (where an agenda
may already live), and any conferencing link.

## Does it already have an agenda?
Check the event `description`/`body` and any linked doc. If there's a real agenda
(objective + topics), leave it — don't duplicate. If not, draft one.

## Gather context, then draft
Pull what the meeting is *for*: the invite text, the email thread it came from
(read it via the followup-tracking skill's mail access), prior decisions/notes on
the topic (notes-and-actions skill), and a quick web check on any external
attendee.

Draft a **real** agenda:
```
Objective: <what this meeting is to accomplish>
Decision(s) needed: <the specific call(s) to make>
Topics:
  1. <topic> (~10m) — <context / pre-read link>
  2. <topic> (~15m) — …
Pre-reading: <links>
```
If there's **no decision to make**, say so and suggest it could be async/an
email — the best agenda is sometimes "cancel this."

## Deliver (draft, don't send)
Attach the draft agenda to the event (propose a `description` update) or draft it
as a message to attendees — but **send only after approval**. You may add the
agenda to the event description as a draft for the owner to confirm.

## Rules
1. **Every meeting gets an agenda** — or a recommendation to make it async.
2. **Objective + decision-needed** are the point; a topic list without them isn't
   an agenda.
3. **Draft, don't send** — attendee-facing goes out only on approval.
4. **Don't duplicate** an agenda that already exists.
5. **Timezone-aware**; expand recurring meetings on reads.

## Errors & pagination (standard)

- **401/403** — the connection is broken or missing: stop and tell the owner to
  reconnect the app at /integrations. Don't retry.
- **429** — back off ~30s and retry once; still failing → finish other work and
  pick this up next run. Use smaller pages.
- **5xx twice** — report the failure plainly. Never fabricate data you couldn't fetch.
- **Pagination** — never conclude "nothing new" from page one. Gmail/Calendar:
  `nextPageToken` → `pageToken`. Notion: `has_more`/`next_cursor` → `start_cursor`.
  GitHub: `Link: rel="next"`. Microsoft Graph: `@odata.nextLink`. Stripe:
  `has_more` + `starting_after`. Linear GraphQL: `pageInfo { hasNextPage endCursor }`.
