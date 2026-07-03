# Quality playbook

How {{agent_name}} judges severity and readiness for {{company_name}}'s
{{repositories}}. **Edit this** — it's the bar. Team's stated line:
{{quality_bar}}.

## Severity matrix (edit to your reality)
- **S1 — blocking:** data loss/corruption, security exposure, payments broken,
  a critical flow down for all users, no workaround. → urgent; blocks release.
- **S2 — major:** a critical flow broken for a segment, or a painful workaround
  exists. → high; blocks release unless explicitly waived by a human.
- **S3 — minor:** a non-critical path misbehaves; reasonable workaround. →
  medium; ships with a note.
- **S4 — cosmetic:** visual/copy issues, no functional impact. → low; ships.
Severity is defensible both directions — no inflation to win arguments, no
deflation to make a date.

## Critical flows (always regression-check — edit)
From {{product_surface}}. List them explicitly, e.g.:
1. Sign-up / sign-in
2. The core action (booking / checkout / send)
3. Payments + billing
4. Notifications going out
A change touching any of these gets a test plan, every time.

## Test-plan template
```
Risk: <what could break + blast radius>
Cases:
- happy: <do X> → <expect Y>
- failure: <the external call fails / record gone / permission denied> → <expect>
- edge: <boundary / double-submit / timezone / unicode> → <expect>
Regression: <adjacent critical flows to re-check>
Not covered: <named unknowns + why>
```

## Release bar
- Open S1 in scope → **not ready.**
- Open S2 in scope → not ready unless a human explicitly waives it (logged).
- S3/S4 → ship with notes.
- A critical flow untested for a risky change → at best ready-with-risks, named.

## The verification standard
"Fixed" means: original repro passes + Sentry event count stopped after deploy +
the adjacent path still works. Anything less is "merged," not "fixed."

## Add your own
Environments + how to reach them, test accounts/data, the deploy cadence, who
owns the ship decision, and any compliance/regression suites that must run.
