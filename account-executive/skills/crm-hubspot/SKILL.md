---
name: crm-hubspot
description: Use to read and keep the HubSpot CRM honest — searching deals/contacts/companies, reading a deal's full context, updating stage/amount/close-date/next-step, logging notes after calls, and creating follow-up tasks. Covers the HubSpot CRM v3 API through the mcp__apps__request proxy, the search filter syntax, pipeline stages, and associations.
---

# CRM (HubSpot)

The CRM is your source of truth. You read and update it through the **`request`**
tool:

```
request({ provider:"hubspot", method, path, query?, body? })
```

- **Base is `https://api.hubapi.com`.** Auth is injected — never handle a token.
- Result is `{ status, body }`. A 401/403 means HubSpot isn't connected — say so
  and ask the user to connect it at /integrations (don't guess).
- Objects live under `/crm/v3/objects/<type>` (`deals`, `contacts`,
  `companies`, `notes`, `tasks`).

## Read the open pipeline

Search is a POST with filter groups (AND within a group, OR across groups). Ask
for exactly the properties you need:

```
request({ provider:"hubspot", method:"POST", path:"/crm/v3/objects/deals/search",
  body:{
    filterGroups:[{ filters:[
      { propertyName:"dealstage", operator:"NEQ", value:"closedwon" },
      { propertyName:"dealstage", operator:"NEQ", value:"closedlost" }
    ]}],
    properties:["dealname","amount","dealstage","closedate","hs_lastmodifieddate","pipeline"],
    sorts:[{ propertyName:"hs_lastmodifieddate", direction:"DESCENDING" }],
    limit:100 } })
// → body.results[] each { id, properties:{...} }
```

Pagination: if `body.paging.next.after` is present, pass it as `after` on the
next search to page through — never assume one page is the whole pipeline.

## Understand the stages

Stage ids are pipeline-specific (not the labels). Fetch them once to map
labels ⇄ ids and know which stages are "closed":

```
request({ provider:"hubspot", method:"GET", path:"/crm/v3/pipelines/deals" })
// → body.results[].stages[] { id, label, metadata.isClosed, metadata.probability }
```

## Read a deal's full context

Get the deal, then its associated contacts/company so you know the buying
committee:

```
request({ provider:"hubspot", method:"GET", path:"/crm/v3/objects/deals/<dealId>",
  query:{ properties:"dealname,amount,dealstage,closedate,pipeline",
          associations:"contacts,companies" } })
// → body.properties + body.associations.contacts.results[].id
// then batch-read the contacts:
request({ provider:"hubspot", method:"POST", path:"/crm/v3/objects/contacts/batch/read",
  body:{ properties:["firstname","lastname","email","jobtitle"],
         inputs:[{ id:"<contactId>" }] } })
```

## Keep it current (update)

Patch only the fields that changed — stage, amount, close date, or a custom
next-step field:

```
request({ provider:"hubspot", method:"PATCH", path:"/crm/v3/objects/deals/<dealId>",
  body:{ properties:{ closedate:"2026-08-15", amount:"24000",
                      next_step:"Technical review booked 7/22" } } })
```

Moving a deal to a **won/commercial** stage or changing **amount** is
approval-gated — draft the update and get {{rep_name}}'s yes first.

## Log a note after a call

Notes need a timestamp and an association to the deal (and usually the contact):

```
request({ provider:"hubspot", method:"POST", path:"/crm/v3/objects/notes",
  body:{ properties:{ hs_note_body:"Discovery call — pain: 12% no-show rate; metric: hours lost/wk; champion: Dana (Ops Lead); next: security review.",
                      hs_timestamp:"<epoch ms or ISO>" },
         associations:[{ to:{ id:"<dealId>" },
           types:[{ associationCategory:"HUBSPOT_DEFINED", associationTypeId:214 }] }] } })
// associationTypeId 214 = note→deal; 202 = note→contact
```

## Create a follow-up task

Every commitment becomes a dated task so nothing slips:

```
request({ provider:"hubspot", method:"POST", path:"/crm/v3/objects/tasks",
  body:{ properties:{ hs_task_subject:"Send security doc to Dana",
                      hs_task_body:"Promised on 7/15 call",
                      hs_timestamp:"<due epoch ms>",
                      hs_task_status:"NOT_STARTED", hs_task_priority:"HIGH" },
         associations:[{ to:{ id:"<dealId>" },
           types:[{ associationCategory:"HUBSPOT_DEFINED", associationTypeId:216 }] }] } })
// associationTypeId 216 = task→deal
```

## Rules

1. **Search before you edit.** Read the deal's real state (stage, amount,
   contacts) before changing anything.
2. **Least update.** PATCH only changed properties; never blow away fields you
   didn't mean to touch.
3. **Stage ids, not labels.** Map via `/crm/v3/pipelines/deals`; a label isn't a
   valid value.
4. **Won / amount / close-to-commercial = approval.** Draft the change; don't
   apply it without {{rep_name}}.
5. **Every call → a note; every commitment → a task.** If it isn't in the CRM,
   it didn't happen and it will be forgotten.
6. **Page through search.** Follow `paging.next.after` — a partial pipeline read
   leads to wrong "no next step" conclusions.
