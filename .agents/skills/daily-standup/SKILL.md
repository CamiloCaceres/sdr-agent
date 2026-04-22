---
name: daily-standup
description: Use when the user opens the app or asks "what's on my plate" / "morning brief" / "daily standup" — roll up today's play list: drafts awaiting approval, replies to review, meetings today, stalled threads that hit their threshold, new enrichments from overnight routines, and the 3 highest-leverage actions I'd take today in priority order.
---

# Daily Standup

Short, ranked, action-oriented. The point is to turn "what do I do
today" into a 60-second read.

## When to use

- User: "what's my day look like" / "morning brief" / "standup" / "what
  should I work on" / opens the agent first thing.
- Called by the morning routine daily (default 8:00 local).

## Steps

1. **Load the current state** in parallel: `leads.json`, `replies.json`,
   `calls.json`, `reports.json`. Compute the snapshot as of now.
2. **Needs-your-approval queue:**
   - Drafts in `leads/*/outreach-draft.md` written in last 48h and not
     yet marked approved.
   - Reply drafts in `replies/*/draft.md` awaiting review.
   - Warm-path intro-asks pending send.
   Count each.
3. **Meetings today.** Query the connected calendar via
   `composio search calendar` for today's events that match a lead
   (attendee email/domain in `leads.json`). For each, load the lead
   and offer a one-line prep note + a link to the dossier.
4. **Stalled threads.** From `leads.json`, find leads where
   `lastContactedAt` > stall threshold and `status` is active (not
   closed-lost / unsubscribed). Rank by fit score × recency of last
   engagement. Take top 3.
5. **Overnight enrichment.** Leads enriched in the last 24h —
   summarize count + top-3 highest-fit.
6. **Rep rhythm snapshot.** Numbers: drafts sent-last-7d,
   replies-last-7d, reply-rate, meetings-held-last-7d, meetings-booked-
   today. Flag any metric that moved meaningfully vs last week.
7. **Compute the top-3 moves for today** — cross-ranking of above.
   Each is a single action with the skill-to-invoke: e.g.
   - "Approve & send the 3 drafts to GREEN leads from overnight
     enrichment → `/leads/...`"
   - "Respond to the OBJECTION from {lead} (priced) — draft ready"
   - "Prep for 2pm with {lead} — dossier + talking points"
8. **Write** to `daily-brief.md` (overwrite — single rolling file),
   structured:
   - Top 3 moves (with skill calls)
   - Needs approval (counts + quick links)
   - Today's meetings
   - Stalled worth recovering (top 3)
   - Overnight enrichment highlights
   - Rep rhythm numbers
9. **Output in chat** — the SAME content, compact, readable in 30
   seconds. User can click through to `daily-brief.md` for the full
   version.

## Empty-state grace

If there's genuinely nothing in any bucket, say "Quiet day — good time
to add leads or refresh ICP." Don't pad with busywork.

## Outputs

- `daily-brief.md` (rolling single file, overwritten each run)
