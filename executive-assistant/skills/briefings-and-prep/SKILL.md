---
name: briefings-and-prep
description: Use to build the morning brief and per-meeting prep — gathering context from the calendar, inbox, past notes in Notion, and a web check on external people, then synthesizing a scannable brief, and saving meeting notes/action items back to Notion. Covers the Notion API through the mcp__apps__request proxy for reading past context and writing notes.
---

# Briefings & prep

Your job here is synthesis: pull the scattered context and hand
{{principal_name}} exactly what they need to walk in ready — nothing more.

## The morning brief (delivered as one email)
Assemble, in this order:
1. **Today at a glance** — each meeting from the calendar (calendar-management
   skill) with a one-line prep note: who, why, the one thing to know, the
   decision wanted.
2. **Needs you** — the 3-5 inbox threads that actually require
   {{principal_name}} (inbox-triage skill), each one line.
3. **Follow-ups due today** — from the running list (coordination skill).
4. **Fix-its** — conflicts, unprotected focus time, or gaps to resolve, each
   with a recommendation.

Keep it scannable — a screen, not a scroll. Draft the email; it goes out per
the schedule to {{principal_name}} only (internal), so it doesn't need approval
the way outbound-to-others does.

## Per-meeting prep
For each meeting, gather:
- **Prior context** — search past notes + threads. Read relevant Notion notes:
  `POST /v1/search` with `body:{ query:"<person/company/topic>", filter:{ property:"object", value:"page" } }`, then read the page blocks
  (`GET /v1/blocks/<pageId>/children`).
- **Who they are** — for external attendees, a quick `web-search` on the
  person/company (recent news, role, funding) so {{principal_name}} isn't
  meeting a stranger cold.
- **The ask** — what {{principal_name}} wants out of the meeting, and any open
  thread it connects to.

Deliver prep in the morning brief, and re-surface the specific brief ~15 min
before the meeting if it's significant.

## Save notes + action items (Notion)
After a meeting (or when {{principal_name}} shares notes), write them somewhere
findable so next time's prep is richer. Create a note page under the notes
database/parent:

```
request({ provider:"notion", method:"POST", path:"/v1/pages",
  body:{ parent:{ database_id:"<notes db id>" },
         properties:{ Name:{ title:[{ text:{ content:"<Meeting> — <date>" } }] } },
         children:[
           { object:"block", type:"heading_2",
             heading_2:{ rich_text:[{ text:{ content:"Decisions" } }] } },
           { object:"block", type:"bulleted_list_item",
             bulleted_list_item:{ rich_text:[{ text:{ content:"<decision>" } }] } },
           { object:"block", type:"heading_2",
             heading_2:{ rich_text:[{ text:{ content:"Action items" } }] } },
           { object:"block", type:"to_do",
             to_do:{ rich_text:[{ text:{ content:"<owner>: <action> (due <date>)" } }], checked:false } }
         ] } })
```

(If there's no notes database configured, keep notes as tasks/reminders and
mention that a Notion notes DB would make prep sharper.) Every action item that
belongs to {{principal_name}} also becomes a tracked follow-up (coordination
skill).

## Rules

1. **Brief, not dump.** One thing to know per meeting, one line per item. If
   {{principal_name}} has to read more to learn less, it's wrong.
2. **Never walk in cold.** Every meeting gets at least a one-liner; external
   people get a quick web check.
3. **Notes feed the next prep.** Write decisions + action items back to Notion
   so context compounds.
4. **Recommend, don't just report.** For conflicts/gaps, include the fix, not
   just the problem.
5. **Internal brief, no approval; anything to others, approval.** The brief to
   {{principal_name}} is internal. Any prep that goes to an attendee is
   outbound → draft for approval.
