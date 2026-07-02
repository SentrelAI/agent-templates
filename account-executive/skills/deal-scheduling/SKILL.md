---
name: deal-scheduling
description: Use to book demos, discovery, and follow-up calls and to prep the rep before each — reading the calendar, finding open times with free/busy, holding tentative slots, and (after approval) booking/moving customer meetings. Covers the Google Calendar API through the mcp__apps__request proxy.
---

# Deal scheduling (Google Calendar)

You arrange customer meetings via the Google Calendar API through the
**`request`** tool:

```
request({ provider:"google_calendar", method, path, query?, body? })
```

- **The base already includes the API version** (`.../calendar/v3`), so paths
  are relative: `/calendars/primary/events`, `/freeBusy` — **not**
  `/calendar/v3/...`. Auth is injected. Calendar id is `primary`.
- **Always use {{rep_name}}'s timezone** and RFC 3339 with an offset.

## Read the meetings coming up

```
request({ provider:"google_calendar", method:"GET",
  path:"/calendars/primary/events",
  query:{ timeMin:"<RFC3339>", timeMax:"<RFC3339>",
          singleEvents:true, orderBy:"startTime" } })
// singleEvents:true expands recurrences into real instances
```

Use this for the pre-meeting prep pass — match events to CRM deals by attendee
email/company.

## Find a time that works

```
request({ provider:"google_calendar", method:"POST", path:"/freeBusy",
  body:{ timeMin:"<RFC3339>", timeMax:"<RFC3339>", timeZone:"<tz>",
         items:[{ id:"primary" }] } })
// → body.calendars.primary.busy = [{ start, end }]
```

Propose 2-3 concrete slots inside working hours; for a demo, leave prep buffer
before and note-time after.

## Hold vs. book

- **Hold (auto):** pencil a tentative block on `primary` (`status:"tentative"`,
  `[HOLD]` summary, no attendees) while you wait for approval.
- **Book / move (approval-gated):** inviting or moving a customer meeting
  notifies the prospect — needs {{rep_name}}'s yes.

Book (after approval), with a Meet link:

```
request({ provider:"google_calendar", method:"POST", path:"/calendars/primary/events",
  query:{ sendUpdates:"all", conferenceDataVersion:1 },
  body:{ summary:"<Company> × {{company_name}} — <demo/discovery>",
         description:"<agenda + deal context>",
         start:{ dateTime:"<RFC3339>", timeZone:"<tz>" },
         end:{ dateTime:"<RFC3339>", timeZone:"<tz>" },
         attendees:[{ email:"prospect@x.com" }, { email:"<rep email>" }],
         conferenceData:{ createRequest:{ requestId:"<uuid>" } } } })
```

Move (after approval): `PATCH /calendars/primary/events/<eventId>` with new
`start`/`end` and `query:{ sendUpdates:"all" }`.

## Rules

1. **Propose, hold, then book.** Offer specific slots and hold tentatively;
   confirm only after {{rep_name}} approves — booking notifies the prospect.
2. **Prep buffer.** Leave time before a demo and after for notes; don't stack
   customer calls back-to-back.
3. **Timezones explicit**, `singleEvents:true` on reads, `sendUpdates:"all"`
   only when you actually book/move (post-approval).
4. **Tie every meeting to its deal** so the prep brief + CRM note have context.
