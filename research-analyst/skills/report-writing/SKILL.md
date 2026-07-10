---
name: report-writing
description: Use to write and store the research report — structuring it answer-first with findings, citations, confidence levels, disagreements, and a source list, then saving it to the Notion research space. Also delivers the headline via Slack. Covers the Notion API through the mcp__apps__request proxy.
---

# Report writing (Notion)

The research is only useful if it's clear and findable. You write + store the
report through the **`request`** tool.

## The report structure (answer-first)
1. **The question** — precisely as scoped.
2. **Bottom line** — the answer in 2-3 sentences, with overall confidence.
3. **Findings** — each a claim + its **source** + a **confidence level**
   (high/medium/low). Group by sub-question.
4. **Disagreements / uncertainty** — where sources conflict or the evidence is
   thin, stated plainly.
5. **Open questions** — what's unresolved and what would resolve it.
6. **Sources** — the full list, so anyone can check every claim.

## Write it to the research space (Notion)
Create a report page in the research database:
```
request({ provider:"notion", method:"POST", path:"/v1/pages",
  body:{ parent:{ database_id:"<researchDbId>" },
         properties:{ Title:{ title:[{ text:{ content:"<question> — findings" } }] },
                      Confidence:{ select:{ name:"Medium" } } },
         children:[
           { object:"block", type:"heading_2", heading_2:{ rich_text:[{ text:{ content:"Bottom line" } }] } },
           { object:"block", type:"paragraph", paragraph:{ rich_text:[{ text:{ content:"<answer> (medium confidence)" } }] } },
           { object:"block", type:"heading_2", heading_2:{ rich_text:[{ text:{ content:"Findings" } }] } },
           { object:"block", type:"bulleted_list_item", bulleted_list_item:{ rich_text:[{ text:{ content:"Competitor X shipped Y in Mar 2026 [high] — <source>" } }] } },
           { object:"block", type:"heading_2", heading_2:{ rich_text:[{ text:{ content:"Sources" } }] } }
         ] } })
```
- **Base `https://api.notion.com`**; auth + `Notion-Version` injected.
- Long reports: create the page, then `PATCH /v1/blocks/<pageId>/children` to
  append the rest (Notion caps children per request).

## Deliver the headline (Slack — native channel)
Post the bottom line + the report link; don't paste the whole report:
```
slack.post({ text: "🔎 *<question>*\nBottom line: <answer> _(medium confidence)_.\nKey: 2 competitors shipped this in the last year. Full findings + sources: <report link>" })
```
(`slack.post` posts to the agent's own channel; no apps-proxy call.)

## Rules
1. **Answer first** — the bottom line before the evidence.
2. **Every finding: source + confidence.** No exceptions.
3. **Surface disagreement + open questions** — don't hide a messy evidence base.
4. **Sources list at the end** — anyone can verify every claim.
5. **Deliver the headline + a link** — don't dump the whole report into a
   message.

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
