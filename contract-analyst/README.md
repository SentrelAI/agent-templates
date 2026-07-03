# Lex — contract analyst bundle

A meticulous contract analyst. Reads every agreement before you sign or renew —
extracts the terms that matter, flags clauses that deviate from your playbook
(🔴/🟡/⚪, each with the quoted clause + plain-language consequence + a suggested
redline **for counsel**), and tracks renewal dates and obligations so no
contract auto-renews unseen. **Analyzes and flags; it never gives legal advice
and never approves, signs, or negotiates.**

## How it works

- **Contract review** via **Google Drive** + **Gmail/Outlook** — reads the full
  document set (MSA + order forms + amendments + incorporated ToS; PDFs and
  exported Google Docs), catches agreements arriving by email, and extracts the
  same key terms every time, quoted with section numbers.
- **Risk & redlines** — clause-by-clause against `contract-playbook.md`:
  deviations tiered red/yellow/note with quote → translation → consequence →
  playbook position → suggested redline. Hunts the classics (cap carved out of
  the indemnity, URL-ToS they can change, the auto-renewal trap, IP reach).
- **Obligations & renewals** via **Notion** — one log row per agreement, the
  full review on its page, reminders set **before** every notice window, and a
  weekly watch that frames each renewal as a decision with leverage — posted via
  **Slack** (native `slack.post`).

## Connect at /integrations
- **Google Drive** (required) — where contracts live.
- **Notion** (required) — the contract log.
- **Gmail / Outlook** — incoming agreements + counterparty threads.
- **Slack** (optional channel) — summaries + renewal warnings.

## Guardrails (baked in — absolute)
- **Never gives legal advice** — no opinions on legality, enforceability, or
  "safe to sign." Legal-judgment questions route to counsel with the analysis
  prepared.
- **Never approves or signs** anything; **never negotiates** with the
  counterparty; nothing goes to the other side without a human's yes.
- Redlines are **suggestions for your lawyer**, not demands sent outward.

Edit `knowledge/contract-playbook.md` to set your standard positions and hard
lines — the playbook *is* the risk model — without touching the persona.
