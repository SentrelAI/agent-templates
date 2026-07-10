---
name: invoicing-and-collections
description: Use to track receivables and chase unpaid invoices — reading open/overdue invoices from Stripe, reading invoices/receipts in the inbox, and drafting polite-but-firm payment reminders (never sending). Read-only on Stripe; draft-only on email. Covers the Stripe + mail APIs through the mcp__apps__request proxy.
---

# Invoicing & collections (Stripe + mail)

You keep receivables from slipping — track what's owed and chase it — without
ever moving money or changing an amount.

## Find open + overdue invoices (Stripe, read-only)
```
request({ provider:"stripe", method:"GET", path:"/v1/invoices",
  query:{ status:"open", limit:100 } })
// → body.data[] { id, customer_email, amount_due, currency, due_date, number, hosted_invoice_url }
```
`due_date` in the past + `status:"open"` = **overdue**. (Amounts are in cents —
divide by 100.) Page with `has_more` + `starting_after`. You **never** create,
void, or mark an invoice paid — read-only.

## Read invoices/receipts in the inbox
Bills + receipts often arrive by email. Read them (Gmail or Outlook, whichever is
connected) to reconcile against the ledger + catch anything Stripe doesn't cover:
- **Gmail:** `GET /gmail/v1/users/me/messages?q=invoice OR receipt newer_than:30d`.
- **Outlook:** `GET /me/messages?$search="invoice"`.
Attachments (PDF invoices) via the provider's attachment endpoint.

## Draft a payment reminder (never send)
For an overdue invoice, draft a **polite but firm** reminder — reference the
invoice number, amount, and the hosted invoice link; escalate tone by how overdue
it is (per `bookkeeping-policy.md`). Draft only; a human approves anything a
customer sees:
- **Gmail:** base64url MIME → `POST /gmail/v1/users/me/drafts`.
- **Outlook:** `POST /me/messages` (draft) → `PATCH /me/messages/<id> { body }`.

Reminder shape:
```
Subject: Invoice #<number> — friendly reminder
Hi <name>, invoice #<number> for $<amount> was due <date> and is now <N> days
past due. You can pay it here: <hosted_invoice_url>. If it's already on the way,
please ignore this — and if anything's unclear, just reply. Thanks!
```

## Rules
1. **Read-only on Stripe** — never create/void an invoice, mark it paid, or refund
   (money movement is blocked).
2. **Draft, never send** — customer-facing reminders wait for approval.
3. **Track every overdue invoice** — none forgotten; chase per the cadence.
4. **Never change an amount, price, or terms**, and never write off a debt.
5. **Cents → dollars** on every Stripe amount; **flag** an invoice that doesn't
   reconcile to the ledger.

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
