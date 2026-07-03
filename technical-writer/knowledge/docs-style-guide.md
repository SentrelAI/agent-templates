# Docs style guide

How {{agent_name}} writes docs for {{company_name}}. **Edit this** to set your
conventions without touching the persona. Audience: {{audience}}.

## Structure (every reference)
1. What it does — one line.
2. How to use it — the call/usage.
3. Parameters — name · type · required/optional · default · description.
4. A **working example** — realistic values, correct field names.
5. Returns / errors — the shape + failure modes.
6. Gotchas — nulls, rate limits, edge cases.

## Voice
- Precise, concrete, brief. Show with examples; cut prose that doesn't help
  integration.
- Second person ("you"), active voice, present tense.
- Avoid "simply / just / obviously" — if it were obvious they wouldn't be reading.

## Terminology
- **One name per concept**, matching the code + product. (Keep a glossary here:
  the canonical term for each concept, and the synonyms to avoid.)
- Match the code's casing exactly in identifiers.

## Code examples
- Runnable, realistic values (not `foo`/`bar` where a real shape teaches more).
- Show the response shape. Include auth/setup a first-timer needs.
- Language(s) + formatting conventions for snippets: (edit — e.g. `bash` for curl,
  plus one SDK language).

## Accuracy (hard rules)
- **Never invent** a param/return/endpoint/default/behavior. Code is truth.
- Unverifiable → `TODO: verify` + flag; never confident prose over a guess.
- Distinguish current vs. deprecated/planned; note version where it matters.

## Changelog
- Grouped Added / Changed / Fixed / Deprecated / **Breaking**.
- In user-facing terms; breaking changes loud + with a migration path.

## Approval
- **Nothing publishes** to a live docs site without a human — Tess drafts to
  review.

## Add your own
Your docs IA (where API refs vs. guides vs. changelog live), the SDK languages to
show, the deprecation policy, versioning scheme, and links to exemplar docs pages.
