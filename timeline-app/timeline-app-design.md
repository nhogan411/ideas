# Timeline App — Design Spec
*Created: August 19, 2026*

---

## Origin & Motivation

When a family member is seriously ill, you end up communicating a long, convoluted history of events to doctors, nurses, insurers, and specialists — over and over. You naturally forget when things happened, or forget events entirely. The same problem shows up in other contexts: the work done on a house over the years, the trips you've taken, the history of a car, a journalist building a chronological record of an investigation.

This app is a structured, collaborative, embeddable timeline tool. It lets individuals and groups capture events with flexible dates, organize them with tags, control who sees and contributes to what, and share or embed the result.

---

## What It Is

A **SaaS web application**, API-first in architecture. Users sign up, get a single private timeline by default, and start logging. Organization, sharing, and publishing are layered on top as needed — not required upfront.

### It is not:
- An npm/pnpm package (the visualization layer may be extracted as one later, once we know what it actually needs to be)
- A native mobile app (responsive web only, built mobile-first for entry creation)
- A calendar or task manager
- A social network

---

## Core Philosophy

**Private by default. Always.**

Every entry, tag, and timeline is private to the owner unless explicitly shared. Users opt *into* sharing — they never have to opt out. This is a stated product value, not just a technical default. It should appear on the landing page, in onboarding, and in the privacy policy.

**No advertising. No data selling. You own your data.**

Users can export everything or delete everything at any time. This commitment is written in plain English and surfaced prominently — not buried in legal docs.

---

## Use Cases

The app is general-purpose, but these are the clearest target scenarios:

- **Family medical timelines** — track a loved one's health history collaboratively across caregivers, share selectively with doctors
- **Personal life timelines** — trips taken, home improvements, car maintenance, major life events
- **Investigative journalism** — build a chronological record of a story, embed it as an interactive piece for readers
- **Legal timelines** — personal injury, custody disputes, insurance claims; collaborative with lawyers/paralegals
- **Property history** — every repair, upgrade, inspection, and appliance purchase on a home
- **Pet health records** — vet visits, medications, weight over time
- **Sales operations** — prospect touchpoints, deal milestones, closed dates and amounts
- **Academic research** — source tracking, reading history, how an argument developed
- **Genealogy** — family history passed down and collaboratively built

---

## Data Model

### Timeline

Every user gets one timeline by default. A timeline is the top-level container for all of a user's entries.

- **Visibility**: private (default) or public (anyone with the link can view)
- Future: users may be able to create multiple named timelines, but v1 starts with one

### Entry

The atomic unit of the app. Every entry belongs to a timeline.

| Field | Notes |
|---|---|
| `title` | Short label for the event |
| `note` | Free-text description |
| `start_datetime` | The start of the event |
| `start_precision` | How precise `start_datetime` is (see Date Precision) |
| `end_datetime` | Optional. For events with duration |
| `end_precision` | Precision of end date |
| `timezone` | The timezone in which the entry was recorded (e.g., `America/Chicago`) |
| `tags` | Array of tag IDs applied to this entry |
| `created_by` | User who created the entry |
| `created_at` | Server timestamp |
| `entry_permissions` | Optional overrides (see Permissions) |

Entries also carry an **edit history** — a log of every change, who made it, and when. Non-deletable.

### Date Precision

Entries do not require a specific date and time. Precision is stored alongside the date so the app knows how to display and sort the entry.

| Precision level | Example | Stored as |
|---|---|---|
| `datetime` | March 14, 2020 at 2:30pm | Exact timestamp + timezone |
| `day` | March 14, 2020 | Date only, no time |
| `month` | May 2020 | First day of the month |
| `season` | Summer 2020 | First day of the meteorological season (Spring: Mar 1, Summer: Jun 1, Fall: Sep 1, Winter: Dec 1) |
| `year` | 2020 | January 1 of the year |
| `approximate` | Around 2018 | A year value with a ±1 year margin stored alongside it; displayed as "Around [year]" |

- Entries with fuzzy dates render as a block spanning their precision range in the timeline visualization — a "May 2020" entry occupies the full month, not a single day
- Zooming in on the timeline does not make fuzzy entries more precise — there is simply no more data to show
- Fuzzy entries are sorted by their stored start date. `approximate` entries sort by the central year. Ties are broken by creation order.

### Timezone Handling

- Each entry stores its original timezone alongside the datetime
- **Display default**: all datetimes are shown converted to the viewer's local timezone
- **Toggle**: viewers can switch to "show in original timezone" — all entries display as they were at the time and place they were recorded
- **Entry creation**: defaults to the device's local timezone; the creator can override this (e.g., logging something that happened last week in a different city)

### Tag

Tags are the organizational and sharing unit of the app. Any entry can have any number of tags. Tags are flat — no hierarchy.

