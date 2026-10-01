# Moderation & Signal Notes

> **Status: WIP.** Captures reasoning from brainstorming about why noise reduction and anti-anonymity are treated as first-class design constraints, not bolt-on moderation policy. Referenced from `project-brief.md`.

## A Note on "More Conversation" and What That Requires of Us

The goal is to increase the quantity and quality of communication between residents and government — but *quantity* is not the actual goal, it's a side effect we have to manage. If we're not deliberate, we don't get more informed citizens — we get NextDoor: a feed where opinion volume crowds out signal, pseudonymous accounts say things they'd never say with their name attached, and the loudest/angriest voices define the "neighborhood consensus" because everyone else opted out of the noise. That is the explicit failure mode we are designing against.

### 1. No functional anonymity

Every account is tied to a verified identity (not necessarily publicly displayed, but known to us and attributable if needed) and a verified neighborhood/ward. Verification can be lightweight (email + address-adjacent neighborhood confirmation, not notarized ID), but there is no "guest commenting" and no disposable-account path. Anonymity is what turns a civic feedback tool into a shouting match; real-name-adjacent accountability is what keeps people closer to how they'd behave at an actual public hearing. This is a harder constraint than typical moderation — it's an architectural decision, not a policy you can bolt on later.

**Open problem: lightweight verification alone doesn't stop motivated bad actors** (e.g., a business owner rallying 5 employees to each make an account with a nearby address). Email + address-adjacent confirmation stops *lazy* ballot-stuffing, not *coordinated* ballot-stuffing. This isn't solved yet — see "Verification Tiers" below for the current thinking on a layered approach rather than one binary gate.

#### Verification Tiers (layered, not binary)

Full prevention of fake/duplicate accounts isn't realistically achievable without unacceptable friction (mailed ID checks, notarization) that would kill adoption for a civic tool that's supposed to be *low*-friction. Instead of one gate, use a dial between friction and trust:

- **Base tier — email + lightweight address-adjacent neighborhood claim.** Low friction, matches the "no notarized ID" instinct. Unlocks structured input (checkboxes, yes/no) immediately.
- **Server-side velocity/fraud detection (always on, invisible to users).** Cluster detection — many new accounts from the same IP/device/signup window all responding to the same proposal — flags for human-in-the-loop review (see §4) rather than blocking at signup. This is the actual backstop against coordinated brigading, not the base-tier verification step.
- **Optional "verified resident" tier — mailed postcard code to a claimed address** (same mechanic NextDoor actually uses). Real friction and real cost (postage, delay), opt-in, not required to participate at the base tier. Unlocks a higher-trust-weighted view of a resident's input for officials (see "Tenure & Trust Gating" below) — doesn't gate *participation*, gates *how much an official's view weights that input*.
- **Phone verification (SMS)** sits in between — cheap and fast, but phone numbers are now a commodity (VOIP/burner), so treat it as a weak signal that's only useful *combined* with another signal, not standalone.

Deliberately not using government ID verification (Persona/Stripe Identity/etc.) despite it being the strongest signal — the privacy/trust liability of a civic/government-adjacent app asking residents for a driver's license is too high relative to what it buys, and conflicts with the "not notarized ID" design instinct already established.

### 2. Structured input is the default; free text is the exception, not the norm

Noise mostly isn't bad actors — it's good-faith people rambling. Every feedback surface should lead with checkboxes/scales/structured choices, and treat open text as a narrow, optional elaboration on a structured answer, never the primary input. If someone wants to write a paragraph, that's a signal they have something specific to say — worth capturing precisely (see note on "traffic" below) — but the UI should never invite open-ended venting as the first move.

