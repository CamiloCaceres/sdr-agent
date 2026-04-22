# SDR — Data Schema

All records share these base fields:

```ts
interface BaseRecord {
  id: string;          // UUID v4
  createdAt: string;   // ISO-8601 UTC
  updatedAt: string;   // ISO-8601 UTC
}
```

All writes are atomic: write to a sibling `*.tmp` file, then rename
onto the target path. Never edit in-place. Never write anywhere under
`.houston/<agent>/` — the Houston file watcher skips those paths and
reactivity breaks.

---

## Config — what the agent learns about the user

Nothing in `config/` is shipped in the repo. Every file appears at
runtime, written by `onboard-me` or by progressive capture inside
another skill the first time it needs the value.

### `config/profile.json` — written by `onboard-me`

```ts
interface Profile {
  userName: string;
  company: string;
  team?: string;
  onboardedAt: string;        // ISO-8601
  status: "onboarded" | "partial";
}
```

### `config/icp.json` — written by `onboard-me` or `score-icp-fit`

```ts
interface Icp {
  industry: string[];
  size: { min?: number; max?: number; band?: string };
  role: string[];              // e.g. ["Head of RevOps", "VP Marketing"]
  stage?: string[];            // e.g. ["Series B", "Growth"]
  triggers: string[];          // signal patterns (hiring, funding, stack change)
  examples: string[];          // named anchor accounts
  fitCriteria: {
    hard: string[];            // criteria that MUST hit for GREEN
    soft: string[];            // criteria that soft-rank
  };
  source: "paste" | "url" | "file" | "connected-crm";
  capturedAt: string;
}
```

### `config/product.json` — written by `onboard-me`

```ts
interface Product {
  name: string;
  oneLine: string;
  pitch30s: string;
  painsSolved: string[];
  differentiators: string[];
  pricing?: { model: string; range?: string };
  source: "paste" | "url" | "file";
  capturedAt: string;
}
```

### `config/voice.md` — written by `onboard-me` (and refreshable)

Markdown. 3-5 verbatim samples of the user's outbound style plus a
short "tone notes" block (greeting habits, closing habits, sentence
length, formality, quirks).

### Other config files (written just-in-time)

| File | Written by | When |
|------|------------|------|
| `config/objections.json` | `triage-inbound-reply` | First time a priced/timing/etc objection comes back |
| `config/stall-threshold.json` | `generate-pipeline-report` or stalled routine | First time the user invokes either and hasn't set it (default 14d) |
| `config/qualification.json` | `analyze-discovery-call` | First run — user picks MEDDPICC / BANT / own list |
| `config/differentiators.json` | `build-battlecard` | First battlecard — captures honest strengths + weaknesses vs peers |
| `config/reference-customers.json` | `build-battlecard` or `find-references` | First time the user names-a-reference |
| `config/notify.json` | Any skill that optionally pings team | Opt-in only. Maps categories → channel |
| `config/notes-sync.json` | `capture-call-notes` | Opt-in; toggles notes-app upsert |

---

## Domain data — what the agent produces

Sits at the agent root. These are the fast indexes + per-entity
detail folders the skills read and write.

### `leads.json` — lead index

```ts
interface LeadRow extends BaseRecord {
  slug: string;                // kebab(firstname-lastname-company)
  firstName: string;
  lastName: string;
  email?: string;
  company: string;
  title?: string;
  industry?: string;
  headcount?: string;
  source: string;
  status: "new" | "enriched" | "scored" | "drafted" | "engaged"
        | "meeting-ask" | "meeting-held" | "closed-won"
        | "closed-lost" | "unsubscribed" | "inbound-unmatched";
  fitScore?: "GREEN" | "YELLOW" | "RED";
  fitReasoning?: string;
  fitScoredAt?: string;
  enrichedAt?: string;
  lastContactedAt?: string;
  lastReplyAt?: string;
  lastDraftedAt?: string;
  draftCount: number;
  replyCount: number;
  sources: string[];           // URLs used during enrichment
  crmRecordUrl?: string;
  crmSyncedAt?: string;
  topWarmPath?: string;
  warmPathsFoundAt?: string;
  recentCalls?: { callId: string; date: string; summary: string }[];
}
```

Written by: `enrich-contact`, `score-icp-fit`, `draft-grounded-email`,
`triage-inbound-reply`, `capture-call-notes`, `find-references`.

### `leads/{slug}/` — per-lead detail

| File | Written by | Content |
|------|------------|---------|
| `lead.json` | `enrich-contact` | Full structured dossier |
| `dossier.md` | `enrich-contact` | Human-readable dossier |
| `fit.md` | `score-icp-fit` | GREEN/YELLOW/RED scorecard |
| `warm-paths.md` | `find-references` | Ranked warm paths + intro-ask drafts |
| `outreach-draft.md` | `draft-grounded-email` | Subject + body + grounding + alts |
| `battlecard-vs-{competitor}.md` | `build-battlecard` | Per-prospect competitor card |

### `replies.json` — inbound reply index

```ts
interface ReplyRow extends BaseRecord {
  leadSlug?: string;                  // null if inbound-unmatched
  from: { name: string; email: string };
  receivedAt: string;
  classification: "INTERESTED" | "ASKING-QUESTION" | "OBJECTION"
                | "NOT-NOW" | "NOT-INTERESTED" | "UNSUBSCRIBE"
                | "OUT-OF-OFFICE" | "WRONG-PERSON";
  actionItems: string[];
  hasDraft: boolean;
  status: "triaged" | "drafted" | "approved" | "sent" | "dismissed";
}
```

### `replies/{id}/` — per-reply detail

| File | Content |
|------|---------|
| `reply.json` | Raw inbound + classification + action items |
| `draft.md` | Draft response (when classification is non-terminal) |

### `calls.json` — call index

```ts
interface CallRow extends BaseRecord {
  callId: string;                      // kebab(date-primary-external-name)
  leadSlug: string;
  date: string;                        // ISO-8601
  duration?: number;                   // minutes
  attendeesInternal: string[];
  attendeesExternal: string[];
  nextStep: string;                    // "TBD" if not agreed
  painCount: number;
  objectionCount: number;
  actionItemsTheirs: number;
  actionItemsOurs: number;
  analyzed: boolean;
}
```

### `calls/{call_id}/` — per-call detail

| File | Written by | Content |
|------|------------|---------|
| `notes.json` | `capture-call-notes` | Structured notes (agenda, pains, objections, decisions, actions, next step) |
| `notes.md` | `capture-call-notes` | Human-readable notes |
| `analysis.md` | `analyze-discovery-call` | Talk-ratio, pain score, qual gaps, risk/opp, draft follow-up |

### `reports.json` + `reports/{period}/`

```ts
interface ReportRow extends BaseRecord {
  period: string;                      // "2026-W17" or "2026-04" or "2026-04-21"
  kind: "weekly" | "monthly" | "daily" | "custom";
  generatedAt: string;
  summary: string;
  deltasVsPrior?: string;
  path: string;
}
```

`reports/{period}/pipeline.md` — narrative pipeline report (written by
`generate-pipeline-report`).

### `battlecards.json` + files

```ts
interface BattlecardRow extends BaseRecord {
  slug: string;                        // slug-vs-competitor
  leadSlug: string;
  competitor: string;
  generatedAt: string;
  sources: string[];
}
```

Battlecard files live under `leads/{slug}/battlecard-vs-{competitor}.md`.

### `daily-brief.md` — rolling standup file

Single file. Overwritten each run of `daily-standup`. Contains
top-3 moves, approvals queue, today's meetings, stalled-worth-
recovering, rep rhythm numbers.
