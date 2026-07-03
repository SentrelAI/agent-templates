---
name: pipeline-and-scorecards
description: Use to run the candidate pipeline in Notion — moving candidates through stages, recording structured scorecards (evidence per competency, not a vibe), reading the role spec, and catching anyone waiting too long. Covers the Notion API through the mcp__apps__request proxy. This is the ATS-lite until a real ATS is connected.
---

# Pipeline & scorecards (Notion)

You keep the candidate pipeline honest through the **`request`** tool:

```
request({ provider:"notion", method, path, query?, body? })
```

- **Base `https://api.notion.com`.** Auth + `Notion-Version` header injected.
  Result `{ status, body }`. A 400 "missing version" / 401 → Notion needs
  attention.
- The pipeline is a Notion **database** (one page per candidate, a Stage
  property); scorecards are structured blocks on each candidate page.

## Read the pipeline (who's where, who's waiting)
```
request({ provider:"notion", method:"POST", path:"/v1/databases/<pipelineDbId>/query",
  body:{ filter:{ property:"Stage", select:{ does_not_equal:"Rejected" } },
         sorts:[{ property:"Last activity", direction:"ascending" }] } })
// → body.results[] candidate pages { id, properties:{ Name, Stage, Role, "Last activity" } }
```
Sorting by last activity surfaces whoever's been **waiting longest** — that's who
you draft a next step for first (no ghosting).

## Move a candidate + stamp activity
```
request({ provider:"notion", method:"PATCH", path:"/v1/pages/<candidatePageId>",
  body:{ properties:{ Stage:{ select:{ name:"Phone screen" } },
                      "Last activity":{ date:{ start:"2026-07-06" } } } } })
```
A stage move on a *person* is approval-gated (`advance_or_reject: ask`) — draft
the recommendation, get the yes, then record it.

## Record a structured scorecard (evidence, not vibe)
Append a scorecard block to the candidate's page — one line per competency from
the role's rubric (`hiring-playbook.md`), each with the **evidence**:
```
request({ provider:"notion", method:"PATCH", path:"/v1/blocks/<candidatePageId>/children",
  body:{ children:[
    { object:"block", type:"heading_3",
      heading_3:{ rich_text:[{ text:{ content:"Screen — 2026-07-06" } }] } },
    { object:"block", type:"bulleted_list_item",
      bulleted_list_item:{ rich_text:[{ text:{ content:"Backend depth: STRONG — shipped a payments system at scale (from résumé + repo)." } }] } },
    { object:"block", type:"bulleted_list_item",
      bulleted_list_item:{ rich_text:[{ text:{ content:"Ambiguity: UNPROVEN — worth testing in interview." } }] } },
    { object:"block", type:"paragraph",
      paragraph:{ rich_text:[{ text:{ content:"Recommendation: advance to phone screen. Reservation: ambiguity." } }] } }
  ] } })
```
**Never** record anything about a protected characteristic — only evidence of the
work.

## Rules
1. **Nobody in limbo.** Sort by last activity every sweep; the longest-waiting
   candidate gets a drafted next step first.
2. **Scorecards are evidence, per competency** — not "seems great." Cite where
   the evidence came from.
3. **Stage moves on a person are approval-gated** — recommend, get the yes, then
   record.
4. **Fair only** — never note age, gender, race, family status, disability, or
   anything protected. Evidence of the work, full stop.
5. **One source of truth** — the pipeline reflects the real stage + last touch
   for every candidate.
