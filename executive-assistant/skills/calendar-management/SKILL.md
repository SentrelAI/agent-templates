---
name: calendar-management
description: Use to read and arrange the principal's calendar — works with EITHER Google Calendar or Outlook/Microsoft 365. Listing the day/week, finding open times with free/busy, holding tentative slots, and proposing/booking/moving meetings (booking + moving are approval-gated). Covers both APIs, provider detection, timezones, and the recurring-event gotcha.
---

# Calendar management (Google **or** Outlook)

You protect and arrange {{principal_name}}'s calendar through the **`request`**
tool, on whichever calendar is connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"google_calendar"`** → base `https://www.googleapis.com/calendar/v3`
  (already includes the version → paths are relative: `/calendars/primary/events`,
  `/freeBusy`). Calendar id is `primary`.
- **`provider:"outlook"`** → base `https://graph.microsoft.com/v1.0`, prefix
  `/me`.
- Auth injected for both — never handle a token. **Always work in
  {{principal_name}}'s timezone** ({{working_hours}}); send RFC 3339 datetimes
  with an offset (Google) or `{dateTime,timeZone}` objects (both). **Detect
  which is connected and stay on it.**

---

## Google Calendar

### Read the day / week
```
request({ provider:"google_calendar", method:"GET", path:"/calendars/primary/events",
  query:{ timeMin:"<RFC3339>", timeMax:"<RFC3339>", singleEvents:true, orderBy:"startTime" } })
```
`singleEvents:true` **expands recurring events** — always set it on reads.

### Find open time (free/busy)
```
request({ provider:"google_calendar", method:"POST", path:"/freeBusy",
  body:{ timeMin:"<RFC3339>", timeMax:"<RFC3339>", timeZone:"<tz>",
         items:[{ id:"primary" }, { id:"person@x.com" }] } })
// → body.calendars[<id>].busy = [{ start, end }]
```

### Hold vs. book / move
- **Hold (auto):** create a `status:"tentative"` `[HOLD]` event on `primary`, no
  attendees.
- **Book / move (approval-gated):** notifies attendees → needs approval.
```
// book (after approval, with a Meet link):
request({ provider:"google_calendar", method:"POST", path:"/calendars/primary/events",
  query:{ sendUpdates:"all", conferenceDataVersion:1 },
  body:{ summary, description, start:{ dateTime, timeZone }, end:{ dateTime, timeZone },
         attendees:[{ email:"x@y.com" }], conferenceData:{ createRequest:{ requestId:"<uuid>" } } } })
// move: PATCH /calendars/primary/events/<eventId> with new start/end + query sendUpdates:"all"
```

---

## Outlook / Microsoft 365 (Microsoft Graph)

### Read the day / week
Use `calendarView` (it **expands recurrences** into instances — the Graph
equivalent of `singleEvents`):
```
request({ provider:"outlook", method:"GET", path:"/me/calendarView",
  query:{ startDateTime:"<ISO>", endDateTime:"<ISO>", $orderby:"start/dateTime",
          $select:"subject,start,end,attendees,onlineMeeting,location" } })
// → body.value[]  (each start/end is { dateTime, timeZone })
```

### Find open time (free/busy)
```
request({ provider:"outlook", method:"POST", path:"/me/calendar/getSchedule",
  body:{ schedules:["principal@x.com","person@y.com"],
         startTime:{ dateTime:"<ISO>", timeZone:"<tz>" },
         endTime:{ dateTime:"<ISO>", timeZone:"<tz>" },
         availabilityViewInterval:30 } })
// → body.value[].scheduleItems = [{ status, start, end }]  (busy blocks per person)
```

### Hold vs. book / move
- **Hold (auto):** create an event with `showAs:"tentative"` + `[HOLD]` subject,
  no attendees.
- **Book / move (approval-gated):** Graph **emails the invite on create** — so
  creating with attendees notifies them; do it only after approval.
```
// book (after approval, with a Teams link):
request({ provider:"outlook", method:"POST", path:"/me/events",
  body:{ subject, body:{ contentType:"HTML", content:"<agenda>" },
         start:{ dateTime:"<ISO>", timeZone:"<tz>" }, end:{ dateTime:"<ISO>", timeZone:"<tz>" },
         attendees:[{ emailAddress:{ address:"x@y.com", name:"X" }, type:"required" }],
         isOnlineMeeting:true, onlineMeetingProvider:"teamsForBusiness" } })
// move: PATCH /me/events/<eventId> with new start/end (Graph notifies attendees)
```

---

## Rules (both providers)
1. **Protect first.** Keep working hours + focus blocks sacred; add buffers;
   flag back-to-backs. Don't book over them without asking.
2. **Propose, then book.** Offer 2-3 specific slots and hold tentatively; confirm
   only after {{principal_name}} approves — booking/moving notifies attendees.
3. **Timezones explicit** in the principal's tz. Expand recurrences on reads
   (`singleEvents:true` / `calendarView`).
4. **Notify only on real book/move, post-approval** (Google `sendUpdates:"all"`;
   Outlook create/patch with attendees emails them).
5. **One provider per session** — detect Google vs. Outlook and stay on it.
