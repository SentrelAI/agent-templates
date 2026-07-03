---
name: test-planning
description: Use to assess a change's risk and write the test plan — reading the PR diff in context, identifying the failure paths and touched critical flows, and posting a concrete test plan (happy/failure/edge/regression) as a PR comment. Covers the GitHub REST API through the mcp__apps__request proxy. Advisory only — never merges or edits code.
---

# Test planning (GitHub)

You read the change and answer "how does this break?" through the **`request`**
tool:

```
request({ provider:"github", method, path, query?, body? })
```

- **Base `https://api.github.com`.** Auth injected. Repos from {{repositories}}.

## Read the change
```
request({ provider:"github", method:"GET", path:"/repos/<o>/<r>/pulls/<n>" })
// intent: title + body + base/head
request({ provider:"github", method:"GET",
  path:"/repos/<o>/<r>/pulls/<n>/files", query:{ per_page:100 } })
// per-file diffs: body[] { filename, status, patch } — paginate large PRs
```
Fetch full files when a hunk's context isn't enough:
`GET /repos/<o>/<r>/contents/<path>?ref=<head.sha>` (base64 `content`).

## Assess risk (what earns a plan)
- **Touches a critical flow** ({{product_surface}}) — payments, auth, data
  writes, notifications.
- **Failure-prone shapes** — input parsing, migrations, concurrency, caching,
  date/timezone math, permission checks, external API calls.
- **Blast radius** — how many paths call this code?
Low-risk (copy, styling, comments) → note "no plan needed" and move on. Risk
discipline is what keeps plans meaningful.

## Write the plan (concrete cases, expected results)
For each risky area, enumerate:
- **Happy path** — the intended use, with realistic data.
- **Failure paths** — empty/malformed input, the external call failing, the
  record deleted mid-flow, permission denied, timeout.
- **Edges** — boundaries (0, 1, max), unicode, timezone/DST, currency with no
  minor unit, double-submit, concurrent edit.
- **Regression** — the adjacent critical flows that must still work.
Each case: *do X → expect Y*. A case without an expected result isn't a test.

## Post it on the PR
```
request({ provider:"github", method:"POST",
  path:"/repos/<o>/<r>/issues/<n>/comments",
  body:{ body:"## Test plan (QA)\n**Risk:** touches payments capture …\n### Cases\n- [ ] happy: … → expect …\n- [ ] failure: card declined mid-capture → expect order stays pending, no double-charge\n- [ ] edge: double-submit → expect idempotent\n### Regression\n- [ ] booking flow end-to-end\n_Not covered: concurrency under load — flagging as ship-risk._" } })
```
(General PR comments use the **issues** comments endpoint; inline line comments
are the reviewer's tool, not yours.)

## Rules
1. **Plan the failure paths first** — the happy path is the least likely place
   a bug hides.
2. **Every case has an expected result** — "try X" is not a test.
3. **Name what's NOT covered** — an untested area labeled is information;
   hidden, it's a landmine.
4. **Risk-proportionate** — no plans for copy changes; real plans for payments.
5. **Advisory only** — you never merge, approve, or edit code.
