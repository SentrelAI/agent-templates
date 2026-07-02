---
name: internal-comms
description: Use to post to the team's internal Slack — escalation handoffs, urgent flags (outage/security/very-upset customer), and the daily queue summary. Internal-only; customer replies never go through Slack. Covers the Slack Web API through the mcp__apps__request proxy, resolving a channel name to an id, and formatting a useful escalation.
---

# Internal comms (Slack)

You use Slack to keep the human team in the loop — **internal only**. Customer
replies go through email (email-support skill), never here. You call the Slack
Web API through the **`request`** tool:

```
request({ provider:"slack", method:"POST", path:"/<method>", body:{ ... } })
```

- **Base is `https://slack.com/api`.** Auth is injected — never handle a token.
- Slack "methods" are path segments (`chat.postMessage`, `conversations.list`).
- Result is `{ status, body }`; Slack returns `body.ok` — **check it**, and read
  `body.error` when `ok:false` (Slack returns HTTP 200 even on logical errors
  like `channel_not_found` or `not_in_channel`).

## Resolve the channel id (once)

`{{escalation_channel}}` is a name like `#support-escalations`; `chat.postMessage`
wants a channel **id** (`C…`). Resolve it once and cache:

```
request({ provider:"slack", method:"POST", path:"/conversations.list",
  body:{ types:"public_channel,private_channel", limit:1000 } })
// → body.channels[] ; match .name == "support-escalations" (no leading #) → .id
```

If the bot isn't in the channel you get `not_in_channel` on post — say so and
ask the team to invite it; don't retry blindly.

## Post an escalation (the useful format)

An escalation a human can act on in five seconds:

```
request({ provider:"slack", method:"POST", path:"/chat.postMessage",
  body:{ channel:"<C…>",
    text:"🔺 Escalation: <one-line summary>",
    blocks:[
      { type:"section", text:{ type:"mrkdwn", text:
        "*Escalation — <category>*\n" +
        "*Customer:* <name / email>\n" +
        "*Ask:* <what they want>\n" +
        "*Tried:* <what I already did / KB gap>\n" +
        "*Linear:* <ENG-482 url or 'none'>\n" +
        "*Thread:* <gmail link>" } }
    ] } })
```

Always include a `text` fallback (used in notifications/previews) alongside
`blocks`.

## Urgent flags

For outage / security / data-loss / press-legal / a very upset customer, post
immediately (don't wait for the triage sweep) and make it unmistakable: lead
with `🚨 URGENT`, state the impact, and `@here` only when it truly warrants
interrupting people.

## Daily queue summary

At the end of a triage sweep, one line so the team has a pulse:

```
"📥 Queue: 4 open · 3 drafted (awaiting approval) · 1 escalated (ENG-482) · 0 breaching SLA"
```

## Rules

1. **Internal only.** Never send a customer-facing reply through Slack. Slack is
   for the team; email is for the customer.
2. **Check `body.ok`.** A 200 isn't success in Slack — read `ok` and handle
   `channel_not_found` / `not_in_channel` explicitly.
3. **Make escalations actionable:** customer, ask, what you tried, Linear link,
   thread link. No context = no fast handoff.
4. **Reserve urgency.** `🚨`/`@here` are for genuine outages/security/very-upset
   customers — overusing them trains the team to ignore them.
5. **One summary, not a play-by-play.** Post the queue summary once per sweep,
   not a message per ticket.
