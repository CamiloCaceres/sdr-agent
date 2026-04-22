---
name: generate-pipeline-report
description: Use when the user asks "what's my pipeline" / "how did the week go" / "run the weekly report" — pull pipeline data from the connected CRM via Composio (plus local `leads.json` for unsynced activity), compute stage distribution / movement / stalls / new-this-week, write a narrative summary to `reports/{period}/pipeline.md`, and optionally post to a connected team-chat channel.
---

# Generate Pipeline Report

Modeled on the "CRM reporting agent" workflow shape — not a raw dump,
a narrative that surfaces what's moving, what's stuck, and what needs
attention.

## When to use

- User: "pipeline report" / "how was this week" / "weekly rollup" /
  "state of the funnel".
- Called by a Friday-afternoon routine for the weekly rollup.
- Ad-hoc at any time; also supports daily/monthly windows.

## Steps

1. **Determine the window.** Default: this week (Mon 00:00 → now, user
   timezone). Accept "today" / "this week" / "last week" / "this
   month" / "last 30 days" / explicit date ranges.
2. **Pull CRM pipeline.** Run `composio search crm`. If a CRM is
   connected, pull opportunities/deals where the user is the owner,
   with fields: stage, amount, expected-close, last-activity-date,
   stage-entered-at. If no CRM is connected, fall back to `leads.json`
   + `calls.json` + `replies.json` and note the limitation clearly.
3. **Compute stage distribution** — count + total value per stage at
   the window's end.
4. **Compute movement in the window:**
   - **New this week** — created in window
   - **Progressed** — stage moved forward in window
   - **Regressed** — stage moved back
   - **Stalled** — no activity / no stage change in > stall-threshold
     (from `config/stall-threshold.json`, default 14 days)
   - **Closed-won / closed-lost** — in window
5. **Flag the top 5 deals to watch.** Sort by a composite of value ×
   recency × stage-late-ness. For each: one-line status + the single
   next best action.
6. **Rep metrics (if the user is the rep):** outbound sent,
   meetings-booked, reply-rate, conversion stage-to-stage. Pull from
   local `leads.json` / `replies.json` / `calls.json`.
7. **Risk callouts.** 2-3 things the user should act on this
   week — stalls, missed follow-ups, deals approaching expected-close
   without recent activity.
8. **Write narrative** to `reports/{YYYY-Www}/pipeline.md` with:
   - Header (period, timestamp, source)
   - Summary (1 paragraph, numbers embedded)
   - Stage distribution table
   - Movement table
   - Top 5 deals to watch
   - Risk callouts
   - Rep metrics
   - Data-sources footer with `fetchedAt`
9. **Append to `reports.json`** index: `{ period, generatedAt,
   summary, deltasVsPrior }`. If a prior report exists, compute and
   name the biggest delta week-over-week in the summary.
10. **Team-notify (opt-in).** If `config/notify.json` has
    `pipelineReports: true` and a team-chat channel is connected,
    post the summary paragraph + link. Otherwise skip.

## Never overclaim

If data is missing (no CRM connected, or CRM returns nothing for a
field), say so in the report: "Source: local activity only — connect
a CRM for full pipeline view." Don't synthesize numbers.

## Outputs

- `reports/{YYYY-Www}/pipeline.md`
- Appends to `reports.json`
- Optional: team-chat post
