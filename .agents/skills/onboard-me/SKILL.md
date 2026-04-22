---
name: onboard-me
description: Use when the user explicitly says "onboard me" / "set me up" / "let's get started", or on the first real task when no `config/profile.json` exists — open with a scope + modality preamble naming the three topics (ICP, product + pitch, voice) AND the best way to share each, then run a tight 90-second 3-question interview and write results to `config/`.
---

# Onboard Me

## When to use

First-run setup. Triggered by "onboard me" / "set me up" / "let's get
started", OR about-to-do-real-work and `config/profile.json` is missing.
Only run ONCE unless the user explicitly re-invokes.

## Principles

- **Lead with a scope + modality preamble.** Name the three topics AND
  the easiest way to share each BEFORE the first question. "Give me
  context" in the abstract is too vague.
- **3 questions is the ceiling, not the target.** If you can do 2, do 2.
- **One question at a time after the preamble.** Short follow-ups — the
  preamble did the heavy lifting.
- **Rank modalities:** connected app via Composio > file/URL > paste.
- **Anything skipped** → note "TBD" and ask again just-in-time later.

## Steps

0. **Scope + modality preamble — the FIRST message, then roll into Q1:**

   > "Let's get you set up — 3 quick questions, about 90 seconds.
   > Here's what I need and the easiest way to share each:
   >
   > 1. **Your ICP** — who you sell to (industry, size, role, trigger
   >    signals). *Best: paste a line or two. Or point me at a
   >    connected CRM (Composio) and I'll infer from your top accounts.
   >    Or share your pricing/about page URL — I'll read it.*
   > 2. **Your product + pitch** — what you sell and your 30-second
   >    pitch. *Best: drop a pitch deck, one-pager, or pricing-page URL.
   >    Or paste a short description.*
   > 3. **Your voice** — how you write. *Best: if you've connected an
   >    inbox via Composio, just say so — I'll pull 20-30 of your recent
   >    sent messages. Otherwise paste 2-3 emails you've actually sent.*
   >
   > For any of these you can also drop files or share public URLs.
   > Let's start with #1 — in one line, who's your ICP?"

1. **Capture topic 1 (ICP).** Based on modality chosen: parse paste,
   fetch URL, read file, or run `composio search crm` to discover the
   connected CRM tool slug and pull top closed-won accounts to infer
   patterns. Write `config/icp.json` with `{ industry, size, role,
   triggers, examples, fitCriteria, source, capturedAt }`. Acknowledge
   and roll into Q2: "Got it — now your product + pitch?"
2. **Capture topic 2 (product + pitch).** Same pattern. Write
   `config/product.json` with `{ name, oneLine, pitch30s, painsSolved,
   differentiators, pricing?, source, capturedAt }`. Roll into Q3:
   "Last one — how do you write? Connected inbox, or paste samples?"
3. **Capture topic 3 (voice).** If user took the connected-inbox route:
   run `composio search` for their connected provider's
   list-sent-messages tool; fetch the 20-30 most recent outbound
   messages; extract tone cues (greeting, closing, sentence length,
   formality, quirks); write 3-5 verbatim excerpts plus a tone summary
   to `config/voice.md`. If they pasted samples, write those samples
   verbatim plus a short tone summary.
4. **Write `config/profile.json`** with `{ userName, company, team?,
   onboardedAt, status: "onboarded" | "partial" }`. Use `"partial"`
   if any topic was skipped.
5. **Hand off:** "Ready. Try: `Enrich {person} at {company}` — or drop
   a lead list. I'll ask for anything else just-in-time."

## Outputs

- `config/profile.json`
- `config/icp.json`
- `config/product.json`
- `config/voice.md`
