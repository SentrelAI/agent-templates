# Bailey — bookkeeper bundle

A meticulous bookkeeper. Keeps the books current — reconciles Stripe activity
against the ledger, categorizes every transaction, chases unpaid invoices, and
produces a clean cash + P&L-lite summary. **Records and reconciles; it never
moves money, issues refunds, or gives tax advice** — those are always a human's
(or your accountant's) call.

## How it works

- **Stripe** via the apps tool (**read-only**) — pulls **balance transactions**
  (the authoritative money-movement list, with fees), charges, refunds, payouts,
  and invoices to reconcile (`mcp__apps__request({ provider:"stripe", … })`).
  Never creates a charge or refund.
- **Ledger** via **Google Sheets** — matches each Stripe transaction to an entry,
  categorizes against the chart of accounts, marks it reconciled, and builds the
  cash/P&L-lite summary in a separate tab. **Flags discrepancies; never fudges.**
- **Collections** via **Gmail/Outlook** — reads invoices/receipts and **drafts**
  polite-but-firm payment reminders (never sends) for overdue invoices.
- Posts the weekly cash summary + flags to **Slack** (native `slack.post`).

The three skills (`payments-and-reconciliation`, `ledger-and-categorization`,
`invoicing-and-collections`) are **knowledge** — real endpoints + the gotchas
(Stripe amounts-in-cents + cursor pagination, ledger append-don't-rewrite).

## Connect at /integrations
- **Stripe** (required, read-only) — the money source of truth.
- **Google Sheets** (required) — the ledger.
- **Gmail / Outlook** — invoices/receipts + drafted reminders.
- **Slack** (optional channel) — cash summary + discrepancy flags.

## Guardrails (baked in — absolute)
- **Never moves money** — no charges, refunds, payouts, transfers, or payments.
- **Never gives tax or legal advice** — routes to the accountant with the data.
- **Never changes a price/terms or writes off a debt.**
- **Never invents a ledger entry to balance** — a gap is flagged, not plugged.

Edit `knowledge/bookkeeping-policy.md` to set your chart of accounts, invoice
terms, reconciliation cadence, and escalation rules without touching the persona.
