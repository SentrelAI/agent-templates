---
name: status-reporting
description: Use to keep the project's status doc current and read plans/specs in Notion — searching for the status/plan page, reading its content, and updating a living status (append a dated status block; never rewrite the plan). Covers the Notion API through the mcp__apps__request proxy. The synthesis of an honest weekly status also lives here.
---

# Status reporting (Notion)

You keep the living status doc honest and read the plan through the **`request`**
tool:

```
request({ provider:"notion", method, path, query?, body? })
```

- **Base `https://api.notion.com`.** Auth + the required `Notion-Version` header
  are injected. Result `{ status, body }`. A 400 "missing version" / 401 → the
  Notion connection needs attention.

## Find + read the plan / status page
```
request({ provider:"notion", method:"POST", path:"/v1/search",
  body:{ query:"<project> status", filter:{ property:"object", value:"page" }, page_size:5 } })
// → body.results[] { id, url }
request({ provider:"notion", method:"GET", path:"/v1/blocks/<pageId>/children", query:{ page_size:100 } })
// walk blocks; text in block.<type>.rich_text[].plain_text
```
Read the plan/spec for context; read the status page to see what's already
recorded.

## Update the living status (append, don't rewrite)
Add a **dated status block** to the status page so history is preserved — never
overwrite the plan/spec:
```
request({ provider:"notion", method:"PATCH", path:"/v1/blocks/<statusPageId>/children",
  body:{ children:[
    { object:"block", type:"heading_3",
      heading_3:{ rich_text:[{ text:{ content:"Status — 2026-07-06" } }] } },
    { object:"block", type:"bulleted_list_item",
      bulleted_list_item:{ rich_text:[{ text:{ content:"Shipped: auth revamp (ENG-410)" } }] } },
    { object:"block", type:"callout",
      callout:{ rich_text:[{ text:{ content:"AT RISK: billing migration — QA slipping, moves launch to 7/15 unless we cut scope." } }] } }
  ] } })
```

## The weekly status (what to synthesize)
Pull the board (sprint-management skill) and write an **honest** status:
1. **Shipped** this week.
2. **On track** (one line) vs. **At risk** — with the reason + the date impact.
3. **Blockers + owners.**
4. **Needs a decision.**
Deliver it to the team channel (standup-and-unblocking skill) *and* append it to
the status doc. Flag at-risk dates **before** they slip.

## Rules
1. **Append, don't overwrite.** Status is a dated log; the plan/spec is
   read-only to you.
2. **Honest status only** — "at risk" comes with the reason + date impact; never
   paint it green.
3. **Read the plan for context** before you judge whether something's on track.
4. **One source of truth** — the status doc + the channel post say the same
   thing.
