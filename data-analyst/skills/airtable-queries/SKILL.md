---
name: airtable-queries
description: Use to read structured records from Airtable for analysis — discovering bases/tables/fields, listing records with filters + views, paginating the full set, and writing analysis results to a separate results table (never the source). Covers the Airtable API through the mcp__apps__request proxy, filterByFormula, and the offset-pagination gotcha.
---

# Airtable queries

You read records and write results through the **`request`** tool:

```
request({ provider:"airtable", method, path, query?, body? })
```

- **Base is `https://api.airtable.com`** — paths include the version: `/v0/...`.
  Auth injected. Result `{ status, body }`.
- Airtable "base" id looks like `app…`; a table is referenced by name or id.

## Discover the schema
```
request({ provider:"airtable", method:"GET", path:"/v0/meta/bases" })
// → body.bases[] { id, name }
request({ provider:"airtable", method:"GET", path:"/v0/meta/bases/<appId>/tables" })
// → body.tables[] { id, name, fields:[{ name, type }] }
```
Know the field names + types before you query — they drive filters + parsing.

## List records (filter + view)
```
request({ provider:"airtable", method:"GET", path:"/v0/<appId>/<TableName>",
  query:{ filterByFormula:"AND({Status}='active', {MRR}>0)",
          view:"Grid view", pageSize:100, "sort[0][field]":"Created", "sort[0][direction]":"desc" } })
// → body.records[] { id, fields:{...}, createdTime }, body.offset?
```
`filterByFormula` uses Airtable's formula language: `{Field Name}` for fields,
`AND/OR/NOT`, `=`, comparisons, `DATETIME_DIFF`, etc. URL-encoding is handled by
the proxy.

## Paginate the FULL set (critical for correct analysis)
Airtable returns ≤100 records/page. If `body.offset` is present, there's more —
re-request with `query:{ offset:"<offset>" }` and concatenate until `offset` is
gone. **A partial read = wrong totals**, so always page to the end for counts/
aggregates.

## Write results to a SEPARATE table (not the source)
Create/append to a dedicated results table (make it once in the Airtable UI, or
write to a "Results" table), never editing source records:
```
request({ provider:"airtable", method:"POST", path:"/v0/<appId>/Results",
  body:{ records:[{ fields:{ Metric:"Churn", Value:"4.2%", Period:"2026-06", Note:"SMB-driven" } }] } })
```

## Rules
1. **Page to the end.** Follow `offset` until it's gone — partial reads produce
   wrong counts and aggregates.
2. **Know the field types** (from the schema) before filtering/parsing —
   dates, linked records, and rollups behave differently.
3. **Never modify source records.** Write analysis output to a separate Results
   table (`overwrite_source_data`/`delete_data` are blocked).
4. **`filterByFormula` on the server** beats pulling everything and filtering
   locally — smaller, faster, less to page through.
5. **Note data-quality issues** (blank required fields, dupes) as you read.
