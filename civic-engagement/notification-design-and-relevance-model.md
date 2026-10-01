# Notification Design & Relevance Model

> **Status: WIP.** Captures thinking on how the app presents civic items to users — the "why this matters" framing and relevance vocabulary — and the three-part core philosophy underlying the product. Nothing here is finalized; this will likely require refinement once we see real ingested data and test actual "why this matters" copy against real agenda items.

## Core Philosophy (Three-Part)

1. **Government tells you what is happening / has happened.** The app surfaces civic items at every process stage — scheduled, in progress, completed — faithfully reflecting the source record. This is factual reporting, not editorializing.
2. **The app helps you understand why it matters.** Context is added — scope, who's affected, degree of relevance to *you specifically* — plus (v2) an easy-to-scan rollup of what other residents think. This is interpretation *of fact*, never invented relevance.
3. **You decide whether it matters to you, and what to do about it.** The app's job ends at informing and contextualizing. Acting — attending a hearing, submitting feedback, or simply ignoring it — is entirely the resident's call. The app is an opportunity for action, never a nudge toward a predetermined one.

This maps directly onto the non-goals already established elsewhere (no engagement-optimized feed, no manufactured urgency): the app's success metric is a resident feeling *accurately informed*, not a resident taking any particular action.

## The "Why This Matters" Section

Every surfaced civic item should include a structured "Why this matters" block, built from objective facts first, personalized relevance second. Proposed structure:

1. **What's actually happening** — a faithful, plain-language restatement of the agenda item/ordinance/license application. Extracted and simplified, never embellished. ("The Board of Aldermen will vote on a contract to repave a 1.2-mile stretch of X Road between A and B.")
2. **Who/what this applies to** — the factual scope of the action: specific addresses, a radius, a zoning classification, a ward, a whole department's policy, etc. This is where precision matters most, and where we have to resist the urge to round up relevance. ("This zoning change applies only to commercially-zoned parcels along this corridor; it does not apply to adjacent residential lots.")
3. **Degree of relevance to you** — see Relevance Vocabulary below. Always paired with the reasoning behind the label, not just the label alone. ("Your home is approximately 400 feet from the proposed business address" / "This applies to commercial properties only; your property is zoned residential.")
4. **(v2) What others think so far** — an optional, honest rollup of other users' structured feedback on this item, when enough exists to be meaningful (see `moderation-and-signal-notes.md` for how rollups must avoid false precision and preserve nuance).

### The Core Guardrail

**Never manufacture a reason to care.** If an item is citywide in scope and there's no specific signal it affects a given user's block, say exactly that — don't stretch a tenuous connection to make the notification feel more personally relevant than it is. Honesty about *weak* relevance is just as important as accuracy about strong relevance; a user who catches the app inflating relevance once will stop trusting every "why this matters" block afterward.

## Relevance Vocabulary (draft)

A small, consistent vocabulary for labeling degree of relevance, always shown with the underlying reasoning:

| Label | Meaning | Example reasoning shown to user |
|---|---|---|
| **Direct Impact** | The action directly concerns your property, your block, or something you specifically hold (a permit, a license near you) | "This liquor license application is for the address directly next door to your home." |
| **Nearby** | Within a defined proximity to a neighborhood/ward you're associated with (home or work), but not a direct action on your property | "This is a zoning case ~2 blocks from your home neighborhood." |
| **Indirect** | Affects your broader neighborhood/ward/city but isn't hyper-local — a citywide or department-level policy that touches a topic you follow | "This is a citywide traffic-calming policy change; it doesn't name a specific location near you yet." |
| **You Asked About This** | Matches a topic subscription with no strong geographic signal either way | "You're subscribed to 'Public Transit' — this is a Metro route change that doesn't specifically name your neighborhood." |
| **FYI** | Informational, included for completeness/transparency, weak or no personal relevance signal | "This is a citywide budget item; we have no specific signal it affects your neighborhood directly." |
| **Nothing to Report** | Not a per-item label — used at the digest level to confirm the system is active even when nothing matched this week, so silence reads as "nothing happened" rather than "the app stopped working." | "No new items matched your interests this week." |

This vocabulary is a first draft — it should be pressure-tested against real ingested items (especially ambiguous cases, like a citywide ordinance that happens to reference a specific ward in passing) before being treated as final.

## Schema Implications (flag for `data-models.md`, not yet applied)

The relevance label and "why this matters" text are **per user-per-item**, not per-item — they depend on the specific user's home/work neighborhood relative to the item's scope. This means:

- `CivicItemEnrichment` should continue to hold the *objective* facts (scope, geometry, topics) — this doesn't change.
- `NotificationQueueEntry` likely needs new fields to hold the *personalized* output:
  - `relevance_label` (enum: direct_impact | nearby | indirect | topic_match_only | fyi)
  - `why_this_matters_text` (generated per user, referencing both the objective enrichment data and the user's specific neighborhood/topic match)
- These are computed at notification-generation time (combining `CivicItemEnrichment` + `User` location/topic data), not stored on the item itself, since the same item will have a different relevance label for different users.

This is noted here as a follow-up to fold into `data-models.md` once reviewed, not yet applied to that file.
