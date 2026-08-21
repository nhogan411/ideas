# Team Knowledge Wiki — Proposal

**Status**: Draft for discussion  
**Author**: Nick Hogan  
**Date**: August 2026

---

## The Problem

Questions like these currently take too long to answer, or don't get answered consistently:

- *"What is the WP way of doing X?"*
- *"How does Application A interact with Application B?"*
- *"Is this component accessible? What would make it accessible?"*
- *"How is our Bedrock stack different from vanilla WordPress?"*
- *"Would doing X break our existing patterns?"*
- *"What does this component do, what are its variants, and where's the Figma spec?"*

Answering any of these today requires knowing who to ask, digging through multiple repos, or reverse-engineering intent from code that was never documented. The knowledge exists — it's in READMEs, AGENTS.md files, Storybook stories, Figma specs, GraphQL schemas, and the heads of the people who built these systems. The problem is that it's scattered, siloed, and degrades over time as systems change.

This is a proposal for a system that synthesizes that knowledge, keeps it current automatically, and makes it self-serve.

---

## The Concept

Andrej Karpathy described the core idea in April 2026: add source documents to a folder, use an LLM to extract and organize their contents into a Markdown wiki, and keep it updated as sources change. Casey Newton at Platformer put it into practice for journalism — feeding his entire article archive and daily reading into a self-organizing knowledge base he queries every morning.

The adaptation for a software team: instead of news articles, the sources are your repos. Instead of personal research context, the output is shared team context — architecture decisions, API references, component docs, cross-system relationships.

**The key design principle:** the wiki doesn't become a second source of truth. It generates a *derived view* of documents that already have canonical homes (READMEs, Storybook, Figma specs), linking back to those sources and regenerating automatically when they change. Every page carries a "last synced against [commit / version / date]" stamp so staleness is visible rather than silent.

Newton's experience is worth noting directly: this approach works and delivers real value, but it requires maintenance (pages grow too long, processes break), and LLM-generated docs have a tendency toward dense, compressed prose that's hard to read. Both are solvable design problems, not showstoppers — but they're real costs to plan for.

---

## What We'd Generate

### Source-Tracked Pages (low risk, safe to automate fully)

These are templated extraction jobs drawing from one or two well-defined sources. Low risk of being subtly wrong.

**Per-repo reference pages** — for every significant repo, a page summarizing: what it does, how it's structured, how to run it locally, its key dependencies, and links to its actual docs. Generated from README + docs/ + AGENTS.md on every push to main.

**Per-component pages** — for every component in the GCU component library, a page merging: Storybook usage/props/variants (already machine-readable via react-docgen-typescript), accessibility audit results (already being captured by the Storybook a11y addon), and a link to the Figma design spec. Generated on component library releases.

**External reference pages** — WCAG accessibility criteria, curated technology references (WordPress/Bedrock patterns, Next.js, Drupal, Tailwind). Synced weekly or on manual flag.

### Synthesis Pages (higher value, higher risk — require human review)

These require reasoning across multiple sources, not just extraction. Generated as pull requests; a human reviews and merges before they're treated as authoritative.

**Cross-application relationship pages** — "How does Application A talk to Application B?" Currently the best available answer is: read four different AGENTS.md files and hope they agree.

**Stack comparison pages** — "How is our Bedrock WordPress setup different from vanilla WordPress?"

**Pattern/impact reasoning** — "Would doing X be possible here, or would it break our existing conventions?"

---

## What This Looks Like For Our Stack Specifically

This section is grounded in an actual scan of both GitHub orgs (~95 repos in `gce-digital-marketing`, ~50 in `gce-digital-marketing-private-libraries`).

### The cross-application data flow — completely undocumented

The following data flow was confirmed from the code. It does not exist as a diagram or document anywhere:

