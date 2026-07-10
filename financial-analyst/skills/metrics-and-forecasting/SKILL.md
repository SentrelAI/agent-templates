---
name: metrics-and-forecasting
description: Use to compute the SaaS metrics from real Stripe data — MRR from subscriptions, churn and NRR from month-over-month movement, and the revenue inputs the forecast needs. READ-ONLY on Stripe. Covers the Stripe API through the mcp__apps__request proxy, the cents + cursor-pagination gotchas, and the metric math that's easy to get subtly wrong.
---

# Metrics & forecasting (Stripe)

Revenue truth comes from Stripe. You read it through the **`request`** tool —
**read-only, always**:

```
request({ provider:"stripe", method:"GET", path, query? })
```

- **Base `https://api.stripe.com`**, paths `/v1/...`. Auth injected.
- **Amounts are in cents** — divide by 100 (per currency) before any math.
- **Cursor pagination:** while `body.has_more`, re-request with
  `starting_after:"<last id>"`. A partial pull = a wrong MRR. Page to the end.

## MRR — from active subscriptions
```
request({ provider:"stripe", method:"GET", path:"/v1/subscriptions",
  query:{ status:"active", limit:100, "expand[]":"data.items.data.price" } })
// each item: price.unit_amount × quantity, normalized to monthly:
//   interval "month" → as-is; "year" → /12 (mind interval_count)
```
Sum normalized item amounts across all active subscriptions = MRR. Keep the
per-customer breakdown — churn/NRR math needs it. Include `status:"past_due"`
separately if the policy counts it (state which you used).

## Churn + NRR — month-over-month movement
Compare this month's per-customer MRR map to last month's:
- **Churned** — had MRR then, zero now.
- **Contraction / expansion** — same customer, lower / higher MRR.
- **New** — MRR now, none before.
Then: **logo churn** = churned ÷ starting customers · **revenue churn** =
churned+contraction MRR ÷ starting MRR · **NRR** = (starting + expansion −
churn − contraction) ÷ starting. Label which definition you used — mixing
logo and revenue churn is the classic silent error.

## Cash timing (when the forecast needs cash, not bookings)
- Actual cash movement: `GET /v1/balance_transactions` (has fees + `net`).
- Payouts to the bank: `GET /v1/payouts`.
MRR is a *bookings* view; cash lags it (billing cycles, failed payments,
payout delay). Never treat MRR as cash-in for a runway calc — use balance
transactions for that.

## Feed the forecast
Write the computed actuals (MRR, churn, NRR, cash-in) into the model's Actuals
tab (financial-modeling skill) with the computation date — the forecast's
formulas take it from there.

## Rules
1. **Read-only on Stripe** — never POST; money movement is blocked.
2. **Cents → dollars** everywhere; **page to the end** everywhere.
3. **Normalize intervals** (yearly/quarterly → monthly) before summing MRR.
4. **Label the definition** — logo vs. revenue churn, what counts as active.
5. **MRR ≠ cash.** Use balance transactions for cash/runway math.

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
