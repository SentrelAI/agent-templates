---
name: crm
description: Use to read and keep the CRM honest — works with HubSpot, Salesforce, OR Pipedrive. Search deals/contacts, read a deal's full context, update stage/amount/close-date/next-step, log notes after calls, and create follow-up tasks, on whichever CRM is connected. Covers all three APIs, provider detection, and the per-provider gotchas (HubSpot association ids, Salesforce SOQL, Pipedrive REST).
---

# CRM (HubSpot **/** Salesforce **/** Pipedrive)

The CRM is your source of truth. You read and update it through the **`request`**
tool, on whichever CRM the workspace connected:

```
request({ provider, method, path, query?, body? })
```

- **`provider:"hubspot"`** → base `https://api.hubapi.com`
- **`provider:"salesforce"`** → base is the org's instance URL (Nango injects it);
  paths start `/services/data/v59.0`
- **`provider:"pipedrive"`** → base `https://api.pipedrive.com/v1`
- Auth injected for all — never handle a token. Result `{ status, body }`.
  401/403 = that CRM isn't connected; say so. **Detect which is connected and
  stay on it** for the session.

The same five jobs, per provider: **read the open pipeline → know the stages →
read a deal's context → update fields → log a note + create a task.** Won/amount/
close-to-commercial changes are always approval-gated.

---

## HubSpot (CRM v3)

- **Open pipeline:** `POST /crm/v3/objects/deals/search` with
  `filterGroups:[{filters:[{propertyName:"dealstage",operator:"NEQ",value:"closedwon"},{propertyName:"dealstage",operator:"NEQ",value:"closedlost"}]}]`,
  `properties:["dealname","amount","dealstage","closedate","hs_lastmodifieddate"]`,
  `limit:100`. Page via `body.paging.next.after`.
- **Stages:** `GET /crm/v3/pipelines/deals` → `stages[]{id,label,metadata.isClosed}`.
- **Deal + context:** `GET /crm/v3/objects/deals/<id>?associations=contacts,companies`,
  then `POST /crm/v3/objects/contacts/batch/read`.
- **Update:** `PATCH /crm/v3/objects/deals/<id>` `{properties:{closedate,amount,dealstage}}`.
- **Note:** `POST /crm/v3/objects/notes` `{properties:{hs_note_body,hs_timestamp},associations:[{to:{id:"<dealId>"},types:[{associationCategory:"HUBSPOT_DEFINED",associationTypeId:214}]}]}` (214=note→deal).
- **Task:** `POST /crm/v3/objects/tasks` `{properties:{hs_task_subject,hs_timestamp,hs_task_status:"NOT_STARTED",hs_task_priority:"HIGH"},associations:[{to:{id:"<dealId>"},types:[{associationCategory:"HUBSPOT_DEFINED",associationTypeId:216}]}]}` (216=task→deal).

## Salesforce (REST + SOQL)

- **Open pipeline (SOQL):** `GET /services/data/v59.0/query?q=` +
  `SELECT Id,Name,StageName,Amount,CloseDate,LastModifiedDate FROM Opportunity WHERE IsClosed=false ORDER BY LastModifiedDate DESC`
  (URL-encode the query). Paging: follow `body.nextRecordsUrl`.
- **Stages:** query the picklist —
  `GET /services/data/v59.0/query?q=SELECT MasterLabel,IsClosed,IsWon FROM OpportunityStage ORDER BY SortOrder`.
- **Deal + context:** `GET /services/data/v59.0/sobjects/Opportunity/<id>`; related
  contacts via `SELECT Contact.Name,Contact.Title,Role FROM OpportunityContactRole WHERE OpportunityId='<id>'`.
- **Update:** `PATCH /services/data/v59.0/sobjects/Opportunity/<id>` `{StageName,Amount,CloseDate}` (returns 204, no body).
- **Note:** create a Task-as-log or a ContentNote; simplest is a completed Task —
  see below (Salesforce logs activities as Tasks).
- **Task:** `POST /services/data/v59.0/sobjects/Task` `{Subject,Description,ActivityDate,Status:"Not Started",Priority:"High",WhatId:"<opportunityId>"}` (`WhatId` links it to the deal; `Status:"Completed"` + `ActivityDate:today` logs a done activity).

## Pipedrive (REST v1)

- **Open pipeline:** `GET /deals?status=open&limit=100` → `body.data[]`. Paging:
  `body.additional_data.pagination.more_items_in_collection` + `start`.
- **Stages:** `GET /stages` → `{id,name,pipeline_id}`.
- **Deal + context:** `GET /deals/<id>`; persons via `GET /deals/<id>/persons`.
- **Update:** `PATCH /deals/<id>` `{stage_id,value,expected_close_date}`.
- **Note:** `POST /notes` `{content:"<call notes>",deal_id:<id>}`.
- **Task (activity):** `POST /activities` `{subject,due_date:"YYYY-MM-DD",type:"call",deal_id:<id>}`.

---

## Rules (all providers)

1. **Search before you edit.** Read the deal's real state (stage, amount,
   contacts) first.
2. **Least update.** Change only the fields that changed; never blow away data
   you didn't mean to touch.
3. **Stages are ids/picklist values, not free text.** Map them first (HubSpot
   `/pipelines/deals`, Salesforce `OpportunityStage`, Pipedrive `/stages`).
4. **Won / amount / close-to-commercial = approval.** Draft the change; don't
   apply without {{rep_name}}.
5. **Every call → a note; every commitment → a task/activity.** If it isn't in
   the CRM, it will be forgotten.
6. **Page through results** (HubSpot `paging.next.after`, Salesforce
   `nextRecordsUrl`, Pipedrive `more_items_in_collection`) — a partial read leads
   to wrong "no next step" conclusions.
7. **One CRM per session** — detect which is connected and stay on it.
