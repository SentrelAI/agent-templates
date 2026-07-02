---
name: error-context
description: Use to check whether a PR touches code that's currently failing in production — pulling unresolved Sentry issues and matching their culprit/stack files to the PR's changed files, so risky areas get extra scrutiny (and so a "fix" can be verified against the real error). Covers the Sentry API through the mcp__apps__request proxy.
---

# Error context (Sentry)

Before you bless a change, know whether it lands in a part of the codebase
that's already on fire. You read Sentry through the **`request`** tool:

```
request({ provider:"sentry", method:"GET", path, query? })
```

- **Base is `https://sentry.io/api/0`.** Auth (an API token) is injected — never
  handle it. Result `{ status, body }`. A 401 → Sentry isn't connected.

## Pull the active errors
```
request({ provider:"sentry", method:"GET",
  path:"/organizations/<org-slug>/issues/",
  query:{ query:"is:unresolved", statsPeriod:"14d", sort:"freq", limit:50 } })
// → body[] each { id, title, culprit, count, userCount, permalink,
//                 metadata:{ type, value, filename? } }
```
(Scope to one project with
`/projects/<org-slug>/<project-slug>/issues/` if the org has many.)
`culprit` is usually `function (path/to/file.rb)`, and `metadata.filename`
often names the file — that's your match key.

## Match errors to the PR's changed files
Take the changed file paths from the pr-review skill (`files[].filename`) and
check each active issue's `culprit` / `metadata.filename` for a match:
- **Direct hit** — the PR changes a file that's actively throwing. Flag it: is
  this PR *fixing* that error, or unknowingly touching a fragile area? If it
  claims to fix it, verify the change actually addresses the stack trace.
- **Nearby** — the PR changes a caller/collaborator of a failing file. Note it
  and scrutinize the interaction.

## Read a specific error's detail (when a PR claims to fix it)
```
request({ provider:"sentry", method:"GET",
  path:"/organizations/<org-slug>/issues/<issue-id>/events/latest/" })
// → the stack trace, breadcrumbs, and request context — confirm the PR's change
//   is on the actual failing line/path, not adjacent to it.
```

## Rules
1. **Cross-reference every PR's changed files against active errors.** A change
   in a currently-failing file gets extra scrutiny, always.
2. **"Fixes the error" is a claim to verify** — read the stack trace and confirm
   the change is on the real failing path, not near it.
3. **Surface it in the review** — note active errors in touched files in the
   review body so the human weighs the risk.
4. **Read-only.** Never resolve/ignore/assign Sentry issues from here — that's a
   human's call; you're gathering context.
