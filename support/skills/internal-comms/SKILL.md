---
name: internal-comms
description: Use to post to the team's Slack — escalation handoffs, urgent flags (outage/security/very-upset customer), and the daily/weekly queue summary. Internal-only; customer replies never go through Slack. Uses the native slack.post tool (the Slack channel installed for this agent), not the apps proxy.
---

# Internal comms (Slack)

You keep the human team in the loop over Slack — **internal only** (customer
replies go through email, never here). Slack is a **native channel** for this
agent: it's installed once via **"Install in Slack"** on the agent's Channels
tab, and you post with the built-in **`slack.post`** tool. There's no token,
no channel lookup, no apps-proxy call.

```
slack.post({ text, channel?, thread_ts? })
```

- **`text`** — the message. Slack **mrkdwn** works: `*bold*`, `_italic_`,
  `<url|label>`, `:emoji:`, line breaks with `\n`.
- **`channel`** — omit it. By default the message goes to **your own bound
  channel** (the one you were installed into — this is your escalation/team
  channel). Only pass `channel` when replying in a *different* channel (e.g. a
  thread where you were @mentioned — use the `channel` from the incoming
  message metadata).
- **`thread_ts`** — pass the parent message's `ts` to reply in-thread.
- The tool posts on success and throws on failure (so if Slack isn't installed
  for this agent, you'll get a clear error — tell the user to install it via the
  Channels tab).

## Post an escalation (the useful format)
One post a human can act on in five seconds:

```
slack.post({ text:
  "🔺 *Escalation — <category>*\n" +
  "*Customer:* <name / email>\n" +
  "*Ask:* <what they want>\n" +
  "*Tried:* <what I already did / KB gap>\n" +
  "*Linear:* <ENG-482 url or 'none'>\n" +
  "*Thread:* <gmail link>" })
```

## Urgent flags
For outage / security / data-loss / press-legal / a very upset customer, post
immediately (don't wait for the triage sweep) and make it unmistakable — lead
with `🚨 *URGENT*` and state the impact. Use `<!here>` in the text only when it
truly warrants interrupting people.

## Daily / weekly summary
One line so the team has a pulse (from the triage sweep or the weekly report):

```
slack.post({ text: "📥 Queue: 4 open · 3 drafted (awaiting approval) · 1 escalated (ENG-482) · 0 breaching SLA" })
```

## Rules
1. **Internal only.** Never send a customer-facing reply via Slack — that's
   email. Slack is for the team.
2. **Default to your own channel.** Omit `channel`; only override to reply in a
   specific thread where you were mentioned.
3. **Make escalations actionable:** customer, ask, what you tried, Linear link,
   thread link. No context = no fast handoff.
4. **Reserve urgency.** `🚨`/`<!here>` are for genuine outages/security/very-upset
   customers — overusing them trains the team to ignore them.
5. **One summary, not a play-by-play.** Post the queue summary once per sweep *that had activity* — a sweep with nothing new posts nothing,
   not a message per ticket.
