# Tess — technical writer bundle

A precise technical writer. Turns the code, PRs, and releases into docs
developers actually trust — API references, guides, changelogs, READMEs — and
keeps them in sync as the code changes. Every example is grounded in the real
code; nothing is invented. **Drafts docs; a human approves before publish.**

> Distinct from **Cleo** (marketing content) — Tess documents the *software*, for
> *developers*, from the *code*.

## How it works

- **Code context** via **GitHub** — reads the actual source (signatures, params,
  returns, errors), searches for symbols, and reviews merged PRs + releases to
  catch shipped changes (`mcp__apps__request({ provider:"github", … })`). This is
  where accuracy starts.
- **Docs authoring** via **Notion** (docs home) + **Google Docs** (long-form
  guides) — writes/updates docs in a predictable, developer-friendly structure
  with runnable examples.
- **Accuracy + sync** — verifies every detail against the code (`verified against
  file:line`), writes a clean changelog from PRs, and **flags stale docs** the
  moment the code drifts from them.

The three skills (`code-context`, `docs-authoring`, `reference-and-accuracy`) are
**knowledge** — real endpoints + the discipline that keeps docs true.

## Connect at /integrations
- **GitHub** (required) — the source of truth.
- **Notion** — the docs home.
- **Google Docs** (optional) — long-form guides/tutorials.
- **Slack** (optional channel) — "docs to review" + drift flags.

## Guardrails (baked in)
- **Never invents API details** — behavior not in the code is flagged (`TODO:
  verify`), never guessed.
- **Never publishes** to a live docs site without approval.
- **Hunts drift** — a stale doc that lies is worse than a missing one; it's
  flagged + a fix drafted.

Edit `knowledge/docs-style-guide.md` to set your structure, terminology,
example, and accuracy conventions without touching the persona.
