# Quinn — QA engineer bundle

A quality engineer who thinks like an attacker of your assumptions. Designs test
plans for what's shipping, triages bug reports into clean prioritized issues,
verifies fixes against the original repro, and gives an honest
release-readiness call — before your customers do the testing for you.
**Advises and verifies; a human owns the ship decision.**

> Completes the engineering trio: **Cody** reviews the code, **Quinn** tests the
> behavior, **Patch** fixes the bugs.

## How it works

- **Test planning** via **GitHub** — reads each PR, assesses risk (critical
  flows, failure-prone shapes, blast radius), and posts a concrete plan on the
  PR: happy/failure/edge cases each with an expected result, plus what's *not*
  covered, named.
- **Bug triage** via **Linear + Sentry** — turns reports and new/spiking errors
  into clean, deduplicated issues (steps, expected/actual, env, frequency,
  severity from the matrix), and **verifies fixes** against the original repro +
  the error trend before recommending close.
- **Release readiness** — an explicit **ready / ready-with-risks / not-ready**
  call with the evidence attached and a post-deploy watchlist; delivered via
  **Slack** (native `slack.post`) + email. The go/no-go is `ask`-gated — a human
  ships.

## Connect at /integrations
- **GitHub** (required) — the changes.
- **Linear** (required) — the bug tracker.
- **Sentry** — production error truth.
- **Slack** (optional channel) — triage flags + the readiness call.

## Guardrails (baked in)
- **Never merges, edits, or reverts code** — tests and triages only.
- **Never closes an issue** without verification + the owner's sign-off.
- **The readiness call is a recommendation** — a human owns the release.
- **Unknowns are named** — untested areas are listed, never hidden.

Edit `knowledge/quality-playbook.md` to set your severity matrix, critical
flows, release bar, and verification standard without touching the persona.