```
Drupal (GCU-drupal-10)
  └─ exposes GraphQL with custom bulk queries (gcu_graphql module)
       │
       ▼
ACD (API-Core-Data)
  ├─ authenticates to Drupal via POST /user/login
  ├─ queries Drupal GraphQL + XRM endpoints
  └─ exposes REST at /api/v1/* (programs, AOIs, degree-levels, etc.)
       │
       ├──────────────────────────────────────────┐
       ▼                                          ▼
GCU-nextjs                                  Form Builder
  ├─ consumes Drupal GraphQL directly         └─ optionally fetches from ACD
  └─ consumes ACD REST via CORE_DATA_API_URL    (dataset.source = "acd")
       │
       ▼
NEXT_PUBLIC_RMI_API_URL → Form Builder
  (client-side form submissions)
```

This is the kind of cross-application synthesis page that currently requires asking someone who's been here long enough to know. A wiki page with this diagram — verified against source code and refreshed when any of these repos change — would compress hours of onboarding into minutes.

### The WP plugin dependency graph — machine-readable, currently invisible

We have 20+ WordPress site repos (WP-gcu-events, WP-gcu-hotel, WP-Madonna, WP-Orbis, etc.), all Bedrock-managed with composer.json files listing which plugins and which versions each site has installed. The question "which sites are running version X of wp-plugin-form-builder?" currently requires checking each repo individually. The data is all there — it's just not synthesized anywhere. A generated dependency map per plugin would be immediately useful for release planning and DevOps.

### The FORMS disambiguation — answered nowhere

Eight FORMS repos exist: FORMS-drupal, FORMS-drupal-php8, FORMS-dual-enrollment, FORMS-gce-contact-form, FORMS-landing-pages-php7, FORMS-utilities, FORMS-webservices, and Form-builder. There is no document that explains when to use which, or what the relationship between them is. A developer encountering a forms requirement has to ask around. This is exactly the kind of question a wiki synthesis page can answer once and keep current.

### Component library — already rich in machine-readable metadata

The GCU component library's Storybook setup is more capable than most:

- **react-docgen-typescript** is enabled → TypeScript prop types are already being extracted automatically
- **@storybook/addon-docs** is installed → documentation is already being generated per component
- **@storybook/addon-a11y** is installed → accessibility violations are already being detected and the data is machine-readable

This means per-component wiki pages aren't starting from nothing. Prop tables, a11y pass/fail results, and story variants are all extractable today without touching Figma. Figma integration adds design spec linkage on top of an already useful base.

Additionally, the component library already has a `figma-convert` script that uses Figma MCP to generate component code from design specs — meaning Figma authentication and API access are already established. The wiki pipeline would use the same access differently (reading design specs for documentation rather than code generation).

### The LL (Live Leads) system — five repos, no overview

The Live Leads system spans five repos: LL-backend (Express/TypeScript/Prisma/PostgreSQL), LL-frontend, LL-leads-api, LL-live-leads-service, and LL-main. There is no cross-repo architecture document. An agent can produce one from the codebases.

### Other synthesis page candidates

- **Lambda fleet registry** — six lead-routing Lambda functions (by partner: CSP, Madonna, Naviance, NEU, STK, YouVisit). What triggers each? What do they share?
- **ML pipeline overview** — four repos (ML-bert-api, ML-elastic-reporting, ML-model-training, ML-training-data-generation). Presumably a pipeline. No system-level view.
- **Env var canonical registry** — DRUPAL_BASE_URL, CORE_DATA_API_URL, NEXT_PUBLIC_RMI_API_URL scattered across .env.example files and AGENTS.md notes across repos. One page mapping each var to what reads it and what it points to.
- **AGENTS.md synthesis** — every major repo has an AGENTS.md encoding team conventions. A synthesized cross-repo conventions page would reduce onboarding friction.

---

## Proposed Architecture

### Storage

Plain Markdown files in a new git repo (`team-wiki`), not inside the existing Documentation MkDocs site. Reasons:

- The Documentation repo has a different governance model (manually maintained) and an uncertain future (possible migration to a different platform). Building generated content inside it creates coupling.
- A separate repo means generated and manually-maintained content can't accidentally overwrite each other.
- Both repos cross-link to each other; neither replaces the other.

