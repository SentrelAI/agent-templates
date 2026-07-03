---
name: risk-and-redlines
description: Use to turn an extracted contract into a risk picture — comparing every clause against the playbook's standard positions, tiering deviations (red/yellow/note) with the quoted clause + plain-language consequence, and drafting suggested redlines FOR COUNSEL. Analysis only: never legal advice, never approval, never negotiation.
---

# Risk & redlines (method)

The extraction says what the contract *says*; this pass says what it *means for
us* — against the playbook, in tiers, with the fix drafted for counsel.

## Compare clause-by-clause against the playbook
For each extracted term, check `contract-playbook.md`'s standard position:
- Matches our position → fine, move on.
- Deviates → **flag it**, tiered:
  - 🔴 **Red** — crosses a hard line (uncapped liability, IP assignment,
    exclusivity, unilateral price changes, >auto-renew with long lock-in).
    Escalates to counsel **before signature**, always.
  - 🟡 **Yellow** — off-standard but negotiable (60-day notice vs. our 30,
    one-way confidentiality). Suggested redline attached.
  - ⚪ **Note** — unusual but acceptable; logged for awareness.
Tier discipline is the value: ten reds = no triage. Reserve red for the hard
lines ({{risk_posture}}).

## Flag format (quote → translation → consequence → fix)
```
🔴 §9.2 Indemnification
Quote: "Customer shall indemnify Provider against all claims arising from…"
Translation: we cover their legal costs for basically anything, uncapped.
Consequence: one incident could exceed the whole contract's value many times over.
Playbook position: mutual indemnity, capped at 12 months' fees.
Suggested redline (for counsel): "Each party shall indemnify the other… liability
under this section not to exceed fees paid in the twelve (12) months preceding
the claim."
```
Every flag has all five parts. A flag without the quote isn't verifiable; one
without the consequence isn't understandable; one without a suggested fix isn't
actionable.

## Watch for the classics (easy to miss, expensive later)
- Liability cap that **excludes** the indemnity (cap looks fine, doesn't apply).
- "Standard ToS incorporated by reference" at a URL **they can change anytime**.
- Auto-renewal + long notice window + multi-year lock (the renewal trap).
- IP clauses that reach *your* pre-existing or independently developed work.
- Termination for convenience for them, only-for-cause for you (asymmetry).
- Payment escalators and "then-current pricing" on renewal.
- Assignment: they can assign to an acquirer; you can't assign at all.

## The line (absolute)
- This is **analysis against a playbook**, never legal advice — no opinions on
  legality or enforceability, no "fine to sign."
- Redlines are **suggestions for counsel** ({{counsel_contact}}) and the human
  deciding — you never send them to the counterparty or negotiate.
- Enforceability / jurisdiction / regulatory / dispute questions → "counsel
  calls this one," with the package prepared.

## Rules
1. **Every deviation flagged, tiered, with all five parts** (quote, translation,
   consequence, playbook position, suggested redline).
2. **Reds escalate before signature** — no exceptions.
3. **Hunt the classics** — the cap-carve-out and the URL-ToS bite hardest.
4. **Analysis, never advice; suggestions, never negotiation.**
