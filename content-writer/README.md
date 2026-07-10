# Cleo — content writer bundle

A sharp content writer. Researches the topic properly, drafts clear, on-brand
pieces in your voice (blog posts, guides, emails, landing copy), and keeps the
editorial calendar moving. **Writes and drafts; a human approves before anything
publishes or sends.**

## How it works

- **Drafting** via the apps tool — Cleo writes + structures the piece in
  **Google Docs** (`mcp__apps__request({ provider:"google_docs", … })`),
  applying headings + a working title/meta. It's a *draft*; publishing is a
  human's call (blocked — no CMS wired either).
- **Research** via **web-search** + **Google Drive** — gathers facts + examples
  with a **source for every claim**, and reads the brand's own reference
  material so it adds, not repeats. Never invents a stat or quote.
- **Editorial calendar** via **Notion** — sharpens the brief, moves each piece
  through stages (idea → brief → drafting → **ready for review**), and records
  title/meta/angle in one place.

The three skills (`drafting`, `research-and-outlines`, `editorial-calendar`) are
**knowledge** — real endpoints + gotchas (the Google Docs character-index trap,
Drive search, Notion calendar).

## Connect at /integrations
- **Google Docs** (required) — where drafts are written.
- **Notion** — the editorial calendar + briefs.
- **Google Drive** — reference material + organizing drafts.
- **Slack** (optional channel) — "draft ready for review" + calendar updates.

## Guardrails (baked in)
- **Never publishes or sends to an audience** without a human's approval.
- **Never fabricates** — every claim has a source, or it's cut/flagged.
- Delivers to **"ready for review,"** with sources + open questions attached.

Edit `knowledge/brand-voice-and-style.md` to set the voice, style rules, banned
words, SEO approach, and accuracy standards without touching the persona.

**Email lane:** Cleo drafts email copy; sending campaigns is the marketing agent's (or your ESP's) job — deploy Nova for distribution.
