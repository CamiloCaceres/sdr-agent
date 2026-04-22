---
name: score-icp-fit
description: Use when the user asks "is this a fit" / "score this lead" / "qualify this account", or after `enrich-contact` completes and the lead lacks a fit score — read the enriched dossier, compare against `config/icp.json` across industry / size / role / stage / signal criteria, return a GREEN / YELLOW / RED verdict with per-criterion reasoning, and write it back to the lead record.
---

# Score ICP Fit

Modeled on the "business fit analyzer" workflow shape — structured
go / no-go against explicit criteria, not vibes.

## When to use

- User says: "is Jane at Acme a fit?" / "qualify this lead" / "score
  these leads" (batch).
- Called by `draft-grounded-email` if a lead has no `fitScore` yet — we
  score before we spend effort drafting for a RED.
- Called by a routine nightly to score any un-scored leads.

## Steps

1. **Read `config/icp.json`.** If missing, ask ONCE:
   "I need your ICP before I can score fit. One line: who do you sell
   to? *Paste a description, point me at a connected CRM (I'll infer
   from closed-won), or share your pricing-page URL.*" Write the
   answer, continue.
2. **Load the lead's dossier** from `leads/{slug}/lead.json`. If missing
   or older than 14 days, run `enrich-contact` first.
3. **Score per criterion.** For each ICP criterion (industry, size,
   role seniority, buying stage, trigger signals), mark:
   - **HIT** — dossier clearly matches
   - **MISS** — dossier clearly does not match
   - **UNKNOWN** — data wasn't available (don't penalize)
   Capture the specific dossier field + value that drove each verdict.
4. **Compute the verdict:**
   - **GREEN** — all HARD criteria HIT, majority of SOFT criteria HIT.
   - **YELLOW** — all HARD criteria HIT, minority of SOFT HIT, OR some
     UNKNOWN that could flip it either way.
   - **RED** — any HARD criterion MISS.
   (Hard vs soft is set in `config/icp.json`. Defaults: industry + size
   hard, role + stage + signals soft. User can override.)
5. **Write the scorecard** to `leads/{slug}/fit.md` with the verdict,
   per-criterion table, and a 2-line rationale.
6. **Update `leads.json` entry** — `fitScore: GREEN|YELLOW|RED`,
   `fitReasoning: "..."`, `fitScoredAt`.
7. **Tell the user the verdict + top-line why.** If RED, recommend
   "skip unless you see a specific reason." If YELLOW, name the
   UNKNOWNs and suggest what would flip it. If GREEN, suggest
   `draft-grounded-email` as the next step.

## Batch mode

If the user asks to score "these leads" / a list: iterate, respect a
per-user rate cap (default 20 at once), append rolling results as they
land, and summarize at the end: "10 GREEN · 7 YELLOW · 3 RED."

## Outputs

- `leads/{slug}/fit.md`
- Updates `leads.json` entry: `fitScore`, `fitReasoning`, `fitScoredAt`