- **Private by default** — a tag with no sharing rules is visible only to its owner
- **Invisible to unauthorized users** — if a viewer does not have permission to see a tag, that tag does not appear on the entry. They cannot tell the tag exists.
- Tags are created on the fly during entry creation (search-and-select with inline creation)
- Permissions are configured on the tag, not on individual entries (though entry-level overrides exist — see below)

### User Groups

Users can create named groups of other users (e.g., "Dad's Care Team", "Extended Family"). Groups can be granted permissions on tags, behaving identically to individual users.

- Groups can hold **View** or **Contribute** permissions on a tag
- **Admin** permission stays with named individuals only — groups cannot hold Admin
- Adding a user to a group grants them everything that group has access to; removing them revokes it

---

## Permissions Model

### Tag-Level Permissions

Three levels:

| Level | What they can do |
|---|---|
| **View** | See entries with this tag; read-only |
| **Contribute** | Add new entries with this tag; can edit or delete their own entries; cannot edit or delete entries created by others |
| **Admin** | Full access: add, edit, delete entries; manage who else has access to this tag |

Permissions can be granted to individual users or groups.

### Permission Resolution

When an entry has multiple tags with different permission levels for the same viewer:

- **Visibility**: a viewer sees only the tags they have permission to see. Tags they cannot see do not appear on the entry — and they have no indication those tags exist.
- **Action level**: the viewer's effective permission is the **most permissive** level they hold across any visible tag on that entry. View on Tag A + Contribute on Tag B = Contribute on the entry.

### Entry-Level Permission Overrides

In addition to tag-based permissions, individual entries can carry their own permission rules. These can go in either direction:

- **Restrict**: Tag X is public, but this specific entry is marked private. The entry does not appear publicly, regardless of Tag X's permissions.
- **Grant**: This entry has no shared tags, but the owner shares it directly with a specific user (e.g., sharing one entry with a doctor without granting access to an entire tag).

Entry-level permissions take precedence over tag-level permissions.

### Access Logs

For any tag with sharing enabled, the owner can see an access log: who has access, and the last time each person viewed entries with that tag. Transparency as a feature — "trust us" becomes something verifiable.

---

## The API

The app is API-first. The web application, the embed system, and all third-party automations are consumers of the same API.

### Authentication

| Mode | Used by |
|---|---|
| Session auth (cookie/JWT) | The web app — logged-in users |
| API key auth | Programmatic access — automations, Zapier, Apple Shortcuts, scripts |

API keys are created and managed by the user in their account settings. Keys can be scoped (e.g., create entries only) and revoked at any time.

### Core Endpoints (v1)

- `POST /entries` — create an entry
- `GET /entries` — list entries (filterable by tag, date range, precision)
- `PATCH /entries/:id` — update an entry
- `DELETE /entries/:id` — delete an entry
- `GET /entries/:id/history` — full edit history for an entry
- `GET /tags` — list tags
- `POST /tags` — create a tag
- `PATCH /tags/:id` — update a tag
- `GET /tags/:id/permissions` — get permissions for a tag
- `POST /tags/:id/permissions` — grant access to a user or group
- `DELETE /tags/:id/permissions/:permission_id` — revoke access
- `GET /tags/:id/access-log` — view access log for a tag
- `GET /groups` — list user groups
- `POST /groups` — create a group
- `PATCH /groups/:id` — update a group
- `POST /groups/:id/members` — add a member
- `DELETE /groups/:id/members/:user_id` — remove a member
- `GET /embed/:key` — public embed data endpoint
- `GET /timeline` — get the user's timeline
- `PATCH /timeline` — update timeline settings (visibility, etc.)

### Automation

Any tool that can make an HTTP POST request can add an entry on day one. This covers Zapier, Make, Apple Shortcuts, Home Assistant, custom scripts — without any native integration work required. API key in the header, entry fields in the body, done.

**Webhooks** (outbound — notify external systems when entries are added) are a post-v1 feature.

---

## The Web App

### Entry Creation (Mobile-First)

The entry creation flow must be fast and frictionless. A user in a hospital waiting room cannot fight the UI.

- **Visible by default**: title, date/precision picker, note
- **Tag picker**: search-and-select with inline tag creation; appears below the note
- **Date input**: does not force a specific day — user selects the precision level they have ("I know the month" / "I know the exact date and time")
- **Timezone**: defaults to device local timezone; user can override
- Submit is a single tap/click

### Timeline Visualization (Desktop-Primary)

The core display challenge: a single timeline might span decades with some entries that are years-long and others that are hour-specific. A fixed scale cannot represent both meaningfully.

**Zoomable scale**: users zoom in and out along the time axis. Zoomed all the way out: years are visible. Zoomed in: days or hours. Entry density adjusts to the zoom level — a "May 2020" entry always occupies its full month block regardless of zoom, because there is no more precision to reveal.

**Tag filtering**: selecting one or more tags collapses the view to only matching entries. This is the core collaborative viewing experience — "show me only Dad Medical."

