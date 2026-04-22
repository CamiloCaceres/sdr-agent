---
name: triage-inbound-reply
description: Use when a new reply arrives via any Composio-connected inbox (pulled on a routine or at the user's request) or the user pastes an incoming reply — classify as interested / asking-question / objection / not-now / not-interested / unsubscribe / out-of-office / wrong-person, extract action items, and for non-terminal categories draft a response in the user's voice.
---

# Triage Inbound Reply

Classify inbound, pull context, draft a voice-matched response, queue
for human approval.

## When to use

- User: "check my replies" / "triage new inbound" / pastes an email.
- Routine calls me every N hours to pull from the connected inbox.
- `recover-stalled-thread` calls me to classify responses to a
  re-engagement attempt.

## Steps

1. **Source the replies.** If the user pasted one, use that. Otherwise
   run `composio search` for the connected inbox's search/list tool
   and pull unread threads where the user is a recipient and the most
   recent message is NOT from the user, within the last 48 hours.
2. **For each reply, match to a lead.** Compare the sender's email and
   name against `leads.json`. If no match, create a minimal lead row
   and flag `source: "inbound-unmatched"` — the user may want to
   investigate.
3. **Classify** into ONE of:
   - **INTERESTED** — wants a call, asks pricing, says yes
   - **ASKING-QUESTION** — needs info before they can say yes
   - **OBJECTION** — priced, timing, incumbent, authority — name it
   - **NOT-NOW** — explicit defer with a date or trigger
   - **NOT-INTERESTED** — polite no, respectful close
   - **UNSUBSCRIBE** — remove from list, respect + stop
   - **OUT-OF-OFFICE** — auto-reply, requeue for their return date
   - **WRONG-PERSON** — asked for a referral or pointed elsewhere
4. **Extract action items.** Specific asks the user owes back (send
   pricing, book a time, send case study, etc.).
5. **Draft response** (skip for UNSUBSCRIBE / NOT-INTERESTED /
   OUT-OF-OFFICE — no draft needed; just mark handled):
   - Read `config/voice.md`.
   - INTERESTED → propose times (pull from connected calendar via
     `composio search calendar`); match their tone.
   - ASKING-QUESTION → answer directly, short, cite product.
   - OBJECTION → acknowledge, reframe per `config/objections.json`
     (ask user for their standard rebuttals if missing).
   - NOT-NOW → confirm defer + calendar a re-engagement follow-up at
     their stated date (or +90 days if none).
   - WRONG-PERSON → thank, ask for intro to the right person, write
     the intro-ask message.
6. **Write per-reply record** to `replies/{id}/reply.json` (the
   inbound + classification + action items + draft if any) and
   append to `replies.json` index.
7. **Update lead status.** `leads.json` entry: `status` changes based
   on classification (interested → "meeting-ask", not-interested →
   "closed-lost", etc). `lastReplyAt`, `replyCount++`.
8. **Summarize for the user:** "{N} new replies — {K} interested,
   {M} objections, {P} not-now, {Q} unsubscribe/no. Drafts queued
   for the top {K+M+P}." Nothing is sent.

## Never auto-send

Always queue for human approval. Even the "thanks, respecting your no"
message gets reviewed — reply tone can land badly.

## Outputs

- `replies/{id}/reply.json` per handled reply
- Updates `replies.json` index
- Updates lead status in `leads.json`
- Draft at `replies/{id}/draft.md` for non-terminal categories
