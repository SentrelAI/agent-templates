---
name: obligations-and-renewals
description: Use to run the contract log in Notion — one row per agreement with terms, dates, flags, and status — set reminders BEFORE every notice window, and surface renewals/obligations early enough that the company decides (renew, renegotiate, terminate) instead of defaulting. Covers the Notion API through the mcp__apps__request proxy plus Slack (native slack.post) for warnings.
---

# Obligations & renewals (Notion)

The log is what stands between the company and the auto-renewal it forgot. One
row per agreement, dates tracked, reminders set early. Through the **`request`**
tool:

```
request({ provider:"notion", method, path, query?, body? })
```

- **Base `https://api.notion.com`.** Auth + `Notion-Version` injected. The log is
  a Notion **database**; the full review (extraction + flags) lives on each
  contract's page.

## Log a contract (one row, same fields every time)
```
request({ provider:"notion", method:"POST", path:"/v1/pages",
  body:{ parent:{ database_id:"<contractsDbId>" },
    properties:{
      Name:{ title:[{ text:{ content:"Acme Corp — MSA" } }] },
      Counterparty:{ rich_text:[{ text:{ content:"Acme Corp" } }] },
      Status:{ select:{ name:"Reviewed — 2 flags" } },
      "Term end":{ date:{ start:"2027-03-01" } },
      "Notice deadline":{ date:{ start:"2026-12-01" } },
      "Auto-renews":{ checkbox:true },
      Value:{ number:24000 },
      Risk:{ select:{ name:"🟡 Yellow" } } } } })
```
Append the full review (extraction, tiered flags, suggested redlines) as blocks
on the page — the log row is the radar; the page is the file.

## Track obligations (ours AND theirs)
Deliverables, SLAs, payment milestones, data-deletion duties on termination —
each an entry with an owner + due date. Breaching our own contract because
nobody tracked §7's quarterly report is the avoidable failure.

## The weekly watch (surface EARLY)
Query for anything moving in the next 60 days:
```
request({ provider:"notion", method:"POST", path:"/v1/databases/<contractsDbId>/query",
  body:{ filter:{ or:[
    { property:"Notice deadline", date:{ next_month:{} } },
    { property:"Term end", date:{ next_month:{} } } ] },
    sorts:[{ property:"Notice deadline", direction:"ascending" }] } })
```
Also set a built-in task/reminder at logging time for **before** each notice
window (e.g. 30 days ahead) — the sweep is the net; the reminder is the alarm.

Post the warning where it gets acted on (Slack is a native channel — `slack.post`):
```
slack.post({ text:
  "📋 *Contract watch*\n" +
  "🔴 *Acme MSA* auto-renews 3/1 — notice window closes *12/1* (in 5 weeks). Renew, renegotiate, or terminate? Renegotiating the liability cap is on the table now, locked after.\n" +
  "🟡 Beta DPA: our quarterly security report (§7) due 7/31 — owner needed." })
```
Frame every renewal as the **decision** it is, with the leverage stated —
not just a date.

## Rules
1. **Every contract gets a row** — unlogged = untracked = auto-renewed.
2. **Reminders set BEFORE the notice window**, at logging time — never after.
3. **Track our obligations too** — don't let us be the breaching party.
4. **Renewals are decisions with leverage** — surface them as choices, early,
   with what's negotiable now vs. locked later.
5. **The log row is the radar; the contract page is the file** — both stay
   current.
