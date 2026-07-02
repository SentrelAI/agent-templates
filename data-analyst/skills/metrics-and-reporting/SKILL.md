---
name: metrics-and-reporting
description: Use to turn raw pulls into a decision-ready report — computing the metric + trend correctly, structuring an answer-first report with caveats and confidence, and delivering it to the team's Slack channel (native slack.post) or email. The synthesis layer on top of the data-source skills.
---

# Metrics & reporting

Data-source skills get you the numbers; this turns them into something a human
can act on. The whole value is here — a correct number, honestly framed.

## Compute the metric correctly
- Use the team's **definition** for each of {{key_metrics}} (see
  `analysis-standards.md`) — and if you must choose one, state which.
- Always compare to a baseline: **this period vs. prior period(s)**, or vs.
  target. A number with no comparison isn't a finding.
- Segment when the aggregate hides the story (by plan, cohort, channel) — and
  check whether segmenting *flips* the conclusion (Simpson's paradox).
- Round to the precision the data supports; state N for anything sample-based.

## Structure the report — answer first
1. **Headline** — the number + direction in one line ("Churn 4.2%, up from
   3.6%").
2. **What changed + likely why** — label the "why" a **hypothesis** unless you
   can show it.
3. **Caveat + confidence** — what would change the conclusion; how sure you are.
4. **The one action** it implies.
5. **Link the results tab/base** so anyone can re-run the math.

Keep it scannable — a screen, not a deck. Write the underlying table to a dated
results tab (spreadsheet-analysis skill) and link it.

## Deliver it
Slack is a **native channel** for this agent (installed via "Install in Slack" on
the Channels tab). Post with the built-in **`slack.post`** tool — no token, no
apps-proxy call:
```
slack.post({ text:
  "📊 *Weekly metrics — {{company_name}}*\n" +
  "• MRR $42.0k (+4.7% wow)\n• Churn *4.2%* (up from 3.6% — SMB-driven, hypothesis)\n" +
  "• Activation 61% (flat)\n_Caveat: churn N=210; SMB read is directional._\n" +
  "→ Action: dig into SMB onboarding. Details: <results tab link>" })
```
`text` supports Slack mrkdwn (`*bold*`, `<url|label>`, `\n`). Omit `channel` to
post to the agent's own channel ({{report_channel}}). For email delivery, send
the same structure to the team.

## Rules
1. **Answer first, caveats always.** Lead with the number; never bury it, never
   hide the uncertainty.
2. **Comparison or it's not a finding.** Every metric ships with a baseline.
3. **Label hypotheses.** "The why" is a claim; don't state it as proven.
4. **Reproducible.** Link the results tab; anyone should be able to check you.
5. **One action.** End with the single thing worth doing — not a list of ten.
6. **Surprising = double-check.** Verify a shocking number before you send it;
   a wrong headline erodes trust fast.
