# Civic Pulse STL — Project Brief

> **Status: WIP.** This is a brainstorming-stage concept, not an approved project plan. Nothing described here is scoped, committed, or scheduled. Scope, naming, geography, and feature set are all subject to change as we validate assumptions. This document exists to capture current thinking in one place, not to lock it in.

## The Problem

Local government in St. Louis (and most of its metro area) makes decisions constantly — zoning changes, liquor licenses, traffic calming, redevelopment deals, school board actions — that directly affect residents' neighborhoods and daily lives. Almost none of it reaches residents in a form they can actually use:

- It's scattered across dozens of agency websites and agenda systems, each with its own format.
- It's written in legal/bureaucratic language that's hard to parse quickly.
- It's not personalized — a resident has no easy way to see only what's relevant to where they live and work.
- Communication is one-directional and opt-in-hostile — you have to already know to go looking for it.
- There's effectively no structured way for residents' reactions to reach officials in an aggregated, useful form.

Existing government tools (e.g., Legistar/Granicus "subscribe to agendas" features) solve a narrow slice of this — they let you subscribe to raw agendas from a single body. They don't aggregate across departments, don't personalize by neighborhood, don't translate legal text into plain language, and don't close the loop back to officials.

## The Idea

An app (web-first, mobile-responsive, native app + push notifications later) that:
1. Ingests civic/government data across multiple departments, boards, and (eventually) multiple municipalities.
2. Tags and summarizes it in plain language.
3. Personalizes a feed to each resident based on topics they care about and the neighborhoods/wards they live and work in — without requiring a precise home address.
4. Delivers that feed via in-app browsing and/or a weekly email digest, generated from the same underlying data model (not two separate systems).
5. (V2+) Lets residents give structured, attributable feedback on specific items, rolled up into something useful for officials without losing nuance or becoming a noise-generating platform.

## Current Scope Thinking (v1)

- **Geography:** City of St. Louis only. Not St. Louis County, not surrounding counties — those are explicitly future expansion, not v1.
- **Users:** Residents only. Government staff/official access is an explicit future phase, not v1.
- **Monetization:** None in v1. No payment gateways, no subscriptions. Long-term sustainability is expected to come from eventual institutional contracts (city/aldermen paying for a staff-facing dashboard or data service), not from charging residents. This is a deliberate choice to maximize adoption and trust.
- **Delivery:** Build the core web app/preference model first; treat the weekly email digest as one output of that model, not a separate product. Native app + push notifications are a planned future delivery channel once the core model is proven.
- **Feedback loop (citizen → government):** Deliberately deferred to v2. V1 is one-directional (government → resident) by design, to get the hard parts (ingestion, personalization, trust) right before adding the harder part (structured feedback at scale without becoming noise).

## Explicit Non-Goals (for now)

- Not a crime-mapping / incident-level public safety app. Police-related content should stay framed as policy/oversight/budget, not real-time crime alerts — crossing that line changes the product's trust profile entirely.
- Not a general-purpose neighborhood social network (NextDoor/Facebook Groups). No anonymous posting, no engagement-optimized feed, no infinite scroll of unstructured opinion. See `moderation-and-signal-notes.md` (or the relevant section of the design discussion) for the reasoning — noise reduction is treated as a first-class design constraint, not an afterthought.
- Not trying to monetize residents, directly or via ads, in any currently-discussed version.

## Data Sources We Might Want to Consume

This list reflects research done during brainstorming — it is **illustrative and partial**, not a committed ingestion roadmap. Priority order and final inclusion are still open.

### St. Louis City (v1 priority)
- Board of Aldermen agendas/board bills/ordinances — **CivicClerk** portal, plus legacy PDF archive
- Excise Division liquor license applications/hearings — already Ward/Police District/Neighborhood-tagged in hearing text
- Planning Commission, Board of Adjustment, Preservation Board — zoning/land-use decisions
- Board of Public Service — weekly ordinance/public-works/traffic-calming votes
- Board of Police Commissioners / Civilian Oversight Board — policy/oversight framing only (see non-goals above); note COB status is currently in legal/political limbo as of this writing
- Citizens' Service Bureau (311 equivalent) — has an **Open311 API + CSV** feed already
- SLDC-family redevelopment boards (TIF Commission, LCRA, PIEA, EEZ Commission) and the Land Reutilization Authority (city land bank) — LRA has an existing open data feed
- SLPS (school board)
- City's broader Open Data Portal generally — worth a dedicated inventory pass, several datasets already structured (TIF districts, LRA inventory)

### St. Louis County (future expansion)
- County Council — **CivicWeb** portal (same vendor family as CivicClerk)
- County Board of Police Commissioners
- County Planning Commission (governs unincorporated areas directly — relevant for South County: Affton, Lemay, Oakville, Mehlville/Concord, which have no city council of their own)
- Independent special districts serving unincorporated areas: Mehlville Fire Protection District, Affton Fire Protection District, Mehlville R-IX School District, Lindbergh R-VIII School District — each self-hosted, separate from county government entirely

### Comparable Municipalities (researched as expansion examples, not a fixed list)
- **Clayton, MO** — custom "MeetingsManager" CMS, fairly structured/scrapable
- **Richmond Heights, MO** — Revize CMS, more fragmented; school district on a separate BoardDocs/Simbli platform
- **Chesterfield, MO** — CivicClerk (+ CivicGov for licensing); split across two school districts (Rockwood, Parkway), both mid-migration to Diligent products
- **St. Charles City & County, MO** — CivicPlus/CivicEngage AgendaCenter (a third distinct platform family); TIF tracked more reliably via the **Missouri State Auditor's statewide TIF database** than local portals

### Broader Metro (reference only — not scoped)
St. Louis County (90+ municipalities, including Florissant, University City, Ferguson, Kirkwood, Webster Groves, and many smaller North County municipalities with notable municipal-court accountability history), St. Charles County (O'Fallon, St. Peters, Wentzville, and others), Jefferson County, Franklin County, and the Illinois side (Belleville, East St. Louis, Collinsville, Edwardsville, Granite City, Alton, Fairview Heights) — the Illinois side uses an entirely different state election/court/tax framework and should be treated as its own expansion phase rather than "just another county."

### Platform Landscape Note (relevant to future ingestion architecture)
At least five distinct, non-interoperable agenda/meeting platform families have been identified so far across a small sample of jurisdictions: CivicClerk, CivicWeb, CivicPlus/CivicEngage AgendaCenter, Revize, and assorted custom CMS builds — plus a separate layer of school-board-specific platforms (BoardDocs, BoardBook, Diligent Community/Diligent One), several of which are mid-migration between each other. This suggests an adapter-per-platform ingestion architecture will scale better than a single generic scraper, and that CivicClerk/CivicWeb (shared vendor family) plus CivicPlus (widely used across Missouri generally) are likely the highest-leverage adapters to build first.

## Related Documents
- `data-models.md` — draft entity/data model design and ERD for the application (also WIP).

## Open Questions Still Outstanding
- Go-to-market / initial distribution strategy — explicitly deferred until there's a real product to show.
- Exact sequencing of which data sources to ingest first beyond "liquor licenses + BOA agendas as the narrowest, most structured, most demo-able starting point."
- Legal/ToS review of scraping city/county sites (likely fine for public records, but not yet diligenced).
- Whether/how to pursue city or institutional partnership before vs. after an initial neighborhood-level organic launch.
