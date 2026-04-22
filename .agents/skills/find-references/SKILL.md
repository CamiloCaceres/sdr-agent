---
name: find-references
description: Use when the user asks for "warm intros" / "who do I know at {company}" / "references for {prospect}" — search the user's network via any Composio-connected profile provider and connected CRM (first-degree connections, past colleagues, existing customers in the same space), rank the best warm paths, and draft the intro-ask messages.
---

# Find References

Modeled on the "reference finder agent" shape — pulls from the user's
own network surfaces to find warm paths instead of cold-starting.

## When to use

- User says: "warm intro to Jane at Acme?" / "who do I know at Acme?" /
  "find references for this prospect" / "any customers in fintech I can
  reference?"
- Called inline by `draft-grounded-email` when a lead is GREEN and
  there's no prior contact — we check for warm paths before cold
  outreach.

## Steps

1. **Resolve the target** — person + company. Load
   `leads/{slug}/lead.json` if it exists.
2. **Discover network sources.** Run `composio search` for:
   - connected profile providers (to read 1st-degree connections)
   - connected CRM (to read existing customers + contacts)
   - connected email/calendar (to detect prior correspondence)
3. **Search first-degree connections** via the profile provider for
   anyone who currently works at the target company OR worked there in
   the last 3 years. Capture name, current role, relationship signal
   (years connected, mutual interactions if visible).
4. **Search the CRM** for:
   - existing customers at the target company (for inbound warm path)
   - your own closed-won accounts in the same industry / size (for
     reference customers to name-drop)
5. **Search inbox + calendar** for prior correspondence between the
   user and anyone at the target company — dormant relationships are
   the highest-leverage warm paths.
6. **Rank candidates** by warmth score:
   - **HOT** — recent correspondence, or active 1st-degree connection
     currently at target.
   - **WARM** — 1st-degree connection who worked there recently, or
     existing customer employee.
   - **COOL** — dormant dormant (>2y) correspondence or weak connection.
   Discard anything cooler.
7. **For each top-3 warm path,** draft a short intro-ask message in the
   user's voice (read `config/voice.md`). Format: "Quick one — any
   chance you still talk to {contact} at {target}? Trying to get to
   them for {one-line reason}. Happy to send a forwardable blurb."
8. **Also identify reference customers** — up to 3 existing customers in
   the prospect's same vertical/size that the user could name-drop
   (with permission). List as "Reference candidates" separately.
9. **Write** to `leads/{slug}/warm-paths.md` with the ranked list, draft
   intro-asks, and reference candidates.
10. **Summarize to the user:** "3 warm paths (1 HOT, 2 WARM) + 2 named
    references. Drafts at {path}. Want me to send the top intro-ask?"
    (NEVER send without approval.)

## Outputs

- `leads/{slug}/warm-paths.md` (ranked paths + draft intro-asks +
  reference candidates)
- Updates `leads.json` entry: `warmPathsFoundAt`, `topWarmPath` (name).
