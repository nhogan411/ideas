# Civic Pulse STL

An app concept to make local government activity in the St. Louis metro area (zoning changes, liquor licenses, school board actions, redevelopment deals, etc.) accessible, understandable, and personally relevant to residents — and to give residents a structured way to get their reactions back to officials.

**Status:** Early brainstorming / WIP. Nothing here is scoped, committed, or finalized — these docs capture current thinking so it isn't lost, not a locked-in plan.

## Files

| File | Contents |
|---|---|
| [`project-brief.md`](project-brief.md) | The core pitch: the problem (civic info is scattered, jargon-heavy, not personalized, one-directional), and the shape of a solution. Start here. |
| [`positioning.md`](positioning.md) | An application of April Dunford's "Obviously Awesome" positioning framework — competitive alternatives, unique attributes, target market, category. |
| [`data-models.md`](data-models.md) | Draft entity/data model thinking (e.g. `CivicItem` vs. `CivicItemEnrichment`, `SourceConfig`, `GoverningBody`) and the design principles behind the separation of raw fact from AI-generated interpretation. |
| [`notification-design-and-relevance-model.md`](notification-design-and-relevance-model.md) | The three-part core philosophy for how the app presents civic items ("what happened" → "why it matters" → "you decide") and the relevance/notification vocabulary. |
| [`moderation-and-signal-notes.md`](moderation-and-signal-notes.md) | Reasoning on why noise reduction and "no functional anonymity" are treated as first-class design constraints rather than bolt-on moderation policy, to avoid a NextDoor-style failure mode. |