**Concrete shape, per proposal:**
- A top-level binary **"Do you support this?"** (yes/no).
- A group of **checkboxes** for "What matters to you about this?" (e.g., Traffic, Noise, Parking, Property Values...). Checking a box reveals a **single-line text input** (not a textarea) scoped to that specific concern — capped length, intentionally not a place to write a paragraph. The text input is optional even when the box is checked.
- After concerns are checked (and optionally elaborated on), the user has an **optional** opportunity to rank their checked concerns from highest to lowest priority, using up/down move controls per item (not drag-and-drop — see rationale below). Ranking is all-or-nothing at the concern-set level, not per-item: if the user never engages with ranking, every checked concern is simply **unranked**; the moment they move any item, every checked concern receives a defined position (e.g., swapping positions 1 and 3 leaves 2 and 4 where they were, but all four are now ranked). There is no partial state where some concerns are ranked and others aren't, and no ties.
  - **Why ranking, not a severity scale (e.g., "minor/moderate/major" per item):** ranking is comparative, not absolute, so it doesn't invite the false-statistical-precision trap described in note #5 below (a severity scale produces numbers people will be tempted to average across respondents — "avg severity: 2.3" — which is exactly the fabricated precision we're trying to avoid). Rank order instead produces an honestly-phrasable frequency stat ("Traffic was ranked the #1 concern by 60% of respondents who selected it").
  - **Why up/down controls, not drag-and-drop:** drag-to-reorder is a known mobile-UX/accessibility pain point (poor touch affordance, bad screen-reader support). Building up/down as the *only* mechanism (not just a fallback) gives one implementation that works the same on desktop and mobile and is natively screen-reader-describable ("move Traffic up one position"), rather than building drag-and-drop as primary and bolting on an accessible fallback later.
  - **Why ranking is fully optional, same as the text elaboration:** the whole input flow should be a friction ladder, not a wall — check → (optional) elaborate → (optional) rank. Nobody is blocked from submitting at any level. Forcing a resident to comment on or rank every concern they checked risks pushing them toward the shallow "just check boxes and leave" behavior we're trying to avoid, by making deeper engagement feel costly rather than invited.
  - **Data handling note:** since initial checkbox-click order isn't a deliberate signal, the backend must treat "unranked" as "no priority signal available," never silently default to storing click order as rank 1..N. Doing so would manufacture the same false precision this whole design is trying to avoid — just one level up, in the data model instead of the UI.

### 3. Reduce volume deliberately, not just by "letting the market decide"

NextDoor-style platforms converge on infinite scroll + reactions + algorithmic engagement, which literally rewards noise. We should do close to the opposite: rate-limit how often someone can comment per item, cap total visible comments per item (or require sorting/summarization above a threshold), and never optimize for time-on-app or engagement metrics. If an item has 40 pieces of feedback, the default view should be the *rolled-up summary*, with raw comments one click deeper — not a scrolling wall of takes. Engagement is not success here; a resident reading one clear summary and feeling informed is success.

### 4. Moderation is a precondition, not a bolt-on

Before any open-text feature ships: a clear community standard, rate-limiting/anti-brigading detection, and a human-in-the-loop review path — especially for anything that could target a protected class, a specific individual/business owner, or incite harassment (liquor license fights over who owns the bar are a predictable flashpoint). One bad incident can end this product's credibility; the bar for "ready to ship" on any feedback feature is high.

### 5. Preserve what people actually meant when rolling up feedback

"I'm concerned about traffic" could mean commute volume, pedestrian safety, speeding, or parking — and collapsing all of that into one aggregate stat makes officials *less* informed while looking more data-driven, which is worse than not collecting it. Rollups should: keep structured sub-tags where possible, use AI clustering on free text to catch what doesn't fit predefined tags (and surface representative quotes, not just a number), flag ambiguous/unspecified responses honestly rather than silently bucketing them, and avoid false statistical precision — "of respondents in this neighborhood," never "X% of the neighborhood."

### 6. AI summary layer replaces raw comment surfacing by default

Rollups shouldn't present a scrolling wall of raw comments as the primary view for *anyone*, resident or official. Instead, an AI layer summarizes and groups free-text elaboration into the rollup — the default experience is "here's the AI summary/recap of what people said about Traffic," not a feed of individual comments. This is the implementation of note #3's "rolled-up summary, raw one click deeper" principle, made concrete: summary is the default surface, raw text is opt-in-deeper, not the other way around.

