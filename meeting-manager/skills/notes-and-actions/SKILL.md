---
name: notes-and-actions
description: Use to capture a meeting's real output in Notion — decisions, action items (each with an owner + due date), and open questions — as structured, findable notes, and to maintain the action-item tracker. Covers the Notion API through the mcp__apps__request proxy. Capture, don't decide.
---

# Notes & action items (Notion)

You capture what a meeting *produced* — not a transcript. Decisions, owned action
items, open questions. Through the **`request`** tool:

```
request({ provider:"notion", method, path, query?, body? })
```

- **Base `https://api.notion.com`.** Auth + `Notion-Version` injected. Notes live
  as pages; action items are rows in a tracker database (owner + due + status).

## Write the meeting notes page
```
request({ provider:"notion", method:"POST", path:"/v1/pages",
  body:{ parent:{ database_id:"<meetingsDbId>" },
         properties:{ Name:{ title:[{ text:{ content:"<Meeting> — <date>" } }] } },
         children:[
           { object:"block", type:"heading_2", heading_2:{ rich_text:[{ text:{ content:"Decisions" } }] } },
           { object:"block", type:"bulleted_list_item", bulleted_list_item:{ rich_text:[{ text:{ content:"Ship v2 on 7/20; cut the export feature to hit it." } }] } },
           { object:"block", type:"heading_2", heading_2:{ rich_text:[{ text:{ content:"Action items" } }] } },
           { object:"block", type:"to_do", to_do:{ rich_text:[{ text:{ content:"Dana — pull churn cohort by Fri 7/11" } }], checked:false } },
           { object:"block", type:"heading_2", heading_2:{ rich_text:[{ text:{ content:"Open questions" } }] } }
         ] } })
```

## Maintain the action-item tracker (owner + date — always)
Every action item is *also* a row in the tracker DB so the follow-up loop can
find it:
```
request({ provider:"notion", method:"POST", path:"/v1/pages",
  body:{ parent:{ database_id:"<actionsDbId>" },
         properties:{ Task:{ title:[{ text:{ content:"Pull churn cohort" } }] },
                      Owner:{ rich_text:[{ text:{ content:"Dana" } }] },
                      Due:{ date:{ start:"2026-07-11" } },
                      Status:{ select:{ name:"Open" } },
                      Meeting:{ rich_text:[{ text:{ content:"Product sync 7/6" } }] } } } })
```
**No orphan action items** — every one has an owner and a date, or it isn't
captured yet (ask who owns it).

## Capture neutrally
Record the decision the room actually made — accurately, no spin, no editorial.
Don't record your opinion as the outcome.

## Rules
1. **Output, not transcript** — decisions, action items, open questions.
2. **Every action item: owner + due date.** No "someone should."
3. **Decisions captured neutrally** — accurate, no editorializing.
4. **Also to the tracker** — so followups can chase it; the notes page alone
   isn't enough.
