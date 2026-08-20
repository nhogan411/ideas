# Restaurant App — Product Brief

> A Letterboxd-style restaurant discovery and logging app for food-obsessed locals and out-of-towners.
> Starting in St. Louis, MO.

---

## The Idea

There's no great answer to "where should we eat tonight?" — not for a solo diner, not for a couple, and especially not for a group. Yelp shows you everything. Google shows you ads. Your friends' recommendations live in scattered text threads you can't search.

This app is the answer to that question. It's a curated, social, opinionated platform for finding great food — not every restaurant, just the best ones. Think Letterboxd for restaurants: you log where you've been, rate it, see what your friends are into, and discover new spots through people whose taste you trust.

---

## Core Philosophy

- **Curation over completeness.** This is not a directory. It's a guide. Only the best local, non-chain restaurants are listed.
- **Social trust over algorithmic trust.** A friend's rating is worth more than a stranger's star. The social graph is the recommendation engine.
- **Friction-free logging.** Logging a visit should take 30 seconds. No essays required.
- **No content moderation burden.** No free-form reviews means no spam, no fake reviews, no competitor sabotage, no moderation overhead.
- **Public by default.** The value of the app grows with visibility. Privacy options may come later, but the default is open.

---

## Target Users

**Primary (launch):**
- St. Louis food enthusiasts who want a trusted, curated guide to the city's best restaurants
- Out-of-towners visiting St. Louis who want local credibility, not TripAdvisor noise

**Secondary (as the app grows):**
- Casual diners, as the catalog expands beyond best-of recipients
- Other cities, once the St. Louis model is proven

---

## What Gets Listed

**At launch:** Only recipients of recognized annual best-of lists from local publications (e.g., STL Today, Riverfront Times, St. Louis Magazine). Restaurants are tagged with the specific honor they received (e.g., "STL Today Best of 2026") — these tags are informational, not clickable lists.

