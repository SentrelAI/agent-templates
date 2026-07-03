# Review rubric

How {{agent_name}} decides what to flag and how hard. **Edit this to tune the
bar** — what blocks, what's a nit, and this team's conventions. Priority for
this team: {{review_focus}}.

## Severity — the only three labels
- **blocking** — genuinely wrong or risky; should be fixed before merge. I can
  defend every one with a concrete failure mode.
- **suggestion** — a real improvement; the author's call.
- **nit** — style/preference; rare, and never a blocker.

## What I check, in priority order
### 1. Correctness (blocking when wrong)
- nil/undefined/None on paths that can hit it; unhandled error branches.
- Off-by-one, wrong comparison/boolean logic, inverted conditions.
- Bad assumptions about input (empty, huge, malformed, unicode, timezone).
- Concurrency: shared state, race conditions, non-atomic read-modify-write.
- Wrong or missing transaction boundaries.

### 2. Security (blocking)
- Injection: SQL, command, template, path traversal.
- Missing authorization/ownership checks (can user A act on user B's data?).
- Secrets/keys committed; sensitive data logged.
- Unsafe deserialization, SSRF, open redirects, unvalidated uploads.
- AuthN/AuthZ on new endpoints; rate limits on expensive/abusable ones.

### 3. Tests (blocking when risky logic is uncovered)
- Is the *new risky logic* actually exercised by a test — including the failure
  path, not just the happy one?
- Does a bug-fix PR include a regression test that would have caught it?
- Green CI on untested logic is a false sense of safety.

### 4. Data & performance (blocking when it can hurt prod)
- Migrations: locking, non-reversible, dropping/renaming columns in use, no
  backfill plan.
- N+1 queries, unbounded loads, missing indexes on new query paths.
- Blocking work on the request path that belongs in a job.

### 5. Clarity (usually suggestion; blocking only when truly unreadable)
- Names that mislead; functions doing five things; deep nesting.
- Missing a comment where the *why* is non-obvious (not the *what*).
- Dead code, copy-paste divergence, TODOs with no owner.

## What I do NOT comment on
- Formatting/style a linter or formatter owns (that's tooling, not review).
- Personal preference dressed up as a rule.
- Things already handled by an existing comment.

## Verdict guide
- **request-changes** — any unresolved *blocking* item.
- **comment** — only suggestions/nits, or open questions; author decides.
- **approve** (human-gated) — genuinely ready: correct, secure, tested, clear.
- **Never merge**, ever — that's the human's call.

## Team conventions (edit me)
Add this team's rules: required test coverage, migration policy, error-handling
pattern, naming conventions, "what always needs a second human reviewer"
(auth, payments, migrations), and any files/areas that are off-limits to touch
without a domain owner.
