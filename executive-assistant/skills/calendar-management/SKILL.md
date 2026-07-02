---
name: calendar-management
description: Use to read and arrange the principal's calendar — listing the day/week, finding open times across attendees with free/busy, holding tentative slots, and proposing/booking/moving meetings (booking + moving are approval-gated). Covers the Google Calendar API v3 through the mcp__apps__request proxy, timezones, and the recurring-event gotcha.
---

# Calendar management (Google Calendar)

You protect and arrange {{principal_name}}'s calendar via the Google Calendar
API through the **`request`** tool:

```
request({ provider:"google_calendar", method, path, query?, body? })
```

- **The base already includes the API version** (`https://www.googleapis.com/calendar/v3`),
  so paths are relative to it: use `/calendars/primary/events`, **not**
  `/calendar/v3/...`.
- The principal's calendar id is **`primary`**. Auth is injected — never handle
  a token. Result is `{ status, body }`.
- **Always work in {{principal_name}}'s timezone** (from {{working_hours}}).
  Send RFC 3339 datetimes with an offset (`2026-07-08T14:00:00-07:00`) or a
  `timeZone` field; never send a naive local time.

## Read the day / week

```
request({ provider:"google_calendar", method:"GET",
  path:"/calendars/primary/events",
  query:{ timeMin:"<RFC3339>", timeMax:"<RFC3339>",
          singleEvents:true, orderBy:"startTime" } })
// → body.items[] each with summary, start/end (dateTime or date), attendees, hangoutLink/location
```

`singleEvents:true` **expands recurring events** into individual instances —
always set it when reading, or you'll see the series master instead of today's
occurrence.

## Find open time (free/busy across people)

To propose times that work for everyone, query free/busy — don't eyeball it:

```
request({ provider:"google_calendar", method:"POST",
  path:"/freeBusy",
  body:{ timeMin:"<RFC3339>", timeMax:"<RFC3339>",
         timeZone:"<principal tz>",
         items:[{ id:"primary" }, { id:"person@x.com" }] } })
// → body.calendars[<id>].busy = [{ start, end }, ...]
```

Compute the gaps that fall inside {{working_hours}}, avoid focus blocks, leave
buffers, and pick 2-3 candidate slots.

## Hold vs. book

- **Hold (auto):** you may pencil in a tentative block on `primary` while you
  wait for approval — set `status:"tentative"` and a clear `[HOLD]` summary. No
  attendees yet, so no one is invited.
- **Book / move (approval-gated):** creating a real meeting or moving an
  existing one invites/notifies other people, so it needs {{principal_name}}'s
  yes first.

Create (after approval):

```
request({ provider:"google_calendar", method:"POST",
  path:"/calendars/primary/events",
  query:{ sendUpdates:"all" },
  body:{ summary:"<title>", description:"<agenda>",
         start:{ dateTime:"<RFC3339>", timeZone:"<tz>" },
         end:{ dateTime:"<RFC3339>", timeZone:"<tz>" },
         attendees:[{ email:"person@x.com" }],
         conferenceData:{ createRequest:{ requestId:"<uuid>" } } } })
```

(For a Google Meet link, also add `conferenceDataVersion:1` to `query`.)

Move / reschedule (after approval) — patch just the times:

```
request({ provider:"google_calendar", method:"PATCH",
  path:"/calendars/primary/events/<eventId>",
  query:{ sendUpdates:"all" },
  body:{ start:{ dateTime:"<new>", timeZone:"<tz>" },
         end:{ dateTime:"<new>", timeZone:"<tz>" } } })
```

## Rules

1. **Protect first.** Keep working hours + focus blocks sacred; don't book over
   them without asking. Add buffers; flag back-to-backs.
2. **Propose, then book.** Offer 2-3 specific slots and hold tentatively;
   confirm only after {{principal_name}} approves — booking/moving invites
   others.
3. **Timezones are explicit.** Always the principal's tz, RFC 3339 with offset.
4. **`singleEvents:true` on reads** so you see real occurrences, not series
   masters.
5. **`sendUpdates:"all"`** when you finally create/move so attendees actually
   get the invite/update — but only after approval.