**Long-term:** Expand to all quality local, non-chain restaurants. Second and third locations of local originals are fine. National chains (McDonald's, Olive Garden, etc.) are never listed, ever.

**Geographic scope:** The St. Louis metro area — not just city limits. Clayton, Kirkwood, Maplewood, Ladue, and other suburbs are in scope. *Exact bounding box TBD.*

---

## Restaurant Profiles

Each restaurant listing includes:

| Field | Notes |
|---|---|
| Name | |
| Cuisine type | e.g., Italian, BBQ, Japanese |
| Price tier | $ / $$ / $$$ / $$$$ |
| Neighborhood | e.g., The Hill, Cherokee Street, Clayton |
| Dietary flags | Vegan options, Gluten-free options |
| Alcohol | Yes / No / Beer & Wine only |
| Patio | Yes / No / Seasonal |
| Hours | |
| Description | Editorial copy; admin-written at launch |
| Hero photo | Sourced at launch; restaurant-provided when listing is claimed |
| Menu | None at launch; link-out (medium-term); full in-app menu (long-term) |
| Best-of badges | e.g., "STL Today Best of 2026" |
| Aggregate ratings | Calculated from user logs across all rating dimensions |
| Tag frequency | e.g., "52% of visitors tagged this as a Date Night spot" |

### The Description Problem
Restaurant descriptions are the primary differentiator between Listing A and Listing B — without them, the profile is just data. At launch, descriptions will be written and maintained by the admin. This is a real content production burden and needs to be sized honestly before launch. *Word count and style guide TBD.*

When restaurants eventually claim their listing, they may edit or replace the admin-written description.

### The Photo Problem
Photos are critical for conversion — users decide with their eyes. At launch, a single hero photo will be sourced manually (restaurant website, their social media, or Google Places, with reasonable fair use judgment). Long-term, restaurants upload their own photos when they claim their listing. User-submitted photos are explicitly out of scope — filtering quality and managing storage at that scale is a separate product problem.

---

## User Logs

When a user visits a restaurant, they log it. A log consists of:

**Ratings (overall required; sub-dimensions optional):**
- Overall ★★★★★
- Vibes ★★★★★
- Service ★★★★★
- Food Quality ★★★★★
- Pricing ★★★★★

*Note: 5-star scale. Whether half-stars are supported is TBD. Whether sub-dimension ratings are required or optional affects both logging friction and data quality — this tradeoff should be resolved before building.*

**Pre-built tags (select all that apply):**

Tags are globally normalized (one canonical tag, not user-defined). The full list is TBD and should be developed deliberately. Examples:

| Positive | Negative |
|---|---|
| Date Night | Overrated |
| Special Occasion | Noisy |
| Good Cocktails | Overpriced |
| Great for Groups | Bad Service |
| Hidden Gem | Inconsistent |
| Good Happy Hour | Long Wait |
| Good for a Quick Meal | |

The tag list is admin-controlled. Users cannot create new tags. Requests to add tags will be evaluated against a written policy (TBD) to prevent tag sprawl.

**"Went With" tagging:**
Users can tag other app users who joined them on a visit, Strava-style. *Whether the tagged user must accept or reject the tag is TBD — the answer has privacy and UX implications.*

**Visit date:**
User specifies when they visited. Historical logging is supported — users can log visits from years ago to populate their history. *Whether there is a minimum date cutoff is TBD.*

**Duplicate visits:**
Users can log the same restaurant multiple times. Each log is a separate entry. *Whether ratings aggregate across all visits or use the most recent is TBD.*

---

## Core Features

### 1. Find a Restaurant Near Me (Map View)

A map centered on the user's location showing all listed restaurants nearby. Comprehensive filtering:

- Cuisine type
- Price tier
- Neighborhood
- Dietary flags (vegan, gluten-free)
- Alcohol
- Patio
- Best-of badge
- Rating (overall or sub-dimension, minimum threshold)
- Tags applied by the community
- "On my Want-to-Go list"
- "On my Want-to-Go list AND [friend]'s Want-to-Go list"
- "I've never been"
- "I've been and rated ≥ X stars"
- Open now / open this weekend *(requires reliable hours data)*

### 2. Want-to-Go List

Every user has a personal WTG list. Any restaurant can be added from:
- The restaurant profile page
- The map view
- The activity feed

WTG status is a first-class filter across the entire app. Friend WTG overlap is surfaced as a discovery signal ("You and 2 friends want to go here").

### 3. Help Me Decide

The solution to decision fatigue. Two modes, one entry point:

**Mode A: Solo / Couple Randomizer**
User answers qualifying questions (party size, cuisine preferences, price range, neighborhood, patio, dietary needs, etc.) and the app returns a random restaurant that satisfies all criteria. If nothing matches: show a clear "no results" state plus near-misses with a plain-language explanation of why each didn't qualify. The app never forces a "best option" when criteria aren't met.

**Mode B: Group Vote**
The organizer selects up to 5 candidate restaurants and shares a link. Anyone with the link can vote. *Whether non-app-users can vote via the shared link is a key UX and growth decision — it lowers the barrier significantly but adds complexity.* Votes are tallied and the winner is surfaced. Ties are broken randomly.

*Two distinct interaction modes under one "Help Me Decide" entry point, not two separate navigation items.*

### 4. Activity Feed

The social heartbeat of the app. At launch:

- Shows activity from users in the area
- Prioritizes activity on restaurants that are on your WTG list
- Shifts toward friend-activity-first as the social graph matures

**What generates a feed event:** Logging a visit and rating a restaurant.

**What a feed card shows:** User, restaurant, all rating dimensions, tags applied.

**Feed is read-only at launch.** Reactions are a v2 feature. Tapping a feed card opens the restaurant profile, where you can add to WTG or log your own visit.

**Feed sort:** Recency at launch. Algorithmic weighting is a future iteration.

### 5. Search

- By restaurant name
- By cuisine type
- By neighborhood
- *By tag — natural extension, TBD whether v1 or v2*

### 6. Social Graph

- Find users by name or username
- Shareable profile link (e.g., `app.com/u/nickhogan`)
- "People you may know" — friends-of-friends suggestions
- No contact sync (deliberate — invasive and not aligned with the product's tone)

### 7. User Profiles

What lives on a profile:
- Logged restaurants with ratings and tags
- Want-to-Go list
- "Went with" history
- Aggregate stats across rating dimensions
- *"Taste Fingerprint" summary (e.g., "tends to rate vibes highly, tough on service") — compelling v2 feature*

---

## Restaurant Claiming (Long-Term)

Restaurants can claim their listing (similar to Google Business). When claimed:
- Restaurant edits their description and uploads photos
- Restaurant sees analytics: WTG count, log count, aggregate ratings
- Potential future: manage Yelp / Google Business listing from within the app via third-party APIs

No revenue model at this stage. This is a quality and trust feature.

---

## Phased Feature Roadmap

| Phase | Scope |
|---|---|
| **Launch** | Curated STL restaurants, restaurant profiles, logging with ratings + tags, WTG list, map + filtering, Help Me Decide (solo randomizer), activity feed (area-wide), search, social graph basics |
| **V2** | Group vote mode, friend-personalized feed, "went with" tagging, neighborhood explorer, taste fingerprint |
| **V3** | Menu link-outs, restaurant claiming + analytics, reactions on feed, notification layer |
| **V4** | Full in-app menus, reservation integration (Resy / OpenTable API), multi-city expansion |
| **Long-term** | Expand beyond best-of to all quality local non-chain restaurants |

---

## Known Challenges & Open Problems

These are real problems without clean solutions today. Documented here so they aren't forgotten.

### Content & Data

**Restaurant descriptions:** Who writes them, to what standard, and how are they maintained? At launch this is an admin burden. Needs a style guide and a realistic estimate of time-per-description before committing to a launch catalog size.

**Images:** User-submitted photos are out of scope. Restaurant-provided photos require claimed listings. Launch photos require manual sourcing and carry implicit fair use considerations. Long-term, a photo upload pipeline and storage strategy is needed.

**Menus:** In-app menus are high-value but high-maintenance. The plan: no menus at launch → link-out to external sources (medium-term) → full in-app menus (long-term, via restaurant claiming). The link-out approach depends on restaurants having accurate external pages, which isn't guaranteed.

**Closed restaurants:** Restaurants close. What happens to a closed listing, and what happens to logs associated with it? A soft-delete with a "permanently closed" flag is the likely answer, but needs to be defined.

**Data freshness:** Hours, ownership, and menus change constantly. Without a data sync strategy (Google Places API, manual maintenance, or restaurant-managed), data will go stale. This is an ongoing ops cost that scales with catalog size.

### Reviews & Moderation

**No written reviews (for now):** The structured-tags-only approach is deliberate — it eliminates content moderation, fake reviews, and competitor abuse. The tradeoff is reduced expressiveness. This will face user pressure. The position: reviews may come eventually, but only when there's infrastructure to do it right.

**Tag abuse:** Even without written reviews, tags can be weaponized (e.g., competitors mass-tagging rivals as "Overpriced"). The finite, admin-controlled tag list reduces the surface area, but a flagging mechanism or minimum-account-age requirement may eventually be needed.

**"Went with" tag consent:** Allowing a user to tag you on a visit without consent is a potential harassment vector. Requiring acceptance adds friction. This tradeoff is unresolved.

### Social & Identity

**Authentication:** Not yet decided. Options: Apple Sign-In, Google Sign-In, email/password, or a third-party auth provider (Auth0, Clerk). Important: if any social/OAuth login is offered on iOS, Apple Sign-In is required by App Store policy. This decision affects backend architecture from day one.

**Username vs. real name:** Not yet decided. Letterboxd uses both a display name and a @username handle. The identity model affects tagging, search, and profile shareability.

**Public by default:** Everything is public. Users should understand this clearly at signup. A future privacy layer is anticipated but not scoped.

### Product Design

**Cold start / empty state:** A new user with no friends and no logs sees an empty profile and a generic feed. The area-wide feed provides some immediate value, but onboarding needs to solve for day-one utility without a social graph.

**Aggregate rating threshold:** Should aggregate ratings be suppressed until a restaurant has a minimum number of logs (e.g., 3+)? Without a floor, a single log determines the displayed rating, which is misleading.

**Group vote with non-users:** If non-app-users can vote in a group session via a shared link, this is a meaningful acquisition mechanic. If not, the feature's utility is gated behind everyone already having the app. A significant product decision.

---

## Open Technical Decisions

These are not decisions to make now, but they must be made before writing a line of code.

| Decision | Options | Notes |
|---|---|---|
| **Mobile framework** | React Native, Flutter, Native Swift/Kotlin | Solo builder, cross-platform → React Native or Flutter are practical; Native means building twice |
| **Backend** | Node/Express, Rails, Django, Supabase, Firebase | Supabase or Firebase significantly reduce backend overhead for a solo builder |
| **Authentication** | Apple Sign-In, Google, email/password, Clerk, Auth0 | Apple Sign-In is required on iOS if any OAuth is offered |
| **Mapping** | Google Maps SDK, Mapbox, Apple Maps | Cost at scale vs. features vs. platform fit |
| **Restaurant data source** | Manual entry, Google Places API, Yelp Fusion, hybrid | Google Places API can provide hours, address, phone, and photos as a sync layer — reduces manual maintenance significantly |
| **Image storage** | AWS S3, Cloudflare R2, Supabase Storage | Needed even for admin-uploaded hero photos |
| **Analytics / instrumentation** | PostHog, Mixpanel, Amplitude | Critical for a solo builder iterating without a user research budget |
| **Offline / caching** | Full offline, cache-on-load, online-only | Map + "where should we eat" use case creates expectation of resilience on spotty cell |

---

## What This App Is Not

To prevent scope creep, these are explicitly out of scope — permanently, or until a deliberate decision is made otherwise:

- National or regional chain restaurants. Ever.
- User-submitted photos.
- Written reviews (until content moderation infrastructure exists and the tradeoffs are consciously accepted).
- Real-time check-ins.
- Contact list sync.
- Push notifications (v1).
- Privacy settings (v1).
- Reservation booking (long-term only).
- Algorithmic feed ranking (v1 — recency only).
- Heatmaps, trend analytics, or influencer tiers.

---

*Last updated: August 2026*
*Status: Pre-development concept brief*
