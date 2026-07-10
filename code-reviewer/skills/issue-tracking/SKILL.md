---
name: issue-tracking
description: Use to connect a PR to the work it does — finding the linked Linear issue, posting the review verdict as a comment on it, and filing follow-up issues for deferred fixes / tech debt so nothing found in review gets lost. Covers Linear's GraphQL API through the mcp__apps__request proxy.
---

# Issue tracking (Linear)

A review is only closed when the *work* reflects it. You talk to Linear's
**GraphQL** API through the **`request`** tool:

```
request({ provider:"linear", method:"POST", path:"/graphql",
  body:{ query:"<graphql>", variables:{ ... } } })
```

- **Base `https://api.linear.app`**, single endpoint `/graphql` (always POST).
  Auth injected. Data is in `body.data`; **check `body.errors`** (GraphQL returns
  200 even on validation errors).

## Find the issue a PR belongs to
PRs usually reference the issue by identifier in the title/branch (e.g.
`ENG-482`). Resolve it:
```graphql
query($id: String!) { issue(id: $id) { id identifier title state { name } url } }
```
`variables:{ id:"ENG-482" }`. If no identifier is present, search by the PR
title with `issueSearch(query:)`.

## Post the review verdict on the issue
Keep the tracker honest — a comment linking the review:
```graphql
mutation Comment($input: CommentCreateInput!) { commentCreate(input:$input){ success } }
```
`variables:{ input:{ issueId:"<id>", body:"Reviewed PR #123 — requested changes: nil-guard in User#active + a missing test on the deleted-user path. <pr url>" } }`

## File follow-ups for deferred fixes (don't lose them)
When review surfaces real tech debt or a nice-to-have that shouldn't block this
PR, create a tracked issue instead of letting it evaporate:
```graphql
mutation Create($input: IssueCreateInput!) { issueCreate(input:$input){ success issue{ identifier url } } }
```
`variables:{ input:{ teamId:"<teamId>", title:"Refactor User#active — nil handling is duplicated in 3 places", description:"Found during review of PR #123. Not blocking, but worth a cleanup. <links>", priority:3 } }`
(Get `teamId` once via `query { teams { nodes { id key } } }`.)

## Rules
1. **Every review updates the tracker** — a comment with the verdict + PR link so
   the issue reflects reality.
2. **Deferred ≠ dropped.** Anything real you chose not to block on becomes a
   follow-up issue, linked back to the PR.
3. **Don't duplicate** — search before creating a follow-up; `+1`/comment an
   existing one instead.
4. **Check `body.errors`** on every call; a 200 isn't success in GraphQL.

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
