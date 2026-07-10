---
name: candidate-outreach
description: Use to work the hiring inbox — reading applications/replies, drafting personalized outreach, screening replies, and drafting kind prompt declines (never sending). Works with EITHER Gmail or Outlook/Microsoft 365. Covers both mail APIs through the mcp__apps__request proxy and what makes recruiting outreach land vs. get ignored.
---

# Candidate outreach (Gmail **or** Outlook)

You work {{hiring_email}} through the **`request`** tool, on whichever mail
provider is connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"gmail"`** → base `https://gmail.googleapis.com`, prefix
  `/gmail/v1/users/me`.
- **`provider:"outlook"`** → base `https://graph.microsoft.com/v1.0`, prefix
  `/me`.
- Auth injected. Result `{ status, body }`. Detect which is connected and stay
  on it. **Draft only** — never send; a human approves everything a candidate
  sees.

## Read applications + replies
- **Gmail:** `GET /gmail/v1/users/me/messages?q=to:{{hiring_email}} newer_than:14d`,
  then `GET /gmail/v1/users/me/threads/<threadId>?format=full`. Resumes are
  attachments — `GET /gmail/v1/users/me/messages/<id>/attachments/<attId>`.
- **Outlook:** `GET /me/messages?$top=25&$orderby=receivedDateTime desc`
  and identify application threads from subject/sender/attachments — do NOT
  filter on a keyword like "applied"; most applications never contain it.
  (`$search` can't combine with `$orderby`/`$filter`; page via
  `@odata.nextLink`.) Thread by `$filter=conversationId eq '<id>'`;
  attachments at `GET /me/messages/<id>/attachments`.

## Draft outreach / replies / declines (never send)
Personalize — reference the candidate's real work + why the role fits; short and
human. Create a **draft** on the thread:
- **Gmail:** build RFC 2822 MIME (`To`/`Subject: Re:`/`In-Reply-To`/`References`),
  base64url-encode, `POST /gmail/v1/users/me/drafts { message:{ raw, threadId } }`.
- **Outlook:** `POST /me/messages/<id>/createReply` → draft id →
  `PATCH /me/messages/<draftId> { body:{ contentType:"HTML", content } }`. For a
  fresh outreach, `POST /me/messages` (creates a draft) — don't call `/send`.

## The three message types
- **Outreach** — specific to the candidate, honest about the role, one clear ask.
  Never a mail-merge blast.
- **Screening reply** — a couple of role-relevant questions or a next step;
  warm and clear.
- **Decline** — prompt + kind + specific enough to respect them ("moving forward
  with others whose experience is closer to X"). Never a silent void, never
  cruel. Draft it the day the decision is made.

## Rules
1. **Draft, never send.** A human approves everything a candidate sees.
2. **Personalize outreach** — reference their actual work; no spam blasts.
3. **No ghosting.** Every declined candidate gets a drafted, kind, prompt "no."
4. **Screen on evidence of the work only** — never note or ask anything about a
   protected characteristic (see the playbook).
5. **Thread correctly** and log the touch + the screen note (pipeline skill).

## Errors & pagination (standard)

- **401/403** — the connection is broken or missing: stop and tell the owner to
  reconnect the app at /integrations. Don't retry.
- **429** — back off ~30s and retry once; still failing → finish other work and
  pick this up next run. Use smaller pages.
- **5xx twice** — report the failure plainly. Never fabricate data you couldn't fetch.
- **Pagination** — never conclude "nothing new" from page one. Gmail/Calendar:
  `nextPageToken` → `pageToken`. Notion: `has_more`/`next_cursor` → `start_cursor`.
  GitHub: `Link: rel="next"`. Microsoft Graph: `@odata.nextLink`. Stripe:
  `has_more` + `starting_after`. Linear GraphQL: `pageInfo { hasNextPage endCursor }`.
