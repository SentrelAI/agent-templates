# Sentrel Agent Templates

Ready-to-deploy **AI employees** for [Sentrel](https://sentrel.ai) — each one a
self-contained agent bundle (`agent-bundle/v1`): an `agent.yaml` manifest plus
the persona, skills, and knowledge it needs to do real work.

Deploy any of them in one click:

```
https://sentrel.ai/deploy-agent?source=https://github.com/SentrelAI/agent-templates/tree/main/<agent>
```

…or with the CLI:

```sh
npx @sentrelai/agentmanifest deploy ./<agent>
```

## The agents

| Agent | Name | Category | What it does |
|---|---|---|---|
| [general-assistant](./general-assistant) | Gia | starter | Light-touch everything for one person — inbox, calendar, tasks. Start here. |
| [executive-assistant](./executive-assistant) | Vera | personal | Chief-of-staff depth for one principal: briefs, VIP triage, weekly planning. |
| [scheduler](./scheduler) | Rio | ops | Only books meetings — the propose → follow-up → cold state machine, end to end. |
| [meeting-manager](./meeting-manager) | Mira | ops | Team-wide agendas, recaps from your notes/transcripts, action-item chase. |
| [marketing](./marketing) | Nova | marketing | Creative gen, native social publishing, Meta ads within budget, reporting. |
| [content-writer](./content-writer) | Cleo | marketing | Long-form + social copy in your voice, editorial calendar. Drafts; Nova distributes. |
| [sdr](./sdr) | Sarah | sales | Lead lists (Apollo), warm sequences (Instantly), qualification, AE handoff. |
| [account-executive](./account-executive) | Ace | sales | Pipeline hygiene, follow-ups, pre-meeting briefs, CRM as source of truth. |
| [support](./support) | Remy | support | Inbox triage → grounded drafts → escalation; SLA tracking. |
| [bugfixer](./bugfixer) | Patch | engineering | Sentry/Linear triage → root cause → branch/commit/PR with verification. |
| [code-reviewer](./code-reviewer) | Cody | engineering | PR review with a severity taxonomy; never merges. |
| [qa-engineer](./qa-engineer) | Quinn | engineering | Repro-first bug triage, release readiness, fix verification. |
| [data-analyst](./data-analyst) | Dana | engineering | Sheets/Airtable analysis with scripted math, weekly reports. |
| [technical-writer](./technical-writer) | Tess | engineering | Docs that match the code — changelogs, guides, update-in-place. |
| [project-manager](./project-manager) | Milo | ops | Sprint truth from Linear, standups, blocker chase; nudges, never orders. |
| [research-analyst](./research-analyst) | Ren | ops | Triangulated external research with confidence levels and citations. |
| [bookkeeper](./bookkeeper) | Bailey | ops | Stripe reconciliation, invoicing, collections drafts — never moves money. |
| [financial-analyst](./financial-analyst) | Finn | ops | Model-grounded FP&A: runway, variance, scenarios; every number sourced. |
| [contract-analyst](./contract-analyst) | Lex | ops | Contract review with exact-clause quotes, obligations + renewal watch. |
| [recruiter](./recruiter) | Reese | people | Candidate pipeline, fair screening, interview scheduling, zero ghosting. |

### Which assistant?

Four of these overlap on "helps me with my day" — pick by shape of need:
**general-assistant** = one person, light touch, everything (start here) ·
**executive-assistant** = one principal, chief-of-staff depth ·
**scheduler** = booking meetings only, autonomously ·
**meeting-manager** = the team's meetings (agendas/recaps/actions), doesn't book or triage.

## How a bundle works

A bundle is a directory:

```
<agent>/
  agent.yaml            # the manifest — name, model, inputs, skills, schedules
  identity.md           # who the agent is
  personality.md        # how it behaves
  instructions.md       # how it works, step by step
  knowledge/            # reference docs it can read
  skills/<skill>/SKILL.md   # how to use each connected app's API
```

Connected apps are reached through Sentrel's generic proxy
(`mcp__apps__request`) or, for the rich ad surface, the dedicated Meta Ads MCP
(`mcp__meta_ads__*`). The agent has built-in native capabilities too — its own
email outbox, scheduling/reminders, tasks, knowledge base, and web search — so
those never need an external connection.

## Spec

These bundles follow the open `agent-bundle/v1` schema. See the
[agentmanifest](https://github.com/SentrelAI/agent-manifest) tooling + schema.

---

© Sentrel — build your AI workforce.
