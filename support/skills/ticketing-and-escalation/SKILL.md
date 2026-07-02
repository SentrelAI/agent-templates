---
name: ticketing-and-escalation
description: Use when a support ticket is a real bug or feature request that engineering needs — searching for an existing Linear issue before creating a duplicate, filing a clean issue with a repro, commenting to link it back to the customer thread, and reading status to update the customer. Covers Linear's GraphQL API through the mcp__apps__request proxy.
---

# Ticketing & escalation (Linear)

When a ticket is a genuine bug or a feature request, you record it in Linear so
engineering sees it and the customer thread can be linked to a real issue. You
talk to Linear's **GraphQL** API through the **`request`** tool:

```
request({ provider:"linear", method:"POST", path:"/graphql",
  body:{ query:"<graphql>", variables:{ ... } } })
```

- **Base is `https://api.linear.app`**, the single endpoint is `/graphql`
  (always `POST`). Auth is injected — never handle a token.
- The result is `{ status, body }`; the data is in `body.data`, errors in
  `body.errors` (check it — GraphQL returns 200 even on validation errors).

## 1. Find the team (once)

Issues belong to a team. Resolve the team id you file into:

```graphql
query { teams { nodes { id name key } } }
```

Use the team whose `key`/`name` matches engineering (often `ENG`). Cache it for
the session.

## 2. Search before you create (avoid duplicates)

Never file a second issue for a bug that's already tracked. Search first:

```graphql
query Search($q: String!) {
  issueSearch(query: $q) {
    nodes { id identifier title state { name } url }
  }
}
```

`variables: { q: "<key symptom, e.g. 'export CSV 500 error'>" }`. If a close
match exists, **comment on it** (step 4) instead of creating a new one, and
tell the customer it's a known issue.

## 3. Create a clean issue

A good bug report is the difference between a fix and a "can't repro". Include
what you know:

```graphql
mutation Create($input: IssueCreateInput!) {
  issueCreate(input: $input) { success issue { id identifier url } }
}
```

```
variables: {
  input: {
    teamId: "<teamId>",
    title: "<short, specific symptom>",
    description: "**Reported by:** customer via support (thread <link/id>)\n\n**What happens:** …\n**Steps to reproduce:**\n1. …\n2. …\n**Expected:** …\n**Actual:** …\n**Affected account/plan:** …\n**Frequency / first seen:** …",
    priority: 2   // 0 none, 1 urgent, 2 high, 3 medium, 4 low — urgent only for outages/security
  }
}
// → body.data.issueCreate.issue.identifier (e.g. ENG-482) + .url
```

Label it a feature request vs. bug if the team uses labels — fetch label ids
with `query { issueLabels { nodes { id name } } }` and pass `labelIds`.

## 4. Link it back to the customer thread

Add a comment so the issue carries the support context (and vice-versa: put the
Linear `identifier` in your email draft + the Gmail label):

```graphql
mutation Comment($input: CommentCreateInput!) {
  commentCreate(input: $input) { success }
}
```

`variables: { input: { issueId: "<issueId>", body: "Another customer hit this — thread <link>. +1 affected." } }`

## 5. Read status to update the customer

Before telling a customer "it's fixed", verify:

```graphql
query($id: String!) { issue(id: $id) { identifier state { name type } url } }
```

`state.type` is `completed` when it's actually done. Don't promise a fix or a
date the issue doesn't support.

## Rules

1. **Search before create — every time.** Duplicate issues bury signal. `+1`
   an existing one instead.
2. **Write the repro you'd want to receive:** steps, expected vs. actual,
   account/plan, frequency. Vague reports get closed as "can't repro".
3. **Priority is earned.** `urgent` (1) is for outages, data loss, or security
   only. Most support bugs are high (2) or medium (3).
4. **Link both ways.** Linear identifier in the customer thread; thread link in
   the issue. That's how the loop closes later.
5. **Report status honestly.** Only say "fixed" when `state.type == completed`.
   If it's still open, tell the customer it's logged and being worked, without
   inventing a date.
