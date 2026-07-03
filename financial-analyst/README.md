# Finn — financial analyst (FP&A) bundle

An FP&A analyst who tells you what the numbers mean and where they're headed.
Builds the forecast, tracks budget vs. actual with every material variance
*explained*, watches runway and unit economics, and models the "what if" —
grounded in real Stripe + ledger data. **Analyzes and models; it never moves
money, edits source data, or gives tax/legal advice.**

> Pairs with **Bailey** (the bookkeeper): Bailey records what happened; Finn
> projects what happens next. Bailey keeps the books; Finn keeps the forecast.

## How it works

- **Revenue truth** via **Stripe** (read-only) — computes MRR from active
  subscriptions (normalized to monthly), churn/NRR from month-over-month
  per-customer movement, and cash from balance transactions — with the classic
  gotchas handled (cents, cursor pagination, MRR ≠ cash).
- **The model** via **Google Sheets** — actuals → labeled assumption cells →
  forecast formulas, in dated tabs (never overwritten); what-ifs in scenario
  tabs. Reproducible: every figure traces to a cell and a source.
- **Reporting** — budget-vs-actual with the *why* per material variance, runway
  on cash (plan vs. trend), and a scannable weekly update via **Slack**
  (native `slack.post`) / email. Board/investor numbers need a human yes.

The three skills (`financial-modeling`, `metrics-and-forecasting`,
`reporting-and-variance`) are **knowledge** — real endpoints, the metric math
that's easy to get subtly wrong, and the modeling discipline that keeps a
forecast trustworthy.

## Connect at /integrations
- **Stripe** (required, read-only) — revenue actuals.
- **Google Sheets** (required) — the model + forecast.
- **Slack** (optional channel) — the weekly finance update.

## Guardrails (baked in — absolute)
- **Never moves money**; **never edits source data** (model tabs only).
- **Never gives tax or legal advice** — routes to the accountant with the
  analysis prepared.
- **Never presents a forecast as fact** — assumptions documented, ranges over
  false precision, downside stated early.

Edit `knowledge/fpa-standards.md` to lock in your metric definitions, variance
thresholds, and runway warnings without touching the persona.