**Entry detail**: clicking an entry expands it in place (or opens a side panel) to show the full note, all visible tags, who created it, when, and its edit history.

### Mobile Visualization

On mobile, the timeline visualization becomes a **chronological scrollable list**, filterable by tag. The graphical zoomable timeline is a desktop feature.

A true **vertical mobile timeline** (rotate the axis, scroll down through time) is a near-term priority after the initial list view — not a distant future item. Vertical scroll is native to the phone; this is the right long-term mobile visualization.

---

## The Embed

Users can embed a view of any tag or their full timeline into an external page (a news article, a website, a documentation page) via an iframe snippet or JavaScript embed code.

### How It Works

- Owner selects a tag (or their timeline), generates an embed key, gets a code snippet
- The embed renders using **existing tag and entry permissions** — no separate publish queue
- If an entry is visible to the public under the current permissions, it appears in the embed. If it's private, it doesn't.
- To control what appears in an embed, the owner controls entry-level and tag-level visibility directly

### The Embedded Experience

- Renders the zoomable visualization on desktop; falls back to a vertical scrollable list on mobile (same tradeoff as the main app)
- Readers can zoom in and out, filter by tags (if multiple are included), and click entries for detail
- Supports a timezone toggle — readers can switch between their local timezone and the source timezone
- Accepts basic theming (light/dark mode, accent color, font size) so it doesn't look jarring in a publisher's design

### Performance

CDN caching and edge delivery for embed traffic is a future infrastructure concern. Not a v1 problem.

---

## Export & Import

### Export (v1)

Users can export their timeline or any tag-filtered view as:
- **PDF** — a clean, readable document suitable for handing to a doctor, lawyer, or insurer
- **CSV** — tabular data, one row per entry
- **JSON** — full structured data export, suitable for backup or migration

### Import (future)

- **MyChart / HL7 FHIR** — import health records directly from patient portals. The FHIR standard is the path here.
- **CSV import** — bulk entry creation from a spreadsheet

---

## Data Privacy & Security

### Technical

- Encryption at rest (database-level) and in transit (HTTPS everywhere)
- Two-factor authentication support
- Full account and data deletion — user initiates, everything is removed, no shadow copies

### Policy (stated prominently — not buried in legal docs)

> *Private by default. No advertising. No data selling. You own your data and can take it with you or delete it entirely at any time.*

This appears on the landing page, in the onboarding flow, and in the privacy policy — written in plain English.

### Future Consideration

End-to-end encryption (where even the operator cannot read user data) is architecturally possible but makes server-side search significantly harder. Not a v1 decision, but worth knowing the trade-off exists.

---

## Search

Full-text search across entries (title, note) is essential and will become critical fast. Included in v1. Filterable by tag, date range, and precision level.

---

## Edit History

Every change to every entry is logged: what changed, who changed it, when. The log is non-deletable and visible to the entry owner and anyone with Admin on a relevant tag. Critical for medical and legal use cases.

---

## Post-v1 Features

The following are explicitly out of scope for v1 but worth tracking:

| Feature | Notes |
|---|---|
| Notifications | When someone adds to a tag you follow. Needs granular preferences to avoid fatigue. |
| Comments on entries | Threaded discussion on a specific event |
| Webhooks | Outbound — notify external systems when entries are added |
| MyChart / FHIR import | Import health records from patient portals |
| Vertical mobile timeline | Near-term priority after launch |
| CDN/embed performance | Edge caching for high-traffic embeds |
| Multiple timelines per user | v1 starts with one |
| Groups with Admin permission | Currently Admin stays with named individuals only |
| Hierarchical tags | Not needed yet; may never be |
| End-to-end encryption | Trade-off with search; revisit if user demand warrants it |
| npm visualization package | Extract if the visualization layer proves genuinely reusable |

---

## Known Risks & Design Challenges

**Fuzzy date sorting at scale**
When a timeline has hundreds of entries and many are approximate ("sometime in 2019"), ordering and display get messy. The sorting rule (earliest possible date in range, ties broken by creation order) handles this, but the UI needs to communicate ambiguity clearly rather than presenting approximate entries as confidently ordered.

**Onboarding**
A blank timeline is intimidating. New users need a fast path to "this is useful" — sample entries, a guided first entry flow, or some form of immediate value before they give up.

**Notification fatigue** *(post-v1)*
Eight people contributing to a shared tag means a lot of potential noise. Notification preferences need to be granular when this feature ships.

**Permission model complexity**
The permissions model is powerful but not simple. The UI for managing tag permissions, groups, and entry-level overrides needs to make the mental model legible — especially for non-technical users sharing a medical timeline.

---

## What This Is Not (Guardrails)

To keep scope honest:

- Not a calendar — entries don't recur, don't have reminders, don't block time
- Not a task manager — no assignees, due dates, or completion states
- Not a social network — no follows, feeds, or public discovery of other users' timelines
- Not a document editor — notes are text, not rich documents
- Not a native mobile app — responsive web only for v1
