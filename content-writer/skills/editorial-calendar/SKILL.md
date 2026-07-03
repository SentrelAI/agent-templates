---
name: editorial-calendar
description: Use to run the editorial calendar in Notion — reading what's due, sharpening a brief, moving a piece through stages (idea → brief → drafting → ready for review), and recording title/meta/angle. Covers the Notion API through the mcp__apps__request proxy. This is where a piece's plan + status live.
---

# Editorial calendar (Notion)

The calendar is where content gets planned and tracked. You run it through the
**`request`** tool:

```
request({ provider:"notion", method, path, query?, body? })
```

- **Base `https://api.notion.com`.** Auth + `Notion-Version` injected. Result
  `{ status, body }`. The calendar is a Notion **database** (one page per piece,
  a Status property + Due date).

## See what's due / in flight
```
request({ provider:"notion", method:"POST", path:"/v1/databases/<calendarDbId>/query",
  body:{ filter:{ property:"Status", select:{ does_not_equal:"Published" } },
         sorts:[{ property:"Due", direction:"ascending" }] } })
// → body.results[] pieces { id, properties:{ Title, Status, Due, Angle, Keyword } }
```

## Sharpen + read the brief
A vague brief makes a vague piece. Read the piece's page (the brief lives in the
body blocks) — `GET /v1/blocks/<pageId>/children` — and make sure it has: goal,
audience takeaway, angle, keyword/topic, format. If any is missing, propose it
and flag for a human.

## Move a piece through stages
```
request({ provider:"notion", method:"PATCH", path:"/v1/pages/<pieceId>",
  body:{ properties:{ Status:{ select:{ name:"Ready for review" } },
                      Title:{ title:[{ text:{ content:"<final working title>" } }] } } } })
```
Record the **meta description** + the Google Doc link as properties (or a block)
so the reviewer + publisher have everything in one place. Stages: `Idea → Brief →
Drafting → Ready for review → (human) Published`. **Cleo never sets Published.**

## Add an idea
```
request({ provider:"notion", method:"POST", path:"/v1/pages",
  body:{ parent:{ database_id:"<calendarDbId>" },
         properties:{ Title:{ title:[{ text:{ content:"<idea>" } }] },
                      Status:{ select:{ name:"Idea" } } } } })
```

## Rules
1. **Every piece has a real brief** before drafting — goal, audience takeaway,
   angle, keyword, format.
2. **Status reflects reality** — move it as it progresses; stop at "Ready for
   review" (publishing is a human's call).
3. **One place for everything** — title, meta, angle, and the draft link live on
   the piece's page.
4. **Nothing is due-and-forgotten** — the weekly plan surfaces what's due; start
   it early.
