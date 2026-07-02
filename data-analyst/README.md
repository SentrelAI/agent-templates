# Dana — data analyst bundle

A rigorous data analyst. Pulls the numbers from your spreadsheets and Airtable,
answers the real question honestly (with caveats, not spin), tracks the metrics
that matter, and delivers clear, reproducible reports — writing results to a
separate tab so nothing about your source data changes.

## How it works

- **Google Sheets** via the apps tool — reads ranges (raw values), and writes
  analysis **results to a new dated tab** (never the source) through the Sheets
  API (`mcp__apps__request({ provider:"google_sheets", … })`).
- **Airtable** — discovers the schema, queries records with `filterByFormula` +
  views, **pages the full set** (so counts are correct), and writes to a
  separate Results table.
- **Reporting** — computes the metric + trend correctly, structures an
  answer-first report with caveats + confidence, and delivers it to the team's
  **Slack** channel (native `slack.post`) or email.

The three skills (`spreadsheet-analysis`, `airtable-queries`,
`metrics-and-reporting`) are **knowledge** — real endpoints + the gotchas
(A1 ranges + `UNFORMATTED_VALUE`, Airtable `offset` pagination, answer-first
reporting).

## Connect at /integrations
- **Google Sheets** and/or **Airtable** (at least one required) — the data.
- **Slack** (channel, "Install in Slack" on the Channels tab) — report delivery.

## Guardrails (baked in)
- **Never overwrites or deletes source data** — results go to a separate tab/base.
- **Never shares numbers externally** without approval.
- Every answer ships with **caveats + confidence** and a **link to reproduce it**.

Edit `knowledge/analysis-standards.md` to lock in your **metric definitions**
(the #1 source of wrong reports) + data-quality and rigor standards, without
touching the persona.
