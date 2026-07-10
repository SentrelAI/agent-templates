---
name: code-context
description: Use to understand what needs documenting from the source of truth — reading files, searching the codebase for a symbol, and reviewing merged PRs / releases to catch shipped changes. This is where doc accuracy starts. Covers the GitHub REST API through the mcp__apps__request proxy.
---

# Code context (GitHub)

Docs are only as accurate as your reading of the code. You read the source
through the **`request`** tool:

```
request({ provider:"github", method, path, query?, body? })
```

- **Base `https://api.github.com`.** Auth injected. Repos come from
  {{repositories}} as `owner/name`. Result `{ status, body }`.

Resolve `<default_branch>` once per repo: `GET /repos/<o>/<r>` →
`body.default_branch` — never hardcode `main`; on a `master` repo every
"what merged" query silently returns nothing.

## Read the actual source (verify, don't guess)
```
request({ provider:"github", method:"GET",
  path:"/repos/<owner>/<name>/contents/<path>", query:{ ref:"<default_branch>" } })
// → body.content is base64-encoded; decode to read. This is the ground truth
//   for signatures, params, defaults, return shapes, error paths.
```
Read the real function/handler/schema before documenting it. A doc claim without
a source line behind it is a guess.

## Find where a symbol lives
```
request({ provider:"github", method:"GET", path:"/search/code",
  query:{ q:"createInvoice repo:<owner>/<name>" } })
// → body.items[] { path, repository } — then read the file(s)
```

## Catch shipped changes (the sync job)
```
request({ provider:"github", method:"GET", path:"/repos/<owner>/<name>/pulls",
  query:{ state:"closed", base:"<default_branch>", sort:"updated", direction:"desc", per_page:30 } })
// merged PRs: body[].merged_at is set. Read the PR + its /files (patch) to see
// what public behavior/API/config changed → what docs need updating.
request({ provider:"github", method:"GET", path:"/repos/<owner>/<name>/releases",
  query:{ per_page:5 } })
// → release notes + tag → what shipped since the last doc pass
```

## Rules
1. **Read the source before you write** — signatures, params, defaults, returns,
   errors come from the code, not intuition.
2. **Cite the source line** in your notes (e.g. `verified against payments.rb:88`)
   so accuracy is traceable.
3. **Diff-driven** — merged PRs + releases tell you what changed and what docs
   are now stale.
4. **Match the branch/version** you're documenting (usually `main`/latest release)
   — don't document code that isn't shipped.

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
