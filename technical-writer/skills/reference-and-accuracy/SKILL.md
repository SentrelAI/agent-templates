---
name: reference-and-accuracy
description: Use to guarantee docs are correct — verifying every documented detail against the source code, writing runnable examples, generating a clean changelog from merged PRs, and catching docs that have drifted out of sync. The accuracy layer that makes docs trustworthy.
---

# Reference & accuracy (method)

This is the discipline that separates docs developers trust from docs they learn
to ignore. The rule is simple: **the code is the source of truth; verify, never
invent.**

## Verify every detail against the code
Before a fact goes in a doc, confirm it in the source (code-context skill):
- **Names** — exact casing/spelling of params, fields, endpoints (`snake_case`
  vs. `camelCase` matters).
- **Types + required/optional + defaults** — from the schema/signature, not
  assumption.
- **Return shape + error cases** — what actually comes back, including nulls and
  failure paths.
- **Behavior** — read the implementation; don't document what it "should" do.
Note the source (`verified against invoices.rb:88`). If you can't confirm it,
write `TODO: verify` and flag it — never guess.

## Write examples that actually run
- Realistic values, correct field names, a call a reader could paste and run.
- Show the response too (real shape). Include the auth header / setup a first-time
  reader needs.
- If you can trace it, the example should match a real request the code accepts.

## Generate a clean changelog (from merged PRs / releases)
From the PRs/releases since the last version (code-context skill), write a
human changelog grouped by **Added / Changed / Fixed / Deprecated / Breaking** —
in terms of what a *user* experiences, not internal refactors. Call out breaking
changes loudly with the migration path.

## Catch drift (docs that now lie)
When the code changed but the doc didn't, the doc is now wrong — the worst state.
On each sync pass, compare shipped changes against the docs and **flag every
stale doc**, with the specific line that's now incorrect + the fix. A flagged
stale doc gets a drafted correction.

## Rules
1. **Verify against the source** — every name, type, default, return, behavior.
2. **Never invent** — unverifiable → `TODO: verify` + flag, not confident prose.
3. **Runnable examples** — realistic, correct, would-actually-work.
4. **Changelog in user terms** — Added/Changed/Fixed/Deprecated/Breaking; breaking
   changes loud + with a migration path.
5. **Hunt drift** — a stale doc is worse than a missing one; flag + fix it.
