---
name: reporting-and-variance
description: Use to turn the model into decisions — budget-vs-actual with every material variance explained (not just reported), runway math done on cash (not bookings), and a scannable weekly/monthly finance update delivered to the team. The synthesis layer on top of the model + metrics skills.
---

# Reporting & variance (method)

The model computes; this layer explains. A variance without a *why* is a number;
with a why, it's a decision.

## Budget vs. actual
For each category (from the model): **plan · actual · variance ($ and %) · the
why**. Rules of the craft:
- **Materiality first** — explain everything past the threshold in
  `fpa-standards.md`; summarize the rest in one line. Don't bury the one big
  miss under twenty trivial deltas.
- **The why is specific** — "marketing +$14k: the conference we approved in May"
  beats "spend was higher." If you can't explain a material variance, say so and
  flag it — an unexplained variance is a control problem.
- **Trend beats snapshot** — the same category drifting over three months is a
  bigger signal than one bad month. Note run-rate implications.

## Runway (on cash, honestly)
```
runway_months = current cash ÷ trailing-3mo average net burn
```
- **Cash basis** — from balance transactions/payouts (metrics skill), not MRR.
- State the burn definition (gross vs. net) and what's excluded.
- Report both "at plan" and "at current trend" when they differ — that gap *is*
  the message.
- Below the warning threshold → escalate immediately, with options (cut, raise,
  grow) framed for a human decision.

## The update (deliver where the team reads it)
One scannable message — Slack is a native channel for this agent, so post with
the built-in **`slack.post`** (no token, no apps proxy):
```
slack.post({ text:
  "📊 *Finance — week of 7/6*\n" +
  "• MRR $128k (+3.1% MoM) · NRR 104%\n" +
  "• Burn $210k/mo (plan $195k — over on infra, one-time migration)\n" +
  "• Runway: *14mo at plan · 12mo at current trend*\n" +
  "• Watch: churn ticked to 3.8% (assumption: 3%) — moves the base case if it holds\n" +
  "→ Decision needed: none this week. Model: <sheet link>" })
```
Email the fuller monthly pack (budget-vs-actual table + forecast deltas) to the
team; anything going to a **board/investor** is `share_externally` → a human
approves first.

## The board-deck draft (Google Slides, when connected)
For board/investor cycles, draft the finance section as slides — numbers from
the model, nothing invented, and it goes **nowhere without a human**:
```
request({ provider:"google_slides", method:"POST", path:"/presentations",
  body:{ title:"Board deck — finance (draft)" } })   // → body.presentationId
request({ provider:"google_slides", method:"POST", path:"/presentations/<id>:batchUpdate",
  body:{ requests:[
    { createSlide:{ slideLayoutReference:{ predefinedLayout:"TITLE_AND_BODY" } } },
    { insertText:{ objectId:"<placeholderId>", text:"MRR $128k (+3.1% MoM) · NRR 104% · Runway 14mo" } } ] } })
```
(Read `body.slides[].pageElements[].objectId` from `GET /presentations/<id>` to
target placeholders. Base already includes `/v1`.) One slide per message: the
metric headline, the variance story, the runway picture — sourced from the
model, linked back to it.

## Rules
1. **Every material variance gets a why** — or an explicit "unexplained, flagged."
2. **Runway on cash**, definition stated, plan-vs-trend both shown.
3. **Lead with what changed and what it means** — not a wall of numbers.
4. **Decision-oriented** — end with what needs deciding (or "none").
5. **External numbers need a human yes** — internal updates flow, board packs
   don't leave without approval.
