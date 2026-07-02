---
name: deal-scheduling
description: Use to book demos, discovery, and follow-up calls and to prep the rep before each — works with EITHER Google Calendar or Outlook/Microsoft 365. Read the calendar, find open times with free/busy, hold tentative slots, and (after approval) book/move customer meetings on whichever is connected.
---

# Deal scheduling (Google **or** Outlook)

You arrange customer meetings through the **`request`** tool, on whichever
calendar is connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"google_calendar"`** → base `https://www.googleapis.com/calendar/v3`
  (already versioned → relative paths `/calendars/primary/events`, `/freeBusy`).
- **`provider:"outlook"`** → base `https://graph.microsoft.com/v1.0`, prefix `/me`.
- Auth injected for both. **Always use {{rep_name}}'s timezone.** Detect which is
  connected and stay on it.

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

## Outlook / Microsoft 365 (Microsoft Graph)

Same flow, Graph endpoints:
- **Read upcoming** (expands recurrences): `GET /me/calendarView` ·
  `query:{ startDateTime, endDateTime, $orderby:"start/dateTime", $select:"subject,start,end,attendees,onlineMeeting" }`.
  Match events to CRM deals by attendee address/company.
- **Free/busy:** `POST /me/calendar/getSchedule` ·
  `body:{ schedules:["rep@x.com"], startTime:{dateTime,timeZone}, endTime:{dateTime,timeZone}, availabilityViewInterval:30 }`.
- **Hold (auto):** `POST /me/events` with `showAs:"tentative"`, `[HOLD]` subject,
  no attendees.
- **Book / move (approval-gated — Graph emails the invite on create):**
  `POST /me/events` · `body:{ subject, body:{contentType:"HTML",content:"<agenda>"},
  start:{dateTime,timeZone}, end:{dateTime,timeZone},
  attendees:[{ emailAddress:{ address, name }, type:"required" }],
  isOnlineMeeting:true, onlineMeetingProvider:"teamsForBusiness" }`. Move =
  `PATCH /me/events/<id>` (notifies attendees).

## Rules

1. **Propose, hold, then book.** Offer specific slots and hold tentatively;
   confirm only after {{rep_name}} approves — booking notifies the prospect.
2. **Prep buffer.** Leave time before a demo and after for notes; don't stack
   customer calls back-to-back.
3. **Timezones explicit**, `singleEvents:true` on reads, `sendUpdates:"all"`
   only when you actually book/move (post-approval).
4. **Tie every meeting to its deal** so the prep brief + CRM note have context.
