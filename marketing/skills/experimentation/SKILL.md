---
name: experimentation
description: Use when running or interpreting an A/B test, a creative test, a landing-page optimization, or a budget-allocation decision across channels. Covers the experiment loop (hypothesis → test → read → iterate), the statistics that decide when a result is real (and the peeking traps that fake it), contextual-bandit landing-page routing, creative testing at scale via the Meta A/B + lift tools, and MMM-vs-MTA for cross-channel budget. The rule throughout: a test that's read wrong is worse than no test, and any change that spends or publishes is a draft for human approval.
---

# Experimentation & CRO

Spend reporting (the measurement skill) tells you what *happened*.
Experimentation tells you what to *change* — and whether a change actually
worked or you just got lucky. This is where most "growth" goes wrong: tests get
stopped the moment they look good, and a coin-flip gets shipped as a win.

The single rule everything here serves: **a result is only real when the method
that produced it was valid.** Reading a test wrong is worse than running none —
it manufactures false confidence and you scale a loser. You **draft** experiments
and **interpret** them honestly; a human **approves** anything that publishes a
variant or moves budget.

---

## 1. The loop

```
Hypothesis  →  Test design  →  Run to validity  →  Read honestly  →  Iterate
```

1. **Hypothesis** — a specific, falsifiable claim with a direction and a why.
   Not "the new headline is better" but "leading with price will lift CTR for
   cost-sensitive segments because the objection is cost." A hypothesis names
   the metric, the expected direction, and the reason.
2. **Test design** — one change at a time (or a properly-powered multivariate
   design), a single primary metric decided *before* you start, and a stopping
   rule decided *before* you start (see §2). Changing the primary metric or the
   stopping rule after seeing data is how you lie to yourself.
3. **Run to validity** — let it reach the validity bar for the method you chose.
   Don't stop early just because it looks good (§2).
4. **Read honestly** — significance is necessary, not sufficient; also weigh
   effect size, segment behavior, and whether the result is plausible.
