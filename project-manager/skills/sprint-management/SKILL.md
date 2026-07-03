---
name: sprint-management
description: Use to read and maintain the Linear board — pulling issues by state/assignee/cycle, spotting stale and blocked work, updating status from clear signals, commenting to nudge an owner, and filing tracked follow-ups. Reassign/close are NOT done here. Covers Linear's GraphQL API through the mcp__apps__request proxy.
---

# Sprint management (Linear)

You keep the board honest through the **`request`** tool:

```
request({ provider:"linear", method:"POST", path:"/graphql",
  body:{ query:"<graphql>", variables:{ ... } } })
```

- **Base `https://api.linear.app`**, single endpoint `/graphql` (always POST).
  Auth injected. Data in `body.data`; **check `body.errors`** (200 even on
  validation errors).

## Read the active board
```graphql
query($teamId: String!) {
  team(id: $teamId) {
    activeCycle {
      issues {
        nodes {
          identifier title url priority
          state { name type }            # type: backlog/unstarted/started/completed/canceled
          assignee { name }
          updatedAt dueDate
          labels { nodes { name } }      # a "Blocked" label is a common signal
        }
      }
    }
  }
}
```
(Get `teamId` once: `query { teams { nodes { id key name } } }`.) For a broader
view use `issues(filter:{ state:{ type:{ eq:"started" } } })`.

## Spot staleness + blockers (the daily job)
From the pulled issues, flag:
- **stale** — `state.type == "started"` and `updatedAt` older than ~2 days.
- **blocked** — a `Blocked` label, or a blocking relation
  (`relations { nodes { type relatedIssue { identifier } } }`).
- **overdue** — `dueDate` in the past and not completed.
These are what you nudge (standup-and-unblocking skill) and surface in status.

## Update status from a clear signal
When the signal is unambiguous (a PR merged, an owner said "done"), move the
state — never invent progress:
```graphql
mutation($id: String!, $stateId: String!) {
  issueUpdate(id: $id, input: { stateId: $stateId }) { success }
}
```
(Fetch valid state ids: `query { workflowStates { nodes { id name type } } }`.)

## Nudge / file a follow-up (allowed) — but never reassign/close
Comment to nudge:
```graphql
mutation($input: CommentCreateInput!) { commentCreate(input:$input){ success } }
```
File a tracked blocker/follow-up:
```graphql
mutation($input: IssueCreateInput!) { issueCreate(input:$input){ success issue{ identifier url } } }
```
**Do NOT** call `issueUpdate` to change `assigneeId`, scope, or due date, and do
NOT `issueArchive`/delete — those are gated (reassign/rescope = ask, close/delete
= block). Surface them for a human instead.

## Rules
1. **Read reality, don't invent it.** Only move status on a clear signal.
2. **Stale + blocked are the priority.** Find them every pass; a stale started
   issue is a slip forming.
3. **Nudge + follow-up only.** Never reassign, rescope, close, or delete — flag
   for the owner/lead.
4. **Check `body.errors`** on every call.
