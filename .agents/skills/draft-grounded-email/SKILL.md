---
name: draft-grounded-email
description: Use when the user asks to draft outreach / "write an email to {lead}" / "first touch for {lead}" — read the lead dossier (or run `enrich-contact` first if missing), run a fresh signal search, draft a message in the user's voice grounded in at least one specific, cited signal about the prospect (never generic), and store the draft for review.
---

# Draft Grounded Email

Modeled on the "search-grounded email" workflow shape — every draft is
anchored in a real, cited, recent signal about the prospect. No
"I noticed your company is in {industry}" filler.

## When to use

- User: "draft outreach to Jane at Acme" / "first touch for this lead" /
  "write a follow-up to {slug}".
- Batch: "draft outreach to the top 10 GREEN leads."

## Steps

1. **Load the dossier** at `leads/{slug}/lead.json`. If missing or
   stale (>14d), run `enrich-contact` first.
2. **Check fit.** If `fitScore` is missing, run `score-icp-fit`. If
   RED, stop and tell the user why before drafting — don't waste
   the draft budget.
3. **Read voice + product:** `config/voice.md`, `config/product.json`.
   If voice is missing, ask once: "Paste one recent email you've sent
   so I can match your voice — or if you've connected an inbox via
   Composio, just say so and I'll pull recent sent."
4. **Run a fresh signal search.** Even if the dossier has signals,
   query for anything in the last 14 days — news, blog, podcast, job
   posting, funding, exec change, product launch. The freshest,
   most specific signal wins.
5. **Pick the grounding anchor.** One specific signal + one pain
   hypothesis that signal unlocks. Reject vague anchors ("growing
   company", "exciting industry"). Good anchor: "I saw {person} is
   hiring 3 AEs — usually means {pain}."
6. **Draft the message** — 4-5 sentences, in user voice, structure:
   - Line 1: the specific, cited signal ("Saw your Series B — congrats.")
   - Line 2: the inferred pain it opens up
   - Line 3: what we do, one sentence, in their language
   - Line 4: low-friction CTA (15 min? curious thought? worth swapping
     notes?)
   - Match the user's typical sign-off from `config/voice.md`
7. **Add a subject line** — 4-7 words, no clickbait, references the
   signal if it fits.
8. **Write the draft** to `leads/{slug}/outreach-draft.md` with:
   - Subject
   - Body
   - **Grounding** section: the anchor signal + source URL + why-pain
   - **Alt hooks** section: 2 alternative anchors in case the user
     wants to swap
9. **Update `leads.json` entry:** `lastDraftedAt`, `draftCount++`,
   `status: "drafted"`.
10. **Hand to user:** "Drafted. Grounded in {one-line anchor}. Want me
    to tweak tone, pick a different anchor, or queue for send?"
    (NEVER send without approval.)

## What a good anchor looks like (for the model to follow)

- ✓ "Noticed you're hiring a VP of RevOps — that usually means X."
- ✓ "Your CEO's recent post on {topic} suggests Y."
- ✓ "You just moved from {old stack} to {new stack} — Z tends to come up."
- ✗ "I see you work in {industry}." (too generic)
- ✗ "Your company looks exciting." (zero signal)
- ✗ "Hope you're doing well." (padding)

## Outputs

- `leads/{slug}/outreach-draft.md` (subject + body + grounding + alts)
- Updates `leads.json` entry
