---
name: enrich-contact
description: Use when the user names a person / company / email / profile URL and asks to enrich, research, or "look up" — pull firmographics, role context, tech stack, and recent signals via any Composio-connected enrichment + profile + news provider, upsert a record into `leads.json` with per-lead detail in `leads/{slug}/`, sync to the connected CRM if one is linked, and optionally post a summary to a connected team-notification channel.
---

# Enrich Contact

A single pipeline that takes a thin signal (a name, an email, a profile
URL) and turns it into a decision-ready dossier pushed to the places
you actually work.

## When to use

User input shapes that trigger me:
- "Enrich Jane Doe at Acme"
- A profile URL or email pasted
- A spreadsheet of leads dropped in — run me once per row
- Called by another skill (`draft-grounded-email`, `score-icp-fit`,
  `build-battlecard`) when the lead's dossier is missing or stale (>14d).

## Steps

1. **Resolve the lead.** Parse name + company from whatever the user
   gave (email domain, profile URL path, paste). Compute
   `slug = kebab-case(firstname-lastname-company)`. Check `leads.json` —
   if slug exists and `enrichedAt` is < 14 days old, load the existing
   dossier, summarize, and stop unless the user asked to refresh.
2. **Upsert the minimal index row** in `leads.json` with `status: "new"`,
   `source`, timestamps.
3. **Discover provider slugs.** Run `composio search` for the three
   categories I need: profile/role enrichment, company firmographics,
   news/signals. Do NOT hardcode tool names. If a category has no
   connection, record "source unavailable: {category}" and keep going.
4. **Fetch role context** — title, tenure, prior companies, team,
   location. Record source URL + `fetchedAt`.
5. **Fetch firmographics** — industry, headcount band, funding (round +
   amount + date), HQ, tech stack if detected. Source each field.
6. **Fetch recent signals (last 60 days)** — news, blog posts, podcast
   appearances, job postings (hiring signals are pain signals),
   executive changes. Keep top 3-5 most relevant.
7. **Generate pain hypotheses.** Based on role + company stage + signals,
   write 2-3 concrete hypotheses for what might be on this person's
   plate. Label each as **hypothesis** — never a fact.
8. **Write the dossier** to `leads/{slug}/lead.json` with the full
   structured payload and `leads/{slug}/dossier.md` as the human-readable
   version.
9. **Update `leads.json` entry** with top-line fields: `title`,
   `company`, `industry`, `headcount`, `enrichedAt`, `sources` array,
   `painHypotheses` count.
10. **CRM sync (if connected).** Run `composio search crm` → if a CRM is
    linked, upsert the contact + company record. Include only
    deterministic fields (name, title, company, email if known) — never
    sync hypotheses or unverified claims. Record `crmSyncedAt` and the
    CRM record URL on the lead.
11. **Team notify (if connected + requested).** If a team-chat
    integration is linked AND the user configured notify-on-enrich in
    `config/notify.json`, post a 3-line summary + link to the dossier
    to the configured channel. Otherwise skip silently.
12. **Tell the user:** "Enriched — 3 sources, 2 pain hypotheses, CRM
    synced. Want me to score ICP fit or draft outreach?"

## Progressive config capture

- If `config/icp.json` is missing, don't block — complete the enrichment
  without it. Mention at the end: "Heads up, I don't know your ICP yet,
  so I can't score fit. One line: who's your ICP?"
- If the user hasn't set `config/notify.json` (channel for post-enrich
  summaries), don't notify and don't ask — opt-in only.

## Outputs

- `leads/{slug}/lead.json` (full structured dossier)
- `leads/{slug}/dossier.md` (human-readable)
- Updates `leads.json` entry
- Optional: CRM record upsert via Composio
- Optional: team-chat notification
