---
name: bug-triage
description: Use to turn bug reports and new Sentry errors into clean, prioritized, deduplicated Linear issues — reproduce, capture exact steps + expected/actual + environment, assign a defensible severity, and verify fixes against the original repro + error trend before recommending close. Covers Linear GraphQL + Sentry through the mcp__apps__request proxy.
---

# Bug triage (Linear + Sentry)

A bug's fate is decided by the quality of its report. You triage through the
**`request`** tool.

## Catch what production is saying (Sentry)
```
request({ provider:"sentry", method:"GET",
  path:"/organizations/<org>/issues/",
  query:{ query:"is:unresolved is:for_review", statsPeriod:"24h", sort:"freq", limit:25 } })
// → body[] { id, title, culprit, count, userCount, permalink, firstSeen }
```
- **Base `https://sentry.io/api/0`.** New + spiking errors are triage input.
- Detail for a repro: `GET /organizations/<org>/issues/<id>/events/latest/` —
  the stack trace, breadcrumbs, and request context tell you the steps.
- **Read-only** — never resolve/ignore in Sentry; the tracker is where state
  lives.

## Dedup BEFORE filing (Linear)
```
request({ provider:"linear", method:"POST", path:"/graphql",
  body:{ query:"query($q:String!){ issueSearch(query:$q){ nodes{ id identifier title state{name} url } } }",
         variables:{ q:"<key symptom>" } } })
```
Match found → comment `+1` with the new evidence instead of filing a duplicate.
(Base `https://api.linear.app`, always POST `/graphql`, check `body.errors`.)

## File it clean
```
request({ provider:"linear", method:"POST", path:"/graphql",
  body:{ query:"mutation($input:IssueCreateInput!){ issueCreate(input:$input){ success issue{ identifier url } } }",
    variables:{ input:{ teamId:"<teamId>",
      title:"Checkout 500s when the cart has a deleted product",
      description:"**Severity:** S1 — blocks purchase\n**Steps (repro 3/3):**\n1. Add product → admin deletes it → checkout\n**Expected:** graceful 'item unavailable'\n**Actual:** 500 (Sentry: <link>)\n**Env:** prod, Chrome 126\n**First seen:** 2026-07-02 · 41 events / 12 users",
      priority:1 } } } })
```
Severity from the matrix in `quality-playbook.md` — defensible, both directions.
Priority mapping: S1→urgent(1), S2→high(2), S3→medium(3), S4→low(4).

## Verify the fix (merged ≠ fixed)
1. Re-run the **original repro** against the fixed build.
2. Sentry trend: `query:"is:unresolved <error>"` — did events stop after the
   deploy? A fix with the error still firing isn't a fix.
3. Spot-check the adjacent path for the classic regression.
4. Post the verification evidence on the issue; closing is `ask` — recommend,
   with the evidence, and let the owner close.

## Rules
1. **Reproduce before filing** — or file "can't repro yet" with what you tried.
2. **Dedup first** — a `+1` beats a duplicate, every time.
3. **The report is the product:** steps, expected/actual, env, frequency,
   Sentry link, severity. Write the report you'd want to receive.
4. **Verify against the original repro + the error trend** before recommending
   close.
5. **Sentry is read-only; Linear is where state lives.**
