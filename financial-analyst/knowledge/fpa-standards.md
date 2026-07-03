# FP&A standards

How {{agent_name}} runs finance analysis for {{company_name}}. **Edit this** to
lock in your definitions and thresholds. Fiscal basics: {{fiscal_setup}}.

## Metric definitions (edit — consistency is everything)
Define each of {{key_metrics}} exactly; {{agent_name}} labels which definition it
used whenever there's a choice. Starting points to replace with yours:
- **MRR** — sum of active subscription value normalized to monthly; state
  whether past_due counts and how discounts/credits are treated.
- **Churn** — logo vs. revenue churn are different metrics; define both, use
  them by name.
- **NRR** — (starting + expansion − churn − contraction) ÷ starting MRR,
  trailing month (or your window).
- **Burn** — net burn = cash out − cash in, cash basis, trailing-3mo average.
- **Runway** — cash ÷ net burn. State what cash counts (bank + Stripe balance?).
- **Gross margin / CAC / CAC payback** — define the cost allocations.

## Modeling standards
- Assumptions are **labeled cells**; no magic numbers in formulas.
- Forecasts in **dated tabs**, never overwritten; scenarios in scenario tabs.
- Actuals reconcile to source (Stripe + ledger) every cycle — the model never
  wins an argument with the actuals.
- Formulas over pasted values; anyone can trace any figure.

## Variance + escalation thresholds (edit to taste)
- **Material variance:** > ±10% or > $5k on a category, monthly — explain it.
- **Runway warning:** < 12 months → flag prominently. < 9 months → escalate as
  a decision item, with options.
- **Assumption break:** an actual runs >20% off its assumption for 2+ periods →
  update the assumption + call it out.

## The absolute rules (never break)
1. **Never move money** — read + model only.
2. **Never give tax or legal advice** — accounting-treatment questions go to the
   accountant with the analysis prepared.
3. **Never edit source data** (Stripe / source ledger) — writes go to model
   tabs only.
4. **Never present a forecast as a fact** — assumptions + ranges, always.

## Add your own
Your chart-of-accounts mapping to budget categories, the approved budget, the
hiring plan, the accountant's contact + what routes to them, and any board
reporting formats to match.
