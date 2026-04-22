# SDR — Routine Recipes

Houston's **Routines** feature runs recurring jobs on a schedule
(cron-style or heartbeat). Each routine fires a Claude session with a
specific prompt against this agent. Ask me to "set up my routines"
and I'll create these via Houston's routines system, or add them
manually from the **Routines** tab inside the agent.

Each recipe below lists:
- **Schedule** — when it fires
- **Approval mode** — `auto` (I act end-to-end) or `review` (I propose,
  you approve). For anything that touches outbound, always `review`.
- **Prompt** — the exact prompt that runs
- **Expected output** — the skill invoked and what shows up

---

## 1. Morning standup — daily 08:00 local

- **Schedule:** daily at 08:00, user's local timezone
- **Approval mode:** `auto`
- **Prompt:**
  > Run `daily-standup`. Output the top-3 moves today, the approvals
  > queue counts, today's meetings, and the top-3 stalled threads
  > worth recovering. Write to `daily-brief.md` and post the compact
  > version in chat.
- **Output:** `daily-brief.md` updated; chat summary posted.

## 2. Inbox triage — every 2 hours, work hours

- **Schedule:** every 2 hours between 08:00 and 18:00 local, weekdays
- **Approval mode:** `review` (drafts queued, never sent)
- **Prompt:**
  > Run `triage-inbound-reply`. Pull new unread threads from the
  > connected inbox where I'm the recipient and the latest message
  > isn't mine, from the last 4 hours. For each, classify, extract
  > action items, and draft a voice-matched response for non-terminal
  > categories. Do NOT send. Summarize counts in chat only if there
  > are replies — stay silent on a zero-reply run.
- **Output:** `replies/*/reply.json` + `replies/*/draft.md`; index
  updated in `replies.json`.

## 3. Stalled thread sweep — daily 09:00

- **Schedule:** daily at 09:00 local
- **Approval mode:** `review`
- **Prompt:**
  > Read `leads.json`. Find leads where `status` is active (not
  > closed-lost / unsubscribed) and `lastContactedAt` is older than
  > my stall threshold (read from `config/stall-threshold.json`,
  > default 14 days). For each of the top 5 (ranked by `fitScore`
  > GREEN > YELLOW and recency of last engagement), write a short
  > recovery-attempt draft to `leads/{slug}/recovery-draft.md` in
  > my voice. Do NOT send. Summarize the list in chat — bullet per
  > lead with the stall reason and the opening line of the draft.
- **Output:** `leads/{slug}/recovery-draft.md` per flagged lead; chat
  summary.

## 4. Weekly pipeline rollup — Friday 16:00

- **Schedule:** weekly, Friday at 16:00 local
- **Approval mode:** `auto`
- **Prompt:**
  > Run `generate-pipeline-report` for this week. Compute stage
  > distribution, movement, stalls, new-this-week, and top-5 deals
  > to watch. Write to `reports/{YYYY-Www}/pipeline.md` and append to
  > `reports.json`. If `config/notify.json` has `pipelineReports:
  > true` and a team-chat is connected, also post the summary
  > paragraph + link. Otherwise stay in chat.
- **Output:** `reports/{period}/pipeline.md`; reports index updated;
  optional team-chat post.

## 5. Nightly enrichment — daily 19:00

- **Schedule:** daily at 19:00 local
- **Approval mode:** `auto`
- **Prompt:**
  > Read `leads.json`. Find any leads added in the last 24 hours
  > where `enrichedAt` is missing. For each, run `enrich-contact`.
  > Cap at 25 per run to respect API quotas. After each run,
  > immediately run `score-icp-fit` on the enriched lead so the
  > morning standup can prioritize by fit. Summarize counts in chat
  > at the end: "{N} enriched · {G} GREEN · {Y} YELLOW · {R} RED."
- **Output:** `leads/{slug}/lead.json` + `leads/{slug}/fit.md` per
  lead; `leads.json` updated.

## 6. Post-meeting capture — event-driven (optional)

- **Schedule:** triggered when a meeting ending in the connected
  calendar has a transcript available in the connected meeting-notes
  app (Composio trigger)
- **Approval mode:** `review`
- **Prompt:**
  > A meeting just ended and a transcript is available. Run
  > `capture-call-notes` on it, then `analyze-discovery-call` on the
  > captured notes. Write a short handoff in chat with the biggest
  > risk, biggest opportunity, and the draft follow-up for my review.
- **Output:** `calls/{id}/notes.json` + `calls/{id}/notes.md` +
  `calls/{id}/analysis.md`.

---

## How to set these up

Ask me: "Set up my routines — all five." I'll create them via Houston
(names, schedules, prompts, approval modes) and you confirm.

You can also open the **Routines** tab in this agent and add them
manually — the prompts above are ready to copy-paste.

## Approval-mode rule

Anything that could produce outbound-visible side effects
(`review` not `auto`):
- reply drafts (triage, recovery)
- outreach drafts
- team-chat posts IF they include prospect names
Anything that's read-only or strictly internal
(`auto` is fine):
- standup generation
- pipeline reports (internal narrative only)
- enrichment + fit scoring (local writes only, no outbound)
