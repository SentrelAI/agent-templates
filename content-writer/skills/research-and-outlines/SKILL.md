---
name: research-and-outlines
description: Use to research a topic before writing and turn it into a solid outline — gathering facts + real examples + sources with web-search, reading the brand's own reference material in Google Drive, and structuring the piece for the reader. Covers the Google Drive API through the mcp__apps__request proxy plus the built-in web-search.
---

# Research & outlines

Good content is researched, not improvised. This is the step that makes a piece
accurate and worth reading — before a word of prose.

## Research the topic (web-search)
- Use **web-search** to gather facts, real examples, current data, and what's
  already been said well (so you can add, not repeat).
- **Capture the source for every claim** — a stat, a quote, a capability. If you
  can't find a credible source, the claim doesn't go in the piece (or gets
  flagged for a human).
- Distinguish fact from opinion, and note where sources disagree.

## Read the brand's own material (Google Drive)
Don't contradict or duplicate existing content. Find reference docs, prior
pieces, and brand assets:
```
request({ provider:"google_drive", method:"GET", path:"/files",
  query:{ q:"name contains '<topic>' and trashed = false", fields:"files(id,name,mimeType)", pageSize:20 } })
// → body.files[] { id, name, mimeType }
```
Read a Google Doc's text via the drafting skill (`GET /documents/<id>`), or a
plain file via `GET /files/<id>?alt=media`. Note what's already been said so the
new piece complements it.

## Build the outline (structure for the reader)
Before drafting, lay out:
- **The hook** — why a reader clicks + keeps reading.
- **The takeaway** — the one thing they leave with.
- **The spine** — the logical sections (H2s) that get them there.
- **Evidence per section** — the fact/example/source that backs each point.
- **The CTA** — what you want them to do next (if any).
A piece with a real outline writes itself; one without wanders.

## Rules
1. **Source every claim.** No source → cut it or flag it. Never invent a stat or
   quote.
2. **Read before you write** — check the brand's existing content so you add,
   not repeat or contradict.
3. **Outline before prose** — hook, takeaway, spine, evidence. Structure is the
   hard part; nail it first.
4. **Keep the source list** — carry it into the draft so a human can verify every
   claim.

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