The existing `Documentation` repo becomes a **git submodule** in `team-wiki`. When Documentation changes, the submodule pointer updates and the synthesis script re-reads it. Other source repos (ACD, GCU-nextjs, GCU-component-library, etc.) are fetched at synthesis time rather than as permanent submodules — avoiding submodule sprawl across 90+ repos.

Every generated page carries frontmatter:

```yaml
---
source: gce-digital-marketing/API-core-data
source-type: repo-docs        # repo-docs | component | external-ref | synthesis
last-synced-ref: a3f9bc2      # commit SHA, semver, or fetch date
generated: 2026-08-21T14:00Z
owner-team: backend
---
```

### Sync Triggers

| Source | Trigger | Output |
|---|---|---|
| Any repo push to main | GitHub Action (source repo) | Source-tracked page regenerated |
| Component library release | `publish-package.yml` hook | Per-component pages regenerated |
| External references | Weekly cron | Reference pages regenerated |
| Synthesis pages | Monthly scheduled run | Draft PR opened; human reviews and merges |

**Critical constraint:** wiki generation must never block source repo PR merges. If the LLM API is unavailable, the source repo's CI should be completely unaffected. Wiki generation runs as a fire-and-forget dispatch — success or failure doesn't gate anything.

### Staleness Detection

A weekly scheduled job compares each page's `last-synced-ref` against the source repo's current HEAD. Pages that are more than 7 days behind get flagged (issue opened or label added). This keeps staleness visible rather than silent.

### Query Layer (v1)

No new tooling. Developers clone the wiki repo and use opencode or Claude Code pointed at the folder. Since everything is plain Markdown in a git repo, any LLM tooling that reads a directory works immediately.

The internal ai-platform (which already has vector search infrastructure — pgvector + Weaviate — and a configurable Knowledge Management System) is the natural upgrade path for a proper chatbot interface once the content quality is validated. That's phase 2.

### Two-Pass Synthesis

Source material goes through two LLM calls, not one:

1. **Extraction pass** — structured extraction of facts, endpoints, relationships, constraints. Citations required for every claim. If something isn't documented in the source, output `[NOT DOCUMENTED]` rather than inferring.

2. **Style pass** — rewrite the extracted content into readable prose. Short paragraphs over nested bullets. Concrete examples over abstract descriptions. Lead with "what problem does this solve?" Newton explicitly needed this step; Claude's default output style is dense and hard to read at the level of a busy developer skimming docs.

---

## Proposed Build Order

### Phase 1: Prove the pipeline (pilot)

**Scope**: One repo, one generated page, manual trigger only.

Pick `API-core-data` as the pilot target. It has AGENTS.md + README + docs/, it sits at the center of the cross-app data flow, and people who know the system can immediately assess whether the output is accurate.

Deliverables:
- `team-wiki` repo created with frontmatter schema
- `Documentation` repo added as git submodule
- A single GitHub Action in `API-core-data` that generates a wiki page on push to main
- Two-pass LLM pipeline (extraction → style)
- Manual review of output quality by engineers who know the system

**Do not proceed to phase 2 until people who know ACD say the generated page is accurate and useful.**

### Phase 2: Roll out to source-tracked pages

**Scope**: Per-repo pages for the ~10 highest-value repos, via required_workflows.

Targets (in suggested order):
1. GCU-nextjs
2. GCU-drupal-10
3. Form-builder
4. WP-base (and then each WP plugin)
5. LL-backend (and LL system)

### Phase 3: Component library pipeline

**Scope**: Per-component pages for GCU-component-library.

Extract from: Storybook metadata (react-docgen-typescript output, story variants), a11y addon results, Figma spec links. Trigger on component library releases.

### Phase 4: Synthesis pages

**Scope**: Cross-application relationship pages, stack comparison pages, disambiguation pages.

Generated as PRs. Require human review and merge before treated as authoritative. Suggested first targets: cross-app data flow diagram, FORMS disambiguation, WP plugin dependency map.

