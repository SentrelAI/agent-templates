---
name: knowledge-base
description: Use before answering any product question — searching the Notion knowledge base, reading a page's full content to ground the reply, and proposing a doc update (as a comment, never a silent edit) when a ticket exposes a gap. Covers the Notion API through the mcp__apps__request proxy. This is the grounding step; skipping it leads to invented answers.
---

# Knowledge base (Notion)

Before you answer a product question, you search the knowledge base and answer
from what's actually written there. You call Notion's API through the
**`request`** tool:

```
request({ provider:"notion", method, path, query?, body? })
```

- **Base is `https://api.notion.com`.** Auth (and the required `Notion-Version`
  header) are injected by the integration — never handle a token. If a call
  returns 400 "missing version" or 401, say the Notion connection needs
  attention at /integrations rather than guessing.
- Result is `{ status, body }`; read `body`.

## 1. Search for the relevant doc

```
request({ provider:"notion", method:"POST", path:"/v1/search",
  body:{ query:"<the customer's topic, e.g. 'CSV export limits'>",
         filter:{ property:"object", value:"page" },
         page_size:5 } })
// → body.results = [{ id, properties, url }, ...] ranked by relevance
```

Pull the title from `properties` (the title-type property). Pick the best 1-2
matches — don't read the whole workspace.

## 2. Read the page content (this is what you ground on)

`/v1/search` returns metadata, not the body. Fetch the page's blocks to read it:

```
request({ provider:"notion", method:"GET",
  path:"/v1/blocks/<pageId>/children", query:{ page_size:100 } })
// → body.results[] blocks; text lives in block.<type>.rich_text[].plain_text
```

Walk the blocks (`paragraph`, `bulleted_list_item`, `heading_2`,
`code`, `callout`, …) and read `rich_text[].plain_text`. If a block
`has_children:true` (e.g. a toggle), fetch its children by that block id. Now
you have the real, current answer — quote/paraphrase it, and include the page
`url` in your reply so the customer can go deeper.

## 3. If the answer isn't there

Do **not** invent it. Either:
- escalate to the team (the answer needs a human), or
- if it's a genuine doc gap, flag it (step 4) and tell the customer you're
  confirming with the team.

## 4. Propose a doc update (comment — never a silent edit)

When a ticket reveals a gap or an out-of-date doc, you propose the fix as a
**comment** on the page. You never silently edit the knowledge base — it's the
source of truth, and a human owns changes to it:

```
request({ provider:"notion", method:"POST", path:"/v1/comments",
  body:{ parent:{ page_id:"<pageId>" },
         rich_text:[{ text:{ content:"Support gap: customers ask X; this page doesn't cover it. Suggested addition: … (from ticket <link>)." } }] } })
```

For a brand-new topic with no page at all, note it in the escalation/summary to
the team instead of creating pages autonomously.

## Rules

1. **Search before you answer — always.** An answer that didn't start here is a
   guess. Grounding is the whole job.
2. **Read the body, not just the title.** Titles lie; open the page and read the
   blocks before you rely on it.
3. **Cite the source.** Put the doc `url` in the reply. It builds trust and lets
   the customer self-serve next time.
4. **Trust the newer doc.** If two pages conflict, use the more recently edited
   one and flag the conflict to the team.
5. **Propose, don't edit.** Doc changes are comments/flags for a human — never a
   direct write to the knowledge base.
