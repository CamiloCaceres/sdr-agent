---
name: build-battlecard
description: Use when a prospect names a specific competitor — "they're looking at {competitor}" / "build a battlecard for Acme vs {competitor}" — research the competitor (positioning, pricing shape, known weaknesses, recent reviews) and the prospect's specific use case, produce a 1-page prospect-specific card with honest comparison, trap-set questions, objection rebuttals, and proof points the user can cite.
---

# Build Battlecard

Modeled on the "sales battle card for a prospect" workflow shape —
NOT a generic comparison sheet. A per-prospect card anchored in what
THAT prospect actually cares about.

## When to use

- User: "they're evaluating us against {competitor}" / "build me a
  battlecard for the Acme deal vs {competitor}" / "how do I beat
  {competitor} for this one".
- Called inline by `draft-grounded-email` or `analyze-discovery-call`
  when a competitor is named in the transcript.

## Steps

1. **Identify the prospect + competitor.** Load
   `leads/{slug}/lead.json` and `calls/{call_id}/notes.json` if a call
   exists — the prospect's specific evaluation criteria and stated
   pains are the anchor.
2. **Read our product + positioning.** `config/product.json` for what
   we claim. `config/differentiators.json` if it exists. If missing,
   ask once: "What's your honest top-3 differentiator vs {competitor}?
   And your biggest weakness? (I'll keep this to inform
   battlecards — paste, or point me at a Notion/Doc URL.)"
3. **Research the competitor.** Run `composio search` for available
   research tools. Pull:
   - Their marketing-page positioning (one-line pitch, top 3 claims)
   - Public pricing shape (tiers, model)
   - Recent reviews in the last 6 months (G2 / Capterra / Reddit /
     forum threads — via any connected web-search tool)
   - Known weaknesses (complaints about {X}, performance gripes,
     missing features)
   Capture sources + dates.
4. **Research the prospect's use case.** From the dossier and call
   notes, summarize in 2 lines: what they actually need this tool to
   do, and which 3 criteria they care most about.
5. **Build the comparison grid** — for THEIR top 3 criteria only (not
   a 30-row feature matrix). For each: us vs them, honest verdict
   (WE-WIN / THEY-WIN / TIE), one-sentence why.
6. **Trap-set questions.** 3 questions the user can ask in the next
   call that will surface the competitor's weaknesses naturally — not
   gotchas, genuine discovery. Each tied to a known competitor pain
   point.
7. **Objection rebuttals.** Anticipate 3 objections the competitor's
   rep would raise about us; draft a 2-sentence rebuttal for each,
   anchored in our differentiators (no false claims).
8. **Proof points to cite.** 2-3 customer stories from our book that
   match this prospect's profile (pull from
   `config/reference-customers.json` if it exists; if not, ask once
   for a Notion/Doc URL listing our top-cited wins).
9. **Write** to `leads/{slug}/battlecard-vs-{competitor-slug}.md`
   with: prospect + competitor header, criteria grid, trap-set
   questions, objection rebuttals, proof points, research-sources
   footer.
10. **Index.** Append to `battlecards.json`: `{ slug, competitor,
    leadSlug, generatedAt, sources: [...] }`.
11. **Hand off to user:** "Battlecard ready — 3-criterion grid,
    3 trap-set questions, 3 rebuttals, 2 proof points. Want me to
    weave it into your follow-up draft?"

## Honesty rule

Never fabricate "they're weak at {X}" without a cited source. If a
claim has no source, mark it "(hypothesis — verify)" so the user
doesn't accidentally repeat it as fact. Fabricated battlecards blow
up in demos.

## Outputs

- `leads/{slug}/battlecard-vs-{competitor-slug}.md`
- Appends to `battlecards.json`
