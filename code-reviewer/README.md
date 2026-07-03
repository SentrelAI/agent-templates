# Cody — code reviewer bundle

A senior code reviewer. Reads every pull request in context, leaves specific,
kind, actionable comments (bugs, security, missing tests, clarity), links the
work back to its issue, and flags anything touching code that's currently
throwing errors in production. It reviews and recommends — **a human still
merges**.

## How it works

- **PRs** via the apps tool — Cody reads pull requests + per-file diffs and
  posts a review with inline comments + a verdict through the **GitHub** REST
  API (`mcp__apps__request({ provider:"github", … })`). Verdicts are `comment`
  or `request-changes`; `approve` is human-gated and it **never merges or edits
  code**.
- **Issues** via **Linear** — links each PR to its issue, posts the review
  verdict, and files follow-up issues for deferred tech debt so nothing is lost.
- **Error context** via **Sentry** — cross-references a PR's changed files
  against active production errors, so risky areas get extra scrutiny and a
  "fix" can be verified against the real stack trace.
- Triggered by a **GitHub webhook** (review a PR the moment it opens/updates)
  and a twice-daily **queue sweep**; delivers a review digest by email.

The three skills (`pr-review`, `issue-tracking`, `error-context`) are
**knowledge** — real endpoints + the gotchas (the reviews-vs-comments endpoint,
inline-comment `line`/`side`, Linear GraphQL errors, Sentry culprit matching).

## Connect at /integrations
- **GitHub** (required) — read PRs/diffs, post review comments.
- **Linear** — link PRs to issues + file follow-ups.
- **Sentry** — active-error context on changed files.

## Guardrails (baked in)
- **Never merges** — always a human's call.
- **Never edits/pushes code** — Cody reviews, it doesn't rewrite.
- **Never auto-approves** — `approve` is gated; it posts `comment`/`request-changes`.

Edit `knowledge/review-rubric.md` to set the bar — what blocks vs. what's a nit,
test/migration/security policy, and the team's conventions — without touching
the persona.
