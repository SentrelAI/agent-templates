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

| Agent | Role | What it does |
|---|---|---|
| **[marketing](./marketing)** (Nova) | Growth / Marketing Lead | Plans + ships on-brand content, runs paid social via the Meta Ads MCP, sends email from its own outbox, and reports real performance — with Special-Ad-Category compliance, budget guardrails, and human approval on every spend/publish. |
| **[sdr](./sdr)** | Sales Development | Finds + enriches leads (Apollo), runs outbound sequences (Instantly), researches accounts, and books meetings — never sends without approval. |
| **[scheduler](./scheduler)** | Scheduling Assistant | Proposes meeting slots, runs the propose → follow-up → cold state machine on Google Calendar + Gmail, and self-schedules its own reminders natively. |
| **[bugfixer](./bugfixer)** | Engineering | Triages bugs from Sentry/Linear, reads + patches GitHub repos (branch → commit → PR), and comments back — read-first, change-on-approval. |

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
