---
name: document-research
description: Use to research from internal material — finding + reading documents in Google Drive (reports, decks, PDFs) and the Notion knowledge base, so research builds on what the company already knows instead of starting cold. Covers the Google Drive + Notion APIs through the mcp__apps__request proxy.
---

# Document research (Google Drive + Notion)

Before (and alongside) the web, use what the company already has. You read
internal material through the **`request`** tool.

## Google Drive — find + read source docs
```
request({ provider:"google_drive", method:"GET", path:"/files",
  query:{ q:"fullText contains '<topic>' and trashed = false",
          fields:"files(id,name,mimeType,modifiedTime)", pageSize:25,
          orderBy:"modifiedTime desc" } })
// → body.files[] { id, name, mimeType }
```
- **Base is `https://www.googleapis.com/drive/v3`** (versioned → relative paths).
- Read a plain file: `GET /files/<id>?alt=media`.
- **Google-native files** (Docs/Sheets/Slides) must be **exported**, not
  downloaded: `GET /files/<id>/export?mimeType=text/plain` (Docs) or
  `text/csv` (Sheets). `fullText contains` searches inside file contents.
- Prefer the most recent version; note the `modifiedTime` (stale internal docs
  are as risky as stale web sources).

## Notion — read the knowledge base
```
request({ provider:"notion", method:"POST", path:"/v1/search",
  body:{ query:"<topic>", filter:{ property:"object", value:"page" }, page_size:10 } })
request({ provider:"notion", method:"GET", path:"/v1/blocks/<pageId>/children", query:{ page_size:100 } })
```
- **Base `https://api.notion.com`**; auth + `Notion-Version` injected. Walk the
  blocks; text in `block.<type>.rich_text[].plain_text`.

## Use it right
- Internal docs are **sources too** — cite them (title + date + author) with a
  confidence level, same as web sources.
- Check recency and whether a newer doc supersedes an older one.
- Internal ≠ automatically true — a stale strategy deck or an unverified internal
  claim gets the same scrutiny as anything else.

## Rules
1. **Start with what's known** — read internal material so you build on it, not
   repeat it.
2. **Export Google-native files** (`/export`), don't try to download them raw.
3. **Cite internal sources too**, with date + confidence.
4. **Recency matters** — a 2-year-old internal doc can be as wrong as an old web
   page; note the date.
