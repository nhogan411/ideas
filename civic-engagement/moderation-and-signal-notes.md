# Moderation & Signal Notes

> **Status: WIP.** Captures reasoning from brainstorming about why noise reduction and anti-anonymity are treated as first-class design constraints, not bolt-on moderation policy. Referenced from `project-brief.md`.

## A Note on "More Conversation" and What That Requires of Us

The goal is to increase the quantity and quality of communication between residents and government — but *quantity* is not the actual goal, it's a side effect we have to manage. If we're not deliberate, we don't get more informed citizens — we get NextDoor: a feed where opinion volume crowds out signal, pseudonymous accounts say things they'd never say with their name attached, and the loudest/angriest voices define the "neighborhood consensus" because everyone else opted out of the noise. That is the explicit failure mode we are designing against.

### 1. No functional anonymity

Every account is tied to a verified identity (not necessarily publicly displayed, but known to us and attributable if needed) and a verified neighborhood/ward. Verification can be lightweight (email + address-adjacent neighborhood confirmation, not notarized ID), but there is no "guest commenting" and no disposable-account path. Anonymity is what turns a civic feedback tool into a shouting match; real-name-adjacent accountability is what keeps people closer to how they'd behave at an actual public hearing. This is a harder constraint than typical moderation — it's an architectural decision, not a policy you can bolt on later.

### 2. Structured input is the default; free text is the exception, not the norm

Noise mostly isn't bad actors — it's good-faith people rambling. Every feedback surface should lead with checkboxes/scales/structured choices, and treat open text as a narrow, optional elaboration on a structured answer, never the primary input. If someone wants to write a paragraph, that's a signal they have something specific to say — worth capturing precisely (see note on "traffic" below) — but the UI should never invite open-ended venting as the first move.

### 3. Reduce volume deliberately, not just by "letting the market decide"

NextDoor-style platforms converge on infinite scroll + reactions + algorithmic engagement, which literally rewards noise. We should do close to the opposite: rate-limit how often someone can comment per item, cap total visible comments per item (or require sorting/summarization above a threshold), and never optimize for time-on-app or engagement metrics. If an item has 40 pieces of feedback, the default view should be the *rolled-up summary*, with raw comments one click deeper — not a scrolling wall of takes. Engagement is not success here; a resident reading one clear summary and feeling informed is success.

### 4. Moderation is a precondition, not a bolt-on

Before any open-text feature ships: a clear community standard, rate-limiting/anti-brigading detection, and a human-in-the-loop review path — especially for anything that could target a protected class, a specific individual/business owner, or incite harassment (liquor license fights over who owns the bar are a predictable flashpoint). One bad incident can end this product's credibility; the bar for "ready to ship" on any feedback feature is high.

### 5. Preserve what people actually meant when rolling up feedback

"I'm concerned about traffic" could mean commute volume, pedestrian safety, speeding, or parking — and collapsing all of that into one aggregate stat makes officials *less* informed while looking more data-driven, which is worse than not collecting it. Rollups should: keep structured sub-tags where possible, use AI clustering on free text to catch what doesn't fit predefined tags (and surface representative quotes, not just a number), flag ambiguous/unspecified responses honestly rather than silently bucketing them, and avoid false statistical precision — "of respondents in this neighborhood," never "X% of the neighborhood."

### Net Goal

Every piece of information a resident or official sees through this tool should be worth their attention. If we can't guarantee that, we add a gate before we add a feature.
