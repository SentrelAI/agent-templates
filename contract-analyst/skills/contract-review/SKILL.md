---
name: contract-review
description: Use to get a contract in front of your eyes and extract what matters — finding + reading agreements in Google Drive (PDF/Docs, incl. exports), catching contracts arriving by email, and pulling the key terms (parties, term/renewal, liability, IP, termination, payment) into a structured extraction. Covers the Drive + mail APIs through the mcp__apps__request proxy.
---

# Contract review (Drive + mail)

Reading the actual document — all of it — is the job. You reach contracts
through the **`request`** tool.

## Find + read the agreement (Google Drive)
```
request({ provider:"google_drive", method:"GET", path:"/files",
  query:{ q:"fullText contains '<counterparty>' and trashed = false",
          fields:"files(id,name,mimeType,modifiedTime)", orderBy:"modifiedTime desc", pageSize:20 } })
```
- **Base `https://www.googleapis.com/drive/v3`** (versioned → relative paths).
- **PDFs / Word files:** `GET /files/<id>?alt=media` (binary → extract the text).
- **Google Docs:** must be **exported** — `GET /files/<id>/export?mimeType=text/plain`.
- Get *every* related file: the MSA, the order form, the DPA, prior amendments —
  terms incorporate each other, and the order form often overrides the MSA.

## Catch contracts arriving by email
- **Gmail:** `GET /gmail/v1/users/me/messages?q=filename:pdf (contract OR agreement OR MSA OR NDA) newer_than:30d`,
  attachments via `GET /gmail/v1/users/me/messages/<id>/attachments/<attId>`.
- **Outlook:** `GET /me/messages?$search="agreement"` → `GET /me/messages/<id>/attachments`.
Read the thread too — the counterparty's cover email ("just our standard terms")
is context, not truth. The document is the truth.

## Extract the key terms (the same fields, every time)
Pull into the structured log (obligations-and-renewals skill):
- **Parties + entity names** (exactly as written)
- **Term** — start, length, **renewal mechanics** (auto? notice window? where?)
- **Payment** — amounts, escalators, late fees, net terms
- **Liability cap** — amount/formula, carve-outs, and whether it's mutual
- **Indemnification** — who indemnifies whom, for what, capped or not
- **IP** — who owns what's created; any license grants or assignments
- **Termination** — for convenience? for cause? what survives?
- **Confidentiality / data** — obligations, DPA, breach duties
- **Exclusivity / non-compete / non-solicit** — any lock-ins
- **Governing law + dispute** — jurisdiction, arbitration
- **SLAs** — commitments + credits
Quote each key clause with its section number — the quote is the evidence the
risk pass and counsel work from.

## Rules
1. **Read everything** — exhibits, order forms, incorporated URLs. The order
   form overrides the MSA more often than not.
2. **The document is the truth** — never rely on the cover email's summary.
3. **Quote with section numbers** — extraction without citations can't be
   verified.
4. **Same fields every contract** — consistency is what makes the log usable.
5. **Diff amendments against the current agreement** — review what changed, not
   what the email says changed.