5. **Iterate** — a confirmed winner becomes the new baseline; a null result is
   still information (the lever doesn't move this audience — try a different one).

---

## 2. The statistics — when a result is real (and the peeking traps)

Pick the method *before* you start, because each has a different rule for when
you're allowed to look and stop. Mixing them up is the #1 way agents ship noise.

| Method | When you may stop | The trap |
|---|---|---|
| **Frequentist** (t-test, fixed-horizon) | **Only at a pre-set sample size.** | **Peeking invalidates it.** Every interim glance at a fixed-horizon test inflates the false-positive rate. Compute the sample size up front; don't look at significance until you hit it. |
| **Sequential** (always-valid / group-sequential) | **Any time** — early or extended. | None for stopping — these are *designed* for continuous monitoring. This is the method to use when you want a live dashboard you can act on. |
| **Bayesian** (prior → posterior, P(B>A)) | Only with a pre-set horizon **or** a proper decision rule. | **Bayesian does NOT protect against peeking** — a widely-believed myth. Stopping the moment P(B>A) crosses a fixed threshold (e.g. 0.95) while monitoring continuously *inflates false positives just like frequentist peeking.* The posterior stays interpretable; the *stop-on-success* rule is what breaks. |

**The load-bearing correction:** "we're Bayesian so we can peek and stop when it
hits 95%" is **wrong**. If you want to monitor continuously and stop whenever the
data justifies it, use a **sequential / always-valid** test — that's the only
family built for it. Otherwise, fix the sample size in advance and hold.

When you report a test result, state the method and the stopping rule you used,
the same way the measurement skill makes you state the attribution window. "B
won, +12% CTR (sequential test, n=4,200, stopped at the always-valid boundary)"
is a real claim. "B is winning at 96%!" mid-flight on a fixed-horizon test is not.

---

## 3. Landing-page optimization — A/B vs. contextual bandits

Two different jobs, two different tools:

- **A/B test** when you want to *learn* which variant is better and why
  (a clean, attributable result you can generalize). Split traffic evenly, run
  to validity (§2), pick the winner, ship it.
- **Contextual multi-armed bandit** (e.g. Unbounce Smart Traffic) when you want
  to *maximize conversions while learning* — it routes each visitor to the
  variant most likely to convert *people like them*, using context signals
  (geolocation, device, OS, timezone). It needs a short learning phase
  (~30 visitors to begin optimizing, ~50 before it routes reliably) and then
  beats a fixed even split because it stops sending cost-sensitive mobile users
  to the variant that only converts desktop.

Rule of thumb: **bandit when the goal is conversions now and variants can
coexist; A/B when the goal is a clean learning you'll reuse.** Don't run a
fixed-horizon A/B test and then "peek and route" — that's a bandit done wrong.

Draft variants for the client's page; a human approves what publishes.

---

## 4. Creative testing at scale (Meta)

Creative is the highest-leverage variable in paid — test it deliberately, not by
vibes. Use Meta's built-in experiment tools so the split is clean:

- **Check eligibility first:** `mcp__meta_ads__ads_experiment_check_eligibility`.
- **A/B test** (split test, e.g. creative A vs B at equal budget):
  `ads_experiment_abtest_create_test` → read with `ads_experiment_abtest_get_test`.
  Meta randomizes by user (no audience overlap), which a "duplicate the ad set"
  hack does not — use the real tool.
- **Conversion-lift / incrementality** (the gold standard — does the ad cause
  *incremental* conversions vs. a holdout?): `ads_experiment_lift_create_test` →
  `ads_experiment_lift_get_test`. This answers "would these people have converted
  anyway?", which ROAS alone never does.
- List/track running tests with `ads_experiment_list_tests`.

Test **one creative dimension at a time** (hook, format, angle) so a win is
attributable. Reading: tie the winner back to a real outcome metric from the
measurement skill (CPA/ENCAC), not just CTR — a high-CTR creative that doesn't
convert is a thumb-stopper, not a winner. Launching the test spends money →
**draft it, route for approval.**

---

## 5. Cross-channel budget — MMM vs. MTA

When the question is "how should budget split across Meta / Google / TikTok /
email", two model families answer it, and post-iOS14 they are not equal:

- **Multi-touch attribution (MTA)** stitches user-level touchpoints into a
  conversion path. It's granular but **reliant on online user-level signal** —
  exactly what ATT/iOS14 and cookie loss broke. Treat MTA splits as directional,
  degrading as privacy tightens.
- **Marketing mix modeling (MMM)** is a **privacy-friendly, aggregated** method:
  it regresses outcomes against channel-level spend over time, needs **no
  user-level data**, and is therefore resilient to signal loss. It's the more
  durable basis for "where should the next dollar go" at the channel level.
  Meta's open-source **Robyn** is the reference implementation; its **budget
  allocator** simulates reallocations across channels (solving each channel's
  nonlinear saturation curve) to find the mix that maximizes response.

You don't have to *run* an MMM to use the idea: reason in terms of **incremental
response and saturation** (a channel that's saturated returns less per added
dollar) rather than last-click ROAS, and recommend shifting budget toward
under-saturated, incrementally-proven channels. Any reallocation that moves money
is a **draft for human approval.**

---

## Don't

- **Don't peek-and-stop a fixed-horizon test.** Set the sample size first; don't
  read significance until you hit it. This applies to frequentist *and* naive
  Bayesian (0.95-threshold) tests alike.
- **Don't claim "Bayesian, so peeking is fine."** It isn't. Use a sequential /
  always-valid test if you need to monitor continuously.
- **Don't change the primary metric or stopping rule mid-test** to find a win.
- **Don't test more than one thing at once** in an A/B (unless it's a powered
  multivariate design) — you won't know what caused the result.
- **Don't call CTR a win.** Tie creative/LP winners to a real outcome
  (CPA/ENCAC), per the measurement skill.
- **Don't trust last-click/MTA splits as truth** under ATT — reason in MMM terms
  (incrementality + saturation) for budget moves.
- **Don't launch a test or move budget without approval.** You draft and
  interpret; a human publishes and funds.
