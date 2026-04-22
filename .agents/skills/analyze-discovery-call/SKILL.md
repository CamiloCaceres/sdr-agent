---
name: analyze-discovery-call
description: Use when the user asks for a deeper post-call read-out — "how did my call with Acme go" / "analyze the {lead} discovery" — reads the already-captured call notes (or runs `capture-call-notes` first if missing), computes talk-ratio, scores pain-point discovery quality, flags missing MEDDPICC/BANT signals, names the biggest risk and the biggest opportunity, and proposes the next-step message.
---

# Analyze Discovery Call

Modeled on the "sales call analysis agent" workflow shape — not just
notes but diagnostic read-out: what went well, what didn't, what's
missing, what to do next.

## When to use

- User: "how did the Acme call go" / "analyze the discovery with
  {lead}" / "read-out on yesterday's call".
- Runs after `capture-call-notes` has structured the transcript.
- Called by a routine post-meeting for every discovery call.

## Steps

1. **Load structured notes** from `calls/{call_id}/notes.json`. If
   missing, run `capture-call-notes` first. If the raw transcript is
   also available, keep it accessible for quoting.
2. **Compute talk-ratio** (if transcript has per-speaker turns):
   internal-speaker-time / external-speaker-time. Flag >60% internal
   as a red flag — seller overtalking.
3. **Score pain-point discovery (0-3):**
   - 0 — no pain surfaced
   - 1 — pain named generically (no quote, no example)
   - 2 — pain named with a specific example or cost
   - 3 — pain named with specific example, cost, AND current
     workaround/attempted-fix
4. **MEDDPICC / BANT gap check.** Mark each of
   (Metrics, Economic-buyer, Decision-criteria, Decision-process,
   Paper-process, Identify-pain, Champion, Competition) as
   CONFIRMED / PARTIAL / MISSING from the transcript. (If the user
   prefers BANT — Budget / Authority / Need / Timeline — use that
   instead; capture their preference in `config/qualification.json`
   and remember.)
5. **Objection inventory.** For each objection raised in notes, mark
   HANDLED / PARTIAL / UNANSWERED. UNANSWERED means the objection was
   raised and the rep did not address it before moving on — highest
   priority follow-up.
6. **Sentiment arc.** Light read: did the call warm up, stay flat,
   cool off? Cite the turning moment if there was one.
7. **Biggest risk + biggest opportunity.** One sentence each. Risk =
   the thing most likely to kill the deal. Opportunity = the thing
   most likely to accelerate it. Both tied to specific quotes.
8. **Propose the next-step message.** Short draft the user can send
   within 24h — recap + answer the UNANSWERED objection + confirm
   action items + the next scheduled touchpoint. Use `config/voice.md`.
9. **Write** to `calls/{call_id}/analysis.md` with the full read-out
   and the draft message. Update
   `calls/{call_id}/notes.json` → `analysis: { painScore, talkRatio,
   gapList, riskOne, opportunityOne, analyzedAt }`.
10. **Summarize to user:** "Pain discovery: 2/3 · Talk ratio 48/52 ·
    3 qual gaps (Economic-buyer, Paper-process, Competition) · biggest
    risk: {one-line}. Draft follow-up ready at {path}."

## Never invent

Everything scored and flagged must tie back to a transcript quote. If
you can't cite it, mark MISSING / UNKNOWN — don't guess "probably went
well."

## Outputs

- `calls/{call_id}/analysis.md`
- Updates `calls/{call_id}/notes.json` with an `analysis` block
