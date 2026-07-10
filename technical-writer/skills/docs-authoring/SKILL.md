---
name: docs-authoring
description: Use to write and update docs — creating/updating pages in Notion (the docs home) and long-form guides in Google Docs, structured the way developers expect. Covers the Notion + Google Docs APIs through the mcp__apps__request proxy. Draft-only; publishing to a live docs site is a human's call.
---

# Docs authoring (Notion + Google Docs)

You write and maintain the docs through the **`request`** tool.

## Notion — the docs home (pages + reference)
```
request({ provider:"notion", method, path, query?, body? })
```
- **Base `https://api.notion.com`.** Auth + `Notion-Version` injected.
- **Find an existing doc:** `POST /v1/search { query, filter:{ property:"object", value:"page" } }`.
- **Read it** (to update): `GET /v1/blocks/<pageId>/children`.
- **New doc:** `POST /v1/pages { parent:{ database_id }, properties, children:[...] }`.
- **Update in place:** `PATCH /v1/blocks/<pageId>/children` only *appends* —
  used alone it stacks the corrected section under the stale one, the exact
  doc-that-lies this role exists to prevent. To replace a stale section:
  `GET /v1/blocks/<pageId>/children` (paginate via `next_cursor`), `DELETE
  /v1/blocks/<blockId>` for each stale block, then append the corrected
  blocks (≤100 per call). For one block's text: `PATCH /v1/blocks/<blockId>`. Use a `code` block for
  examples:
  ```
  { object:"block", type:"code", code:{ language:"bash",
    rich_text:[{ text:{ content:"curl -X POST https://api.acme.com/v1/invoices …" } }] } }
  ```

## Google Docs — long-form guides/tutorials
For a full guide/tutorial, a document may fit better:
create `POST /documents { title }`, insert the whole text in one batchUpdate at
index 1, then style headings — base `https://docs.googleapis.com/v1`, relative
paths, mind the character-index gotcha (insert first, style second).

## Structure every reference the same way
1. **What it does** — one line.
2. **How to use it** — the call/usage.
3. **Parameters** — name · type · required/optional · default · what it does.
4. **A working example** — realistic values, correct field names, would-actually-run.
5. **Returns / errors** — the shape + the failure modes.
6. **Gotchas** — nulls, rate limits, edge cases.

## Rules
1. **Draft/update, don't publish** — a human ships to the live docs site.
2. **Every reference has a runnable example** with realistic values (not
   `foo`/`bar` where a real shape teaches more).
3. **Update in place** — when a doc exists, edit it to match the code; don't
   leave a stale copy beside a new one.
4. **Predictable structure** — same shape every time so devs find answers fast.

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
