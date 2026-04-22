---
name: capture-call-notes
description: Use when the user pastes a meeting transcript, drops a recording/transcript file, or asks to "process my call with {lead}" — extract structured notes (attendees, agenda actual-vs-intended, decisions, action items with owners, pain points raised, objections, next step), write them to `calls/{id}/notes.md` + `calls/{id}/notes.json`, update the linked lead's dossier, and optionally upsert to a connected CRM or note-storage app.
---

# Capture Call Notes

Modeled on the "meeting transcript enrichment" workflow shape — turn a
raw transcript into structured, queryable, CRM-ready notes.

## When to use

- User: "process my call with Acme" / pastes a transcript / drops a
  `.txt` or `.vtt` file / "capture notes from yesterday's meeting".
- Called by a routine pulling from a connected meeting-notes app
  (Fathom, Fireflies, Grain, Circleback, etc. — discovered via
  `composio search meeting-notes`).

## Steps

1. **Source the transcript.** If pasted, use it. If a file, read it. If
   the user points to a connected provider, run `composio search` for
   a list/search tool, find the most recent meeting matching the
   user's description, and pull the transcript.
2. **Identify the meeting.** Extract date/time, attendees (separate
   internal vs external), duration, meeting title if available.
3. **Match to a lead.** Find the external attendee(s) in `leads.json`
   by name + company. If not found, create a minimal lead row from the
   transcript and mark `source: "meeting-first-contact"`.
4. **Assign an id.** `call_id = kebab(date-primary-external-name)`.
5. **Extract structured notes:**
   - **Agenda actual** — what was actually discussed (not what the
     agenda said).
   - **Pain points raised** — specific phrases in their words, with
     the transcript quote.
   - **Objections raised** — price, timing, authority, fit — quoted.
   - **Decisions** — anything agreed during the call.
   - **Action items** — owner + what + by when. Split internal vs
     external.
   - **Next step** — the single next scheduled touchpoint (if agreed)
     or "next step TBD."
6. **Write structured:** `calls/{call_id}/notes.json` with the full
   schema + `calls/{call_id}/notes.md` as the human-readable rollup.
7. **Update the lead dossier.** Append to
   `leads/{slug}/lead.json` → `recentCalls: [...]` (just id + date +
   one-line summary). Update `lastContactedAt`, `status` (likely
   "meeting-held" or "follow-up-owed").
8. **Add to `calls.json` index** with id, date, lead slug, attendees,
   next-step summary.
9. **CRM sync (if connected).** Run `composio search crm`. If linked,
   upsert a meeting/activity record on the lead's CRM contact.
   Include attendees + date + action items + next step. Never sync
   verbatim transcript unless the user explicitly opts in (it's
   usually out of scope for CRM notes fields).
10. **Note-app sync (if connected).** If the user has a connected
    notes/docs app AND `config/notes-sync.json` says to push, create
    the note there. Otherwise skip silently.
11. **Summarize to the user:** "Captured. 3 pain points, 2 action
    items (1 yours: {X}, 1 theirs: {Y}), next step: {Z}. CRM synced."

## Never invent

If a field isn't clearly present in the transcript, write "not stated"
— never fill in plausible-sounding pain points or owners. The
downstream cost of hallucinated call notes is high.

## Outputs

- `calls/{call_id}/notes.json` (structured)
- `calls/{call_id}/notes.md` (human-readable)
- Updates `leads/{slug}/lead.json` and `leads.json`
- Updates `calls.json` index
- Optional: CRM activity upsert
- Optional: notes-app entry
