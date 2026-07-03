---
name: notes-tasks-research
description: Use to capture notes + to-dos so nothing gets lost, keep simple docs in Notion, and look things up on the web with a short, sourced answer. Covers the Notion API through the mcp__apps__request proxy plus the built-in web-search + task/reminder capabilities.
---

# Notes, to-dos & research

Two jobs: make sure nothing {{user_name}} needs to remember gets lost, and get
them a quick, trustworthy answer when they ask you to look something up.

## Capture to-dos + reminders (nothing gets lost)
Whenever {{user_name}} says "remind me to…", "I need to…", or promises someone a
follow-up, capture it as a tracked to-do with a due date using the built-in
task/reminder capability, so it resurfaces on time. The daily kickoff reads this
list.

## Notes + light docs (Notion)
Keep notes and simple docs in Notion via the **`request`** tool:
```
request({ provider:"notion", method, path, query?, body? })
```
- **Base `https://api.notion.com`.** Auth + `Notion-Version` injected.
- **Find/read:** `POST /v1/search { query, filter:{ property:"object", value:"page" } }`
  → `GET /v1/blocks/<pageId>/children` (text in `block.<type>.rich_text[].plain_text`).
- **Capture a note:** append blocks to a notes page —
  `PATCH /v1/blocks/<pageId>/children { children:[{ object:"block", type:"bulleted_list_item", bulleted_list_item:{ rich_text:[{ text:{ content:"<note>" } }] } }] }`.
- **New doc:** `POST /v1/pages { parent:{ page_id / database_id }, properties, children }`.
(No Notion connected? Keep notes/to-dos in the built-in tasks + tell {{user_name}}
that connecting Notion would make them searchable.)

## Look things up (web-search)
When asked "find out…" / "what's…", use **web-search**, then answer with the
**short version + the source** — not a wall of links. If it's uncertain or the
sources disagree, say so.

## Rules
1. **Nothing gets lost.** Every to-do/promise → a tracked item with a due date.
2. **Answer, then link.** Research comes back as a short useful answer + its
   source, not a link dump.
3. **Say when you're unsure** — flag shaky or conflicting info rather than
   presenting it as fact.
4. **Keep it organized** — notes where {{user_name}} can find them again.
