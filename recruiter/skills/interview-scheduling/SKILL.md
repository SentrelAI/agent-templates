---
name: interview-scheduling
description: Use to coordinate interviews across the panel — finding times that work for multiple interviewers with free/busy, holding tentative slots, and (after approval) booking the interview with the candidate + panel. Works with EITHER Google Calendar or Outlook. Covers both APIs through the mcp__apps__request proxy.
---

# Interview scheduling (Google **or** Outlook)

You coordinate interviews through the **`request`** tool, on whichever calendar
is connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"google_calendar"`** → base `https://www.googleapis.com/calendar/v3`
  (versioned → relative paths `/freeBusy`, `/calendars/primary/events`).
- **`provider:"outlook"`** → base `https://graph.microsoft.com/v1.0`, prefix `/me`.
- Auth injected. Always use {{timezone}} (the team's timezone); confirm the candidate's timezone from their reply and state both in the invite. **Book only after approval** —
  an interview involves other people.

## Find a time across the whole panel
- **Google:** `POST /freeBusy` ·
  `body:{ timeMin, timeMax, timeZone, items:[{id:"panelist1@x.com"},{id:"panelist2@x.com"}] }`
  → `body.calendars[<id>].busy`.
- **Outlook:** `POST /me/calendar/getSchedule` ·
  `body:{ schedules:["panelist1@x.com","panelist2@x.com"], startTime, endTime, availabilityViewInterval:30 }`
  → `body.value[].scheduleItems`.
Compute the overlap that fits everyone + leaves interview length + buffer;
propose 2-3 options.

## Hold vs. book
- **Hold (auto):** a tentative `[HOLD]` block (Google `status:"tentative"` /
  Outlook `showAs:"tentative"`), no attendees, while you wait for approval.
- **Book (approval-gated):** invites the candidate + panel → needs a yes.
  - **Google:** `POST /calendars/primary/events?sendUpdates=all&conferenceDataVersion=1`
    with `attendees` + `conferenceData.createRequest` (Meet link).
  - **Outlook:** `POST /me/events` with `attendees` + `isOnlineMeeting:true,
    onlineMeetingProvider:"teamsForBusiness"` (Graph emails the invite).

## Set the panel up to succeed
In the event description (or a separate note), include: the candidate + résumé
link, the **structured questions/competencies each interviewer covers**, and the
scorecard link — so the interview is consistent and fair, not ad-hoc.

## Rules
1. **Propose, hold, then book.** Offer options + hold tentatively; confirm only
   after approval — booking notifies the candidate + panel.
2. **Respect everyone's time** — find real overlap, add buffers, don't stack a
   panelist back-to-back.
3. **Arm the panel** with the candidate context + the questions to cover, so
   interviews are structured and fair.
4. **Timezones explicit**; `sendUpdates:"all"` (Google) only on the real,
   approved booking.
5. **Chase feedback** afterward so the candidate isn't left waiting on a slow
   panelist.

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
