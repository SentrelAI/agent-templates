# Sarah — Sales Development Rep

Outbound SDR: builds targeted lead lists with Apollo, personalizes with
LinkedIn + news research, runs warm sequences through Instantly, and books
qualified meetings for your AE. Everything she *is* lives in these files;
secrets stay out (tokens are injected per connected app at runtime).

```
sdr/
├── agent.yaml                      # manifest — identity, goal, wiring
├── personality.md                  # how Sarah sounds and behaves
├── identity.md                     # who she is inside the company
├── instructions.md                 # the outbound pipeline she runs
├── knowledge/
│   └── pitch-deck.md               # source of truth for all product claims
└── skills/
    ├── apollo-enrichment/          # build + enrich lead lists
    ├── instantly-outreach/         # warm sending, campaigns, replies
    ├── linkedin-prospecting/       # per-prospect research hooks
    └── news-research/              # web-search news hooks
```

Every connected app (Apollo, Instantly, LinkedIn) is reached through one
tool — `mcp__apps__request({ provider, method, path, query?, body? })`.
The runtime injects each connected account's token, so no API keys live
in this bundle. Connect each app once at `/integrations`.

## What it demonstrates

| Requirement | Where |
| --- | --- |
| Own email address | `channels[0]` (`type: email`, `sarah@{{company_domain}}`) |
| Apollo for enrichment | `integrations` service `apollo` + `skills/apollo-enrichment` (via `mcp__apps__request`) |
| Instantly with warm sending | `integrations` service `instantly` + `skills/instantly-outreach` (warmup-score gate ≥90, paste-token connect) |
| Pitch deck | `knowledge/pitch-deck.md`, enforced by personality ("if it isn't in the deck, I don't say it") |
| Goal: book meetings | `goal.mission` + KPIs (5 meetings/week, 8% positive replies) + `definition_of_done` |
| LinkedIn | `integrations` service `linkedin` + `skills/linkedin-prospecting` (gated API — honest fallback) |
| News | `skills/news-research` via the built-in web search (no app needed) |

## Safety posture

- `send_email: ask` / `cold_email_bulk: ask` — no outbound without approval
- `reply_email: auto` / `book_meeting: auto` — momentum where it's safe
- `delete_data: block`
- No credentials live in the bundle: connected apps inject their tokens
  at call time through `mcp__apps__request`.

## Before deploying

Deploy Sarah from the Templates page. Then: replace the bracketed sections of
`knowledge/pitch-deck.md` with your actual deck, and connect Apollo,
Instantly, and LinkedIn at `/integrations` (Instantly is a paste-token
connect — there's no Nango template for it).
