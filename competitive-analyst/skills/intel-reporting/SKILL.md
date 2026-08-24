---
name: intel-reporting
description: Use when delivering intel — the daily chat digest, the weekly email report, red alerts, and battlecard upkeep. Covers formats, channel routing (chat vs email), the silence rule, and how findings file into Notion when connected.
---

# Intel reporting

Analysis nobody reads is analysis that didn't happen. These formats are
optimized for a founder skimming between meetings.

## Channel routing

| What | Where | When |
|---|---|---|
| 🔴 Red alert | Email {{user_name}} immediately + chat | Within the sweep that confirmed it |
| 🟡 Daily deltas | Chat digest (the sweep's reply) | Only when something's new |
| Weekly report | Email + chat summary line | Every Friday, no exceptions |
| Deep-dive answers | Chat; email too if asked or red | On delivery |

Email goes out from my own address — it's the designed reporting loop
(`send_report_email: auto`). Nothing else I produce leaves the org.

## Daily digest format (chat — under a minute to read)

```
🟡 CompetitorOne shipped bulk export (changelog, yday) — closes a gap our
   sales deck currently exploits → link
🟡 CompetitorTwo pricing: Pro $49→$59, annual-only push (pricing page) → link
⚪ 2 minor items filed to profiles
```

Delta lines only. No pleasantries, no "in summary." Severity emoji leads,
"so what" is welded to the fact, link ends the line.

## Weekly report format (email)

Subject: `Intel week of <date>: <the one-line headline of the week>`

1. **The week in one paragraph** — what actually mattered.
2. **Competitor moves** — per competitor with deltas only: fact → source →
   what it means for {{company_name}}.
3. **Market & funding** — category-level shifts, rounds, entrants.
4. **Prospects surfaced** — the week's rows from prospect-mining, ready for
   handoff.
5. **Recommended actions** — 2-3, ranked, each tied to a finding above
   ("update the comparison page: their new SSO kills our second-strongest
   objection").
6. **Watching next week** — open threads.

Quiet week: sections collapse but the report still ships — "nothing moved"
plus the best available read on *why* (seasonal? heads-down before a
launch?) is real intelligence.

## Red alert format (email, rare)

Subject: `🔴 <Competitor>: <the fact>`. Body: the fact with sources, the
confidence level, what it means for us, and the one question worth
discussing today. Five sentences beat five paragraphs when it's urgent.

## Battlecards

`workspace/battlecards/<slug>.md` per competitor: **How we win** (our real
edges, in the buyer's language) · **Where they're strong** (honest — sales
gets ambushed by omissions) · **Landmines** (claims of theirs to preempt) ·
**Objection counters** (their pitch against us → our response). Refresh
whenever that competitor moves; note the refresh in the weekly report.

## Filing (when Notion is connected)

Weekly reports and battlecards also file into the intel base — find the
database via `POST /v1/search`, confirm with the owner once, remember the id
(a durable fact). Chat/email remain the delivery path; Notion is the
archive, never the only copy.
