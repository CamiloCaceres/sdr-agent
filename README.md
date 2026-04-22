# SDR Agent

Your AI Sales Development Rep for Houston. A standalone agent shaped
around **validated sales-automation workflow patterns** — each skill
maps to a proven shape (contact enrichment, fit scoring, warm-path
search, research-grounded email, reply triage, call capture, discovery
analysis, battlecard generation, pipeline reporting).

Never sends without your approval. Never closes deals. Hands warm
meetings to you.

## Who this is for

Founders and sales leaders doing outbound themselves, or running a
small SDR function who want an AI teammate. You connect your stack
via **Composio** — any inbox, profile provider, enrichment tool, CRM,
meeting-notes app, calendar, team-chat — and the agent discovers tool
slugs at runtime with `composio search`. One agent adapts to whatever
you've got plugged in.

## Install

In Houston: **Add from GitHub** → paste this repo's URL. Houston
installs the agent into its own workspace and seeds the empty indexes
so the dashboard opens clean.

On first open, the Activity tab shows an **"Onboard me"** card in the
"Needs you" column — click it and send any message to start the
3-question setup (ICP, product + pitch, voice). After that the agent
is ready to work.

## First prompts

- `Onboard me` — 3-question setup (~90 seconds).
- `Enrich {person} at {company}` — firmographics, role, signals.
- `Score fit for {lead}` — GREEN / YELLOW / RED against your ICP.
- `Warm paths to {company}` — who in my network can intro.
- `Draft email to {lead}` — research-grounded first-touch.
- `Triage my inbox` — classify new replies, draft responses.
- `Process my call with {lead}` — capture + analyze a discovery call.
- `Battlecard for {lead} vs {competitor}` — per-prospect positioning.
- `Weekly pipeline report` — narrative rollup from CRM + activity.
- `Daily standup` — top-3 moves today.

## Skills

| Skill | What it does |
|-------|--------------|
| `onboard-me` | 3-question setup: ICP · product + pitch · voice |
| `enrich-contact` | Firmographics + role + signals + CRM sync |
| `score-icp-fit` | GREEN / YELLOW / RED against your ICP |
| `find-references` | Warm paths + reference customers |
| `draft-grounded-email` | Outreach anchored in a cited signal |
| `triage-inbound-reply` | Classify + draft response to inbound |
| `capture-call-notes` | Transcript → structured notes + CRM activity |
| `analyze-discovery-call` | Talk ratio, pain score, qual gaps, risk/opp |
| `build-battlecard` | Per-prospect competitive positioning |
| `generate-pipeline-report` | Narrative pipeline rollup |
| `daily-standup` | Top-3 moves, approvals queue, meetings, stalls |

## Routines

Houston can run recurring jobs against this agent — see
[`routines.md`](./routines.md) for the full recipe library. Suggested
defaults:

- **Morning standup** — daily 08:00 → `daily-standup`
- **Inbox triage** — every 2h work hours → `triage-inbound-reply`
- **Stalled sweep** — daily 09:00 → stalled-thread recovery drafts
- **Weekly pipeline** — Fri 16:00 → `generate-pipeline-report`
- **Nightly enrichment** — daily 19:00 → enrich + score any new leads

Ask me "set up my routines" and I'll create them.

## Design rules

- **Composio-only transport.** Every external system flows through
  Composio. No hardcoded tool names in skills.
- **Progressive config.** Only `onboard-me` batches (3 questions).
  Everything else asked just-in-time when a skill first needs it.
- **Modality ranking.** For any input, the agent prefers a connected
  app > file/URL > paste. "Give me context" is never the ask —
  it's "paste, drop a file, or point to a connected app."
- **Never sends, never closes.** Every outbound goes through a
  review queue. Every hand-off waits for your approval.
- **No custom dashboard.** Uses Houston's built-in tabs: **Activity**
  (Kanban), **Job Description** (this README + CLAUDE.md surfaced),
  **Routines** (Houston's routines manager), **Files**,
  **Integrations**.

## Data model

See [`data-schema.md`](./data-schema.md) for the full schema. In
short:
- `config/` — what the agent has learned about you (ICP, voice,
  product, etc.). Written at runtime; never shipped in the repo.
- `leads.json`, `replies.json`, `calls.json`, `reports.json`,
  `battlecards.json` — fast indexes at the agent root.
- `leads/{slug}/`, `calls/{id}/`, `reports/{period}/` — per-entity
  detail folders.

## License

MIT.
