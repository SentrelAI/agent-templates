---
name: ledger-and-categorization
description: Use to maintain the ledger in Google Sheets — reading it, matching Stripe transactions to entries, categorizing against the chart of accounts, marking items reconciled, and building the cash / P&L-lite summary. Covers the Google Sheets API through the mcp__apps__request proxy. Records + reconciles; never invents entries to balance.
---

# Ledger & categorization (Google Sheets)

The ledger is the book of record. You maintain it through the **`request`** tool:

```
request({ provider:"google_sheets", method, path, query?, body? })
```

- **The base includes the version** (`https://sheets.googleapis.com/v4`) — paths
  are relative: `/spreadsheets/<id>/values/<range>`. Auth injected.

## Read the ledger
```
request({ provider:"google_sheets", method:"GET",
  path:"/spreadsheets/<id>/values/Ledger!A1:H",
  query:{ valueRenderOption:"UNFORMATTED_VALUE" } })
// → body.values = [[date, description, amount, category, stripe_id, reconciled, …]]
```
Read raw values (`UNFORMATTED_VALUE`) so amounts are numbers. Map columns by
header, not fixed position.

## Match + categorize
- **Match** each Stripe balance transaction (payments skill) to a ledger row by
  `stripe_id`/amount/date. Mark it reconciled (append/update a `reconciled`
  column) — never delete history.
- **Categorize** new entries against the **chart of accounts** in
  `bookkeeping-policy.md` (e.g. Revenue, Stripe fees, Software, Payroll,
  Contractors, Marketing). Consistency matters — the same vendor → the same
  category. If a transaction doesn't fit a category, **ask**, don't guess.

## Add / update entries (append; don't rewrite history)
```
request({ provider:"google_sheets", method:"POST",
  path:"/spreadsheets/<id>/values/Ledger!A1:append",
  query:{ valueInputOption:"USER_ENTERED" },
  body:{ values:[["2026-07-06","Stripe payout","-4182.50","Transfers","po_123","yes"]] } })
```
To mark an existing row reconciled, `PUT` its specific `reconciled` cell — don't
overwrite the whole row. **Never** invent an entry to make a balance tie out; a
gap is a flag, not a plug.

## Build the summary
Compute cash in / cash out by category, net, and A/R from the ledger; write it to
a dated summary tab (a *separate* tab, not over the ledger). Amounts to the
precision the data supports.

## Rules
1. **Read raw values**; map by header; amounts are numbers, not display strings.
2. **Append, don't rewrite** — the ledger is a record; correct with a new entry,
   don't erase history.
3. **Categorize consistently** against the chart of accounts; **ask** when
   unsure — never guess a category.
4. **Never invent an entry to balance** — an unreconciled gap is flagged, not
   plugged.
5. **Summary to a separate tab** — never overwrite the ledger.
