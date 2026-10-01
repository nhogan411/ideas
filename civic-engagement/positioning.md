# Civic Pulse STL — Positioning

> **Status: WIP / Nothing settled.** This is an early application of April Dunford's positioning framework (from *Obviously Awesome*) to this concept. It reflects current thinking as of this brainstorming session only — product name, category framing, and messaging are all placeholders for discussion, not decisions. Expect this to be revisited once we have real users and real ingested data to test it against.

## Why Do This Exercise

Positioning isn't marketing copy — it's a forcing function to answer "compared to what, for whom, and why does that comparison favor us" before building anything. If we can't answer those cleanly, we risk building a technically impressive tool that nobody has a reason to choose over whatever they're already (half-heartedly) doing.

## 1. Competitive Alternatives

If this app didn't exist, residents would be doing what they do today — and it's genuinely fragmented, not one dominant alternative:
- Local subreddit (e.g., r/saintlouis)
- Neighborhood Facebook groups
- NextDoor
- Local newsletters
- Mainstream local news/TV coverage
- The city's own narrow tools (Legistar/govDelivery agenda subscriptions, department email lists)
- Nothing at all — just not engaging until something personally affects them

**The fragmentation itself is the insight.** No single existing channel is good enough on its own, which is the gap worth exploiting — not "beat NextDoor" or "beat the city's tools," but "replace five unreliable partial sources with one trustworthy complete one."

## 2. Unique Attributes

What this product can offer that no single alternative combines:
- Cross-department, cross-board aggregation into one place (vs. each channel covering a narrow slice)
- Personalization by neighborhood + workplace + topic, without requiring a precise home address
- Plain-language translation of legal/bureaucratic text (feasible now in a way it wasn't a few years ago, due to AI-assisted summarization)
- Time-sensitive surfacing — delivered early enough to actually act (attend a hearing, submit feedback) rather than after-the-fact news coverage
- A deliberate "why this matters" framing with an honest relevance vocabulary (direct impact / nearby / indirect / topic match / FYI) — see `notification-design-and-relevance-model.md`
- (v2) Structured, non-anonymous, nuance-preserving feedback rollup — not a comment section

## 3. Value These Attributes Enable

- Time savings — one place instead of five
- Confidence that nothing personally relevant was missed
- Lower friction to actually *act* on what's learned (not just informed, but equipped to participate)
- Higher signal, lower noise than any social-platform alternative
- Honest, calibrated relevance — never manufactured urgency or inflated personal stakes
- (v2) A real, trustworthy channel for officials to hear aggregated, nuance-preserving constituent sentiment

## 4. Target Segment (Best-Fit Early User)

Explicitly **not** an apathy-conversion product. The product's job is not to manufacture civic interest where none exists — it's to make existing interest easier to act on.

**Best-fit user: already civically-interested residents who are time/attention-constrained** — people who would engage more if it weren't so scattered and effortful across five different channels. "Participation-ready, but logistically blocked" is the sharpest description.

This has a direct product-design consequence, stated as an explicit litmus test for every feature under consideration:
> "How does this feature make it easier for a resident to participate in local government?" **or** "How does this make it easier for local government to hear its constituents?"
> If a proposed feature can't answer one of those two questions, it's probably out of scope.

## 5. Market Category

The category we frame this as matters as much as the feature set — it sets user expectations before they ever open the app.

Rejected framings:
- ❌ **"Neighborhood social network"** — invites direct NextDoor comparison, sets wrong expectations (community/conversation volume), attracts noise-seekers exactly the product is designed to repel.
- ❌ **"Government transparency portal"** — sounds dry and civic-tech-y, undersells the personalization and ease-of-use that's actually the differentiator.
- ❌ **"Newsletter"** — undersells the interactive app core and risks boxing the product into the "easy MVP trap" already identified in earlier brainstorming (building a newsletter system instead of an app with a newsletter view).

Working framing:
- ✅ **"Personal civic briefing / civic participation assistant"** — positions it in the productivity/personal-assistant category (think: a personal analyst or a well-curated morning briefing, not a forum). This sets expectations of curation, brevity, and utility — not community size or conversation volume — which aligns with the anti-noise design philosophy throughout this project.

## 6. Relevant Trends Supporting This Positioning

- Rising institutional distrust and appetite for local accountability — with specific relevance in the St. Louis metro given post-2014 civic engagement history (e.g., Ferguson-era municipal accountability reporting).
- AI-enabled summarization making "translate bureaucratic text into plain language, cheaply, at scale" newly feasible — this specific capability wasn't practically buildable even a few years ago.
- Documented, broad fatigue with algorithmic/outrage-optimized social platforms, creating real appetite for a calmer, utility-first alternative — this is a live cultural moment, not a hard sell.

## Draft Positioning Statement

> For **St. Louis residents who already care about their city but can't keep up with where civic information actually lives**, **Civic Pulse STL** (working name) is a **personal civic briefing tool** that **aggregates everything your local government is doing — across every department and board — into one plain-language feed, personalized to your neighborhood and interests, delivered early enough to actually act on it.** Unlike **piecing it together across Reddit, Facebook groups, NextDoor, local news, and scattered city subscribe tools**, Civic Pulse STL **gives you one trustworthy, low-noise source instead of five unreliable ones — and never becomes another feed you have to scroll.**

## Open Questions / Not Yet Settled

- Whether "personal civic briefing" survives contact with real users, or whether it undersells the (eventual) two-way participation angle once v2 feedback features exist.
- Product name is a placeholder ("Civic Pulse STL") — not evaluated against trademark, domain availability, or how it reads next to the "no NextDoor" positioning (does "Pulse" accidentally imply a social-feed product? worth a second look).
- This positioning has not been tested against any real prospective user — it's a first-pass internal exercise, not validated messaging.

## Related Documents
- `project-brief.md` — overall concept and scope
- `data-models.md` — draft schema
- `notification-design-and-relevance-model.md` — the "why this matters" / relevance vocabulary design that underpins attribute #5 above
- `moderation-and-signal-notes.md` — the anti-noise design philosophy referenced in the target segment and category reasoning
