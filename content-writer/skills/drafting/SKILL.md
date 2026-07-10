---
name: drafting
description: Use to write and structure a draft in Google Docs — creating the doc, inserting the full piece, and applying headings/formatting. Covers the Google Docs API through the mcp__apps__request proxy and the all-important character-index gotcha for batchUpdate. Draft-only; never publishes.
---

# Drafting (Google Docs)

You write the piece in a Google Doc through the **`request`** tool:

```
request({ provider:"google_docs", method, path, query?, body? })
```

- **The base includes the version** (`https://docs.googleapis.com/v1`), so paths
  are relative: `/documents`, `/documents/<id>:batchUpdate`. Auth injected.
- All edits are **batchUpdate requests** that operate on **character indices** —
  index `1` is the start of the body.

## Create the doc
```
request({ provider:"google_docs", method:"POST", path:"/documents",
  body:{ title:"<piece title> — draft" } })
// → body.documentId
```
(Optionally move it into the content folder with the google_drive skill.)

## Insert the full piece in ONE call (the reliable pattern)
Write the whole draft as a single string (headings on their own lines, `\n`
between paragraphs) and insert it once at index 1 — this avoids fragile
index-tracking across many inserts:
```
request({ provider:"google_docs", method:"POST", path:"/documents/<id>:batchUpdate",
  body:{ requests:[ { insertText:{ location:{ index:1 }, text:"<the whole draft text>" } } ] } })
```

## Apply headings + formatting (a second batchUpdate)
Because you know the exact text you inserted, you can compute each heading's
**start/end character index** (offset from index 1) and style it:
```
request({ provider:"google_docs", method:"POST", path:"/documents/<id>:batchUpdate",
  body:{ requests:[
    { updateParagraphStyle:{ range:{ startIndex:1, endIndex:<title-end> },
        paragraphStyle:{ namedStyleType:"TITLE" }, fields:"namedStyleType" } },
    { updateParagraphStyle:{ range:{ startIndex:<h2-start>, endIndex:<h2-end> },
        paragraphStyle:{ namedStyleType:"HEADING_2" }, fields:"namedStyleType" } },
    { updateTextStyle:{ range:{ startIndex:<a>, endIndex:<b> },
        textStyle:{ bold:true }, fields:"bold" } }
  ] } })
```
**Index gotcha:** indices are UTF-16 character offsets and *shift as you insert*.
That's why you insert all text first, then style by ranges computed against the
final text — don't interleave inserts and styles.

## Read / revise an existing draft
```
request({ provider:"google_docs", method:"GET", path:"/documents/<id>" })
// → body.body.content[] structural elements (paragraphs, their text + start/endIndex)
```
To revise, read the content, compute the range to change, then
`deleteContentRange` + `insertText`, or `replaceAllText` for simple swaps.

## Rules
1. **Insert once, then style** — avoids index drift; never interleave.
2. **Draft only** — the doc is a draft; publishing is a human's call (blocked).
3. **Title + meta** — every piece gets a working title (TITLE style) and a meta
   description (note it at the top or in the calendar entry).
4. **Keep the source list** — drop the sources for every claim at the end of the
   draft (research-and-outlines skill) so a human can verify.

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
