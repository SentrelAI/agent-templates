# Scout — Competitive Intelligence Analyst

Watches your market so you don't have to. Scout keeps a living profile on every
competitor, sweeps daily for what changed — funding, launches, pricing, content,
people — verifies before reporting, and mines their unhappy customers as warm
prospects for your pipeline. Tight deltas in chat every morning; a full
intelligence report by email every Friday; an immediate email when something
market-shifting is confirmed.

```
competitive-analyst/
├── agent.yaml                       # manifest — inputs, schedules, permissions
├── identity.md / personality.md / instructions.md
├── knowledge/
│   └── intel-policy.md              # source tiers, severity taxonomy, the ethics line
└── skills/
    ├── competitor-monitoring/       # the daily delta engine
    ├── deep-dive-research/          # triangulation + verification chains
    ├── prospect-mining/             # switching signals → prospect rows
    └── intel-reporting/             # digest/report/alert formats + battlecards
```

## Zero-connection deploy

Scout's core loop runs entirely on built-in web search — deploy with no
integrations and it works. Connect **Notion** to archive reports and
battlecards, **Google Sheets** for a shared prospect tracker, **LinkedIn**
for native company-post monitoring.

## Setup inputs

- **What you sell, and to whom** — Scout's relevance filter; everything is
  judged against it.
- **Competitors** — the initial tracked list. Scout proposes additions when it
  finds players you missed (and never expands the list without your yes).
- **Market watch terms** — optional category-level monitoring.

## Guardrails (baked in)

- Every claim ships with a source link; confirmed/reported/rumor never blur.
- **Scout never contacts anyone** — prospects are surfaced with signals and
  angles; outreach stays with you (or your SDR agent, via the weekly handoff
  section).
- Public information only — the ethics line in `knowledge/intel-policy.md` is
  absolute and editable.
- Nothing Scout writes leaves the org: reports go to you, not the world.

## Works well with

- **Sarah (sdr)** — Scout's weekly prospect rows are formatted for direct
  handoff into her outreach pipeline.
- **Nova (marketing)** — battlecards and positioning shifts feed her
  comparison content.
- **Ren (research-analyst)** — Ren answers arbitrary hard questions; Scout is
  the standing watch on one market. They overlap on method, not on job.