### Phase 5: ai-platform query layer

**Scope**: Index wiki content into the internal ai-platform's Knowledge Management System for a proper chatbot interface.

Depends on ai-platform maturing from experimental to active use.

---

## Risks and Mitigations

### LLM output quality

**Risk**: Generated pages contain confident-sounding factual errors. A wrong architecture diagram in an internal wiki is worse than no wiki.

**Mitigation**: Citation-required extraction (every claim traces to a source file + line). Style pass that preserves citations. Synthesis pages go through human review before publication. Source-tracked pages are clearly stamped as generated and link back to canonical source for verification.

### Prose readability

**Risk**: LLM output is dense, compressed, and hard to read. (Newton documented this specifically, and it's consistently observed in practice.)

**Mitigation**: Explicit two-pass pipeline with a style prompt that instructs plain prose, not nested bullets. Explicit style guide for the extraction prompt. Consider testing with different models for the style pass — Newton found different models have different default writing styles.

### Maintenance burden

**Risk**: CI processes break, pages grow too long and need compacting, coverage gaps accumulate. The system requires ongoing tending.

**Mitigation**: Someone needs to own this. The wiki doesn't maintain itself; it reduces the maintenance burden compared to manual docs, but it doesn't eliminate it. This is the most important operational decision: who is the DRI?

### CI coupling

**Risk**: If wiki generation is added to required_workflows and the LLM API has an outage, every team's PR merges could be blocked org-wide.

**Mitigation**: Non-negotiable constraint: wiki generation must be fire-and-forget (async dispatch, never blocking). The LLM call's success or failure is logged and alerted on, but it does not affect source repo CI status.

### Private repo content leakage

**Risk**: If the wiki repo is public, synthesized content from private repos (gce-digital-marketing-private-libraries) could leak.

**Mitigation**: Wiki repo should be private by default. Revisit if there's a reason to make it public.

### Documentation repo migration

**Risk**: If Documentation migrates to "zensical" (planned, timeline unknown), the submodule approach may need updating.

**Mitigation**: The separate-repo architecture insulates the wiki from Documentation's tooling decisions. The submodule pointer simply updates to wherever Documentation lands.

### Race conditions on simultaneous pushes

**Risk**: Two repos push to main simultaneously. Both trigger wiki generation. Both try to commit to the wiki repo at the same time, causing a git conflict.

**Mitigation**: Serialize wiki commits using GitHub Actions concurrency groups. At most one wiki commit runs at a time; others queue.

---

## Open Decisions

These need answers before implementation begins, but don't need to be resolved for this proposal to be evaluated:

| Decision | Options | Implications |
|---|---|---|
| Wiki repo visibility | Private (safer), Public (easier access) | Affects leakage risk, query access |
| DRI / owner | Named team lead, rotating duty | Without a DRI, synthesis PRs won't get reviewed |
| Synthesis PR reviewer | Specific team or role | Required before Phase 4 |
| Quality gate for org-wide rollout | Human sign-off, automated checks, or both | Gates Phase 2 |
| Cross-org CI auth | GitHub App (recommended), PAT | Required before Phase 1 can run in CI |
| Staleness threshold | Suggested: 7 days | Affects alert volume |

---

## What This Is Not

- A replacement for the Documentation repo
- A second source of truth (canonical docs stay where they are; wiki holds synthesized/derived views)
- A real-time system (event-triggered per repo push, not continuous)
- A public-facing documentation site
- A chatbot (v1 query = opencode pointed at folder; chatbot is phase 5)
- Zero-maintenance (it requires a DRI and ongoing tending)

---

## Recommended Next Step

Run a one-day spike: pick API-core-data, manually extract its README + AGENTS.md + docs/, send it through a two-pass LLM prompt, and evaluate whether the output is accurate and useful. This answers the most important unknown (is the output quality good enough?) before any infrastructure decisions are made.

The rest of the architecture can be built incrementally. The quality question needs to be answered first.
