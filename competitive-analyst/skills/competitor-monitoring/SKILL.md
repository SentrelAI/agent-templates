---
name: competitor-monitoring
description: Use for the recurring sweep across tracked competitors and market terms — finding NEW developments (funding, launches, pricing, content, people, hiring), verifying them, updating profiles and the intel ledger, and assigning severity. This is the delta engine; reporting formats live in intel-reporting.
---

# Competitor monitoring

The sweep answers one question per competitor: **what changed since I last
looked?** Everything else — profiles, ledger, severity — exists to make that
answer fast and non-repetitive.

## Per-competitor sweep recipe (web search, in this order)

1. **News & funding:** search `"<Competitor>" funding OR raised OR Series OR
   acquisition` and `"<Competitor>" announcement`, restricted mentally to the
   last 7 days. Cross-check anything that looks new against the ledger.
2. **Product:** check their changelog/release-notes/docs pages directly
   (keep the URLs in the profile once discovered — `site:<domain> changelog
   OR "release notes" OR "what's new"` finds them the first time). Diff
   against the profile's product timeline.
3. **Pricing:** hit the pricing page. Compare plan names/prices/limits
   against the profile's pricing snapshot; any change is at least 🟡.
4. **Content & positioning:** `site:<domain>/blog` for the week's posts;
   note only what reveals strategy (new segment, new claim, new comparison
   page — especially comparison pages naming our company).
5. **People & hiring:** search `"<Competitor>" hires OR joins OR appointed`
   and skim their careers page. A cluster of hires in one function is a
   roadmap leak (three ML hires = they're building the thing).
6. **Market terms:** one pass over the market watch terms (in my identity) for category-level
   moves — new entrants, regulation, platform shifts.

## Delta discipline

- Before reporting anything, check `workspace/intel-ledger.md` — if it's
  there, it's old news. After the sweep, append what you reported and stamp
  the sweep date per competitor.
- Confirmed developments also update the competitor's profile file — the
  profile is the accumulated truth; the ledger is just the "don't repeat
  yourself" cursor.
- First sweep ever for a competitor: build the initial profile (that's
  backfill, not news — report only a one-line "profile established" with
  anything genuinely live right now).

## Verification before severity

Apply the evidence bar (instructions): two sources to call it confirmed, the
funding chain from deep-dive-research for money news, direct-source check
(their own site) for product/pricing claims. THEN assign 🔴/🟡/⚪ per the
severity taxonomy in knowledge/intel-policy.md. When genuinely uncertain
between two severities, pick the lower and say why — false reds are how
analysts get muted.

## New-competitor radar

While sweeping, note names that keep appearing in the same breath as tracked
competitors ("alternatives to", comparison posts, review-site categories).
Two independent appearances → propose adding them (instructions: propose,
don't self-expand).
