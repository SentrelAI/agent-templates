---
name: spreadsheet-analysis
description: Use to read data from Google Sheets and write analysis RESULTS back to a separate tab (never overwriting source) — listing tabs, reading ranges/values, batch-reading, adding a results tab, and appending computed rows. Covers the Google Sheets API v4 through the mcp__apps__request proxy, A1 range syntax, and the value-input gotcha.
---

# Spreadsheet analysis (Google Sheets)

You read source data and write reproducible results through the **`request`**
tool:

```
request({ provider:"google_sheets", method, path, query?, body? })
```

- **The base already includes the version** (`https://sheets.googleapis.com/v4`),
  so paths are relative: `/spreadsheets/<id>/values/<range>` — **not**
  `/v4/...`. Auth injected. Result `{ status, body }`.
- The `spreadsheetId` is in the sheet URL (`/d/<id>/edit`).

## See what's in the workbook
```
request({ provider:"google_sheets", method:"GET", path:"/spreadsheets/<id>",
  query:{ fields:"sheets.properties(title,gridProperties)" } })
// → body.sheets[].properties.title = the tab names + row/col counts
```

## Read a range
A1 notation: `'<Tab>'!A1:D` (open-ended row range reads the whole column):
```
request({ provider:"google_sheets", method:"GET",
  path:"/spreadsheets/<id>/values/Revenue!A1:H",
  query:{ majorDimension:"ROWS", valueRenderOption:"UNFORMATTED_VALUE" } })
// → body.values = [[row], [row], ...]  (first row usually headers)
```
Use `valueRenderOption:"UNFORMATTED_VALUE"` so you get raw numbers/dates, not
formatted strings you'd have to parse. Batch-read multiple ranges with
`/spreadsheets/<id>/values:batchGet?ranges=A!A:C&ranges=B!A:C`.

## Write RESULTS to a NEW tab (never the source)
1. Add a dated results tab:
```
request({ provider:"google_sheets", method:"POST", path:"/spreadsheets/<id>:batchUpdate",
  body:{ requests:[{ addSheet:{ properties:{ title:"results 2026-07-06" } } }] } })
```
2. Write the computed table into it:
```
request({ provider:"google_sheets", method:"PUT",
  path:"/spreadsheets/<id>/values/'results 2026-07-06'!A1",
  query:{ valueInputOption:"USER_ENTERED" },
  body:{ values:[["metric","this","prior","Δ"],["MRR",42000,40100,"+4.7%"]] } })
```
`valueInputOption:"USER_ENTERED"` lets Sheets parse numbers/percent/dates;
`RAW` stores literal strings. **Append** (don't overwrite) with
`/values/<range>:append`.

## Rules
1. **Read raw** (`UNFORMATTED_VALUE`) so math is on numbers, not display strings.
2. **First row is usually headers** — map columns by name, don't hardcode
   positions that shift.
3. **Never write to the source tab.** Create a separate, dated results tab and
   write there — the source stays untouched (`overwrite_source_data` is blocked).
4. **Check the range covers all the data** (open-ended `A1:H`, not `A1:H100`) or
   you'll silently analyze a truncated set.
5. **Note gaps/dupes** you see while reading — data quality is step one.