- **Open question — who can drill into raw text, and under what role?** Current thinking leans toward: officials/moderators can drill into the raw corpus (useful for audit/dispute resolution and building trust in the summaries), while the general public only ever sees the AI rollup plus AI-selected representative quotes (which note #5 already calls for). Not yet settled — flag for review once we have a real moderator/official-facing view to test this against.
- **Open question — resummarization cadence.** Does the rollup resummarize live as each new response comes in, on a batch cadence (e.g., nightly), or on-demand when viewed? Matters for LLM cost and for whether an early respondent's comment visibly "mattered" before being folded into a larger group. Not yet decided.

### 7. Let the user review their own AI summary before it's submitted (optional)

When a user has written free-text elaboration, give them an (optional — only triggered when there's meaningful free text to summarize, not for pure checkbox-only submissions) opportunity to see the AI's summary of what they wrote *before* it's finalized into the rollup, and to revise if it drifted from what they meant. This directly serves the "structured by default, no raw venting surfaced" goal: it gives the user a forcing function to self-correct the public-facing record of their own input, rather than discovering after the fact that an AI summary misrepresented them.

- Should support both **regenerate** and **direct edit** of the summary text, not just an approve/reject binary — an approve-only flow risks a user rubber-stamping a summary that's subtly off from what they meant (small trust problem: the record shows "approved," but the summary reads as a stance they didn't quite hold).
- As a cheap, non-AI-dependent version of this same idea: when the user also used the (optional) concern-ranking feature from note #2, show them a plain recap of their ranked order ("You ranked: 1. Traffic 2. Parking 3. Noise") as part of the same pre-submit review — reinforces that structured input was captured correctly, no AI needed for this part.

### 8. Proposal comments (maybe, big maybe) stay AI-gated and one-directional

Beyond the per-item "what matters to you" concern tags, there's a (not yet committed) idea to allow free-form comments *on a proposal itself* rather than only on a predefined concern tag. If this ships at all, it should not ship before the AI summarization layer (note #6) exists, and it should be strictly one-directional — residents submit a perspective, the AI rolls it up — with **no reply-to-comment / threading**. The moment threaded replies exist, this becomes a discussion platform between residents rather than a channel from residents to officials, which is the exact NextDoor failure mode this whole document is designed against.

### 9. Tenure & trust gating (inspired by, but not settled on, Stack Overflow's reputation model) — UNRESOLVED

Stack Overflow slowly unlocks account capabilities as accounts age and accumulate reputation (can't do X or Y until the account is old enough or has participated enough). Something in this shape is appealing here — some combination of tenure, verification level, and clean behavior probably *should* earn an account more freedom/flexibility over time — but we talked through several framings (a pure time-based gate; a separate volume-based "engagement unlock" gate; folding everything into one eligibility evaluation shared with the trust-weight gate) and didn't converge. Capturing the state of the discussion rather than a decision:

- **Firmly decided:** whatever this becomes must not be a visible score, leaderboard, or badge system (gamification is explicitly unwanted), and it must not be driven by raw participation *volume* (comment/post count) — that would recreate the exact engagement-chasing dynamic note #3 already rejects, and SO's reputation system is built on volume in a way we don't want to copy.
- **Firmly decided:** however an account's *input* gets weighted in official-facing rollups (call it the trust-weight question) must stay independent of any engagement/feature-unlock mechanism — letting engagement buy more influence with officials would create second-class citizens among residents, which was the strongest argument raised against folding these together.
- **Not decided:** what, specifically, is dangerous/risky enough in this product that it's worth gating behind tenure/trust at all, versus just being open to any verified account from day one. (Free-text elaboration? Proposal comments, if those ever ship? Flagging others' content for review? Something else?) We don't have a clear enough picture yet of what bad behavior looks like in this product to know what's worth gating.
- **Not decided:** whether the gate is purely time-based, or an evaluation combining tenure + verification level + moderation history + other signals not yet identified.
- **Not decided:** whether this is one gate or several with different thresholds for different capabilities.

Revisit this once there's a clearer sense of what actually needs protecting — this note is a placeholder for "there's real design thinking to do here," not a spec.

### Net Goal

Every piece of information a resident or official sees through this tool should be worth their attention. If we can't guarantee that, we add a gate before we add a feature.
