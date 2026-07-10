---
name: financial-modeling
description: Use to build and maintain the financial model in Google Sheets — reading actuals from the ledger, keeping assumptions as labeled cells, writing the forecast to dated tabs, and running what-ifs in scenario tabs (never in the live forecast). Covers the Google Sheets API through the mcp__apps__request proxy and the modeling discipline that keeps a model trustworthy.
---

# Financial modeling (Google Sheets)

The model is a Sheet anyone can open, follow, and challenge. You maintain it
through the **`request`** tool:

```
request({ provider:"google_sheets", method, path, query?, body? })
```

- **The base includes the version** (`https://sheets.googleapis.com/v4`) — paths
  are relative: `/spreadsheets/<id>/values/<range>`. Auth injected.

## Read actuals (raw numbers, whole range)
```
request({ provider:"google_sheets", method:"GET",
  path:"/spreadsheets/<id>/values/Actuals!A1:N",
  query:{ valueRenderOption:"UNFORMATTED_VALUE" } })
```
`UNFORMATTED_VALUE` so math runs on numbers, not display strings. Map columns by
header. Open-ended ranges (`A1:N`) so a truncated read never silently drops rows.

## The model's shape (keep this discipline)
- **Actuals** — reconciled to source (Stripe + ledger). Never hand-edited to fit.
- **Assumptions** — every driver is a *labeled cell* (growth %, churn %, hire
  plan, price): change the cell, the forecast updates. No magic numbers inside
  formulas.
- **Forecast** — formulas off actuals + assumptions, in **dated tabs**
  (`Forecast 2026-07`), never overwritten — forecast-vs-forecast comparison is
  how you grade your own assumptions.
- **Scenarios** — what-ifs live in their own tabs, never in the live forecast.

## Write the forecast / a scenario (new dated tab)
```
// 1) add the tab
request({ provider:"google_sheets", method:"POST", path:"/spreadsheets/<id>:batchUpdate",
  body:{ requests:[{ addSheet:{ properties:{ title:"Forecast 2026-07" } } }] } })
// 2) write it — USER_ENTERED so formulas (=B4*(1+$B$1)) and numbers parse
request({ provider:"google_sheets", method:"PUT",
  path:"/spreadsheets/<id>/values/'Forecast 2026-07'!A1",
  query:{ valueInputOption:"USER_ENTERED" },
  body:{ values:[["driver","value","note"],
                 ["MoM growth","0.08","assumption — base case"],
                 ["Monthly churn","0.03","assumption — trailing 3mo avg"],
                 ["MRR (m+1)","=Actuals!N2*(1+B2)-Actuals!N2*B3","formula"]] } })
```
Write **formulas, not pasted results**, wherever the sheet can compute — that's
what makes the model reproducible and challengeable.

## Rules
1. **Assumptions are labeled cells** — never buried in formulas.
2. **Dated tabs; never overwrite** a prior forecast — history is the scorecard.
3. **Scenarios in scenario tabs** — the live forecast stays clean.
4. **Formulas over pasted values** — reproducible or it isn't a model.
5. **Never edit the Actuals/source tabs** to make something fit (blocked) — a
   mismatch is a flag, not an editing task.

## Errors & pagination (standard)

- **401/403** — the connection is broken or missing: stop and tell the owner to
  reconnect the app at /integrations. Don't retry.
- **429** — back off ~30s and retry once; still failing → finish other work and
  pick this up next run. Use smaller pages.
- **5xx twice** — report the failure plainly. Never fabricate data you couldn't fetch.
- **Pagination** — never conclude "nothing new" from page one. Gmail/Calendar:
  `nextPageToken` → `pageToken`. Notion: `has_more`/`next_cursor` → `start_cursor`.
  GitHub: `Link: rel="next"`. Microsoft Graph: `@odata.nextLink`. Stripe:
  `has_more` + `starting_after`. Linear GraphQL: `pageInfo { hasNextPage endCursor }`.
