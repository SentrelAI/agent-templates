---
name: release-readiness
description: Use to assemble the honest go/no-go read before a release — what shipped, what was tested (and what wasn't), open bugs by severity, Sentry error trends, and an explicit ready / ready-with-risks / not-ready call with what to watch post-deploy. A recommendation for a human, never the decision. Synthesizes GitHub + Linear + Sentry.
---

# Release readiness (method)

The point isn't a gate — it's that the human who ships does so *informed*. You
assemble the evidence and make an explicit call.

## Assemble the evidence
1. **What's shipping** — merged PRs since the last release
   (`GET /repos/<o>/<r>/pulls?state=closed&base=main&sort=updated&direction=desc`,
   keep `merged_at` since the last tag; or compare
   `GET /repos/<o>/<r>/compare/<last-tag>...main`).
2. **What was tested** — which of those changes had test plans, and the case
   results. **What wasn't** — the unverified areas, named.
3. **Open bugs by severity** — Linear query for open issues touching the release
   scope; S1/S2s are the headline.
4. **Error baseline** — Sentry trend going in (`statsPeriod:"7d"`): a release on
   top of a rising error curve is a different bet than on a quiet week.

## Make the call (explicit, one of three)
- **Ready** — risky changes tested, no open S1/S2 in scope, errors stable.
- **Ready with risks** — shippable, but named risks ride along ("S3 pagination
  bug ships; concurrency path unverified"). Most real releases live here.
- **Not ready** — an open blocker per the bar ({{quality_bar}}), or a critical
  path unverified. Say exactly what would flip the call.

## The readiness note (what the human gets)
```
Release readiness — v2.14
Call: READY WITH RISKS
Shipping: 9 PRs (3 risky: payments capture, session refactor, migration)
Tested: payments plan 12/12 pass · session 8/8 · migration dry-run ok
Not covered: concurrent-edit path (no test env) — watch for it
Open bugs in scope: 0 S1 · 0 S2 · 2 S3 (linked)
Errors: stable 7d; watch checkout_500 (fixed this release — confirm it drops)
Post-deploy watch: Sentry checkout_500 count, payments success rate, p95
```
Deliver it where the team decides (Slack `slack.post` + email). The go/no-go
recommendation is `ask`-gated — **a human ships.**

## After the deploy — close the loop
Check the post-deploy watch items within the first hours: did the fixed error
actually stop? did anything new spike? Report back either way — "quiet after 4h"
is information too.

## Rules
1. **One of three explicit calls** — never a shrug. "Ready with risks" must name
   the risks.
2. **Unknowns are named** — untested ≠ fine; untested = listed.
3. **Evidence attached** — the call carries what was tested, what's open, and
   the error baseline; no rubber stamps.
4. **A human ships** — your call informs the decision; it is never the decision.
5. **Follow the release** — the readiness job ends after the post-deploy check,
   not at the merge.
