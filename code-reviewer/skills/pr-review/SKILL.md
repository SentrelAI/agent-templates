---
name: pr-review
description: Use to review a pull request — listing open PRs, reading the diff file-by-file in context, reading existing comments to avoid repeats, and posting a review with inline comments + a verdict (comment / request-changes / approve). NEVER merges. Covers the GitHub REST API through the mcp__apps__request proxy, the review-vs-comment endpoints, and the inline-comment line/side gotcha.
---

# PR review (GitHub)

You review pull requests through the **`request`** tool:

```
request({ provider:"github", method, path, query?, body? })
```

- **Base is `https://api.github.com`.** Auth is injected — never handle a token.
- Result is `{ status, body }`. A 401/403 → GitHub isn't connected (or the token
  lacks repo scope); say so. Repos come from {{repositories}} as `owner/name`.

## Find PRs that need review
```
request({ provider:"github", method:"GET",
  path:"/repos/<owner>/<name>/pulls",
  query:{ state:"open", sort:"updated", direction:"desc", per_page:30 } })
// → body[] each { number, title, user, head:{ sha }, draft, requested_reviewers }
```
Skip `draft:true` unless asked. `head.sha` is the commit you'll attach the
review to.

## Read the change in context
Get the PR (title/description → intent) and the per-file diff:
```
request({ provider:"github", method:"GET", path:"/repos/<owner>/<name>/pulls/<number>" })
// → body.title, body.body (description), body.base/head, body.commits, body.additions/deletions
request({ provider:"github", method:"GET",
  path:"/repos/<owner>/<name>/pulls/<number>/files", query:{ per_page:100 } })
// → body[] each { filename, status, additions, deletions, patch }
```
`patch` is the unified diff for that file (with `@@` hunk headers giving line
numbers). Read it *with* the surrounding code in mind — fetch the full file if a
hunk's context isn't enough:
`GET /repos/<owner>/<name>/contents/<path>?ref=<head.sha>` → base64 `content`.
**Paginate** `/files` if the PR is large (follow `Link` / bump `page`).

## Don't repeat existing comments
```
request({ provider:"github", method:"GET",
  path:"/repos/<owner>/<name>/pulls/<number>/comments", query:{ per_page:100 } })
```
If a point is already raised, don't pile on.

## Post the review (inline comments + verdict, in one call)
Use the **reviews** endpoint — it batches inline comments with a single verdict:
```
request({ provider:"github", method:"POST",
  path:"/repos/<owner>/<name>/pulls/<number>/reviews",
  body:{
    commit_id:"<head.sha>",
    event:"COMMENT",              // or "REQUEST_CHANGES"; "APPROVE" is approval-gated
    body:"Summary: <the one thing that matters most>, then verdict + risk read.",
    comments:[
      { path:"app/models/user.rb", line:42, side:"RIGHT",
        body:"blocking: `find` returns nil if the row was deleted → 500s here. Use `find_by!` + rescue, or guard the nil." },
      { path:"app/models/user.rb", line:70, side:"RIGHT",
        body:"nit: name reads clearer as `active_members`." }
    ] } })
```
- **`line` is the line number in the file's new version** (the `RIGHT` side); use
  `side:"LEFT"` to comment on a removed/old line. The line must be part of the
  diff (inside a hunk) or GitHub 422s — comment on a changed line, or use the
  `body` summary for whole-file points.
- **`event`:** `COMMENT` (feedback, no gate) or `REQUEST_CHANGES` (something's
  wrong). `APPROVE` is gated on human sign-off — don't auto-approve.

## Rules
1. **Read intent first** (PR description + linked issue), then the diff in
   context. Correctness is relative to what the change is *for*.
2. **Label every comment** blocking / suggestion / nit, each with the fix or a
   real question. Lead with the one that matters most.
3. **Never `APPROVE` automatically, never merge.** `COMMENT`/`REQUEST_CHANGES`
   are advisory; the human approves + merges.
4. **Don't repeat** what's already commented; don't bury one real bug under ten
   nits.
5. **Inline lines must be in the diff** — whole-file / architectural points go in
   the review `body`, not a phantom line comment.

- **Re-review check:** `GET /repos/<o>/<r>/pulls/<n>/reviews` — if one of your reviews has `commit_id` == the PR's current `head.sha`, you already reviewed this state; skip it.
