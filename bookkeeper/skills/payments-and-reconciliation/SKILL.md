---
name: payments-and-reconciliation
description: Use to pull Stripe money movement for reconciliation — balance transactions (the authoritative list, with fees), charges, refunds, payouts, and invoice payments — and match each to the ledger. READ-ONLY; never creates charges or refunds. Covers the Stripe API through the mcp__apps__request proxy, cursor pagination, and the amounts-in-cents gotcha.
---

# Payments & reconciliation (Stripe)

You read Stripe to reconcile the books through the **`request`** tool. **This is
read-only** — you never create a charge, refund, or payout (money movement is
blocked).

```
request({ provider:"stripe", method:"GET", path, query? })
```

- **Base is `https://api.stripe.com`** — paths are `/v1/...`. Auth (a key) is
  injected. Result `{ status, body }`.

## The authoritative source: balance transactions
`balance_transactions` is *the* reconciliation list — every money movement (a
charge, refund, payout, fee, adjustment) with its **net** and **fee**:
```
request({ provider:"stripe", method:"GET", path:"/v1/balance_transactions",
  query:{ limit:100, "created[gte]":<unix ts since last recon> } })
// → body.data[] { id, type, amount, fee, net, currency, created, source, description }
```
`type` tells you what it is (`charge`, `refund`, `payout`, `stripe_fee`,
`adjustment`). This is what the ledger should tie to line-for-line.

## Supporting detail (as needed)
- Charges: `GET /v1/charges?limit=100&created[gte]=…`
- Refunds: `GET /v1/refunds?limit=100`
- Payouts: `GET /v1/payouts?limit=100` (bank transfers out)
- Invoice payments: `GET /v1/invoices?status=paid&limit=100`

## Two must-know gotchas
1. **Amounts are in the smallest currency unit (cents).** `amount: 4182` = $41.82.
   Divide by 100 (per-currency) — mismatching this by 100× is the classic bug.
2. **Pagination is cursor-based.** If `body.has_more` is true, re-request with
   `query:{ starting_after:"<last id>" }` until it's false. A partial read means
   an incomplete reconciliation.

## Reconcile
Match each balance transaction to exactly one ledger entry (ledger skill). **Flag,
never fudge:**
- A Stripe transaction with no ledger entry (or vice-versa).
- Amount/fee mismatches.
- Duplicates. Refunds with no matching charge.
Surface flags with the ids + amounts; a human resolves them.

## Rules
1. **Read-only.** Never POST to Stripe — no charges, refunds, or payouts (blocked).
2. **Cents, not dollars.** Convert `amount`/`fee`/`net` from the smallest unit.
3. **Page to the end** (`has_more` + `starting_after`) — partial = incomplete
   reconciliation.
4. **Balance transactions are the source of truth** — reconcile against them,
   including fees.
5. **Flag, don't fudge** — a mismatch is surfaced, never smoothed over.
