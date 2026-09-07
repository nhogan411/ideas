# HTML-to-Markdown Batch Scraper — Design

## Purpose

A general-purpose tool that takes a batch of URLs (from a `.csv`, `.xlsx`, or `.md` file) and produces clean, review-ready Markdown files on disk — a drop-in replacement for the Obsidian Web Clipper's "clip this page" action, but batch-driven and scriptable.

This is **not** a recipe-specific tool. Recipe blogs are one of many source types (food blogs, technique articles, general web content across arbitrary industries/sectors) that will feed the broader recipe-wiki / LLM-wiki project. The tool must stay generic; any recipe-specific structuring happens downstream in the existing LLM ingest step, not here.

## Problem Statement

The current workflow (Obsidian Web Clipper, one URL at a time) produces Markdown that requires heavy manual review before it's usable. Inspecting a real example (`raw/Half Baked Harvest/Better Than Takeout Sweet Thai Basil Chicken..md`) shows the core issue: **junk/boilerplate is not stripped** — tracking ad iframes and newsletter-signup/promo banners survive into the output alongside the real content. Note: not everything that looks like "extra" content is junk — e.g. this domain's Instacart shopping links attached to each ingredient are wanted, not noise, and must be preserved by any cleanup rule (see Per-Domain Override Rules below).

The goal is to reduce this to near-zero for domains the tool has seen before, and to make what's left easy to triage rather than requiring every file to be opened and read.

## Non-Goals (v1)

- Local image downloading (see Future Extensions — seam is built in, not implemented)
- Writing directly into `raw/` (output directory is a config flag; switching later is a one-line change, not a redesign)
- Structured recipe-schema (JSON-LD) extraction — this stays a generic content scraper
- Paywall/login-wall bypass
- LLM-based content cleanup (kept as a possible future fallback, not primary mechanism)

## Architecture

Runs inside Docker. No local Python install or virtualenv required.

```
docker-compose.yml + Dockerfile + justfile
  - justfile wraps docker compose commands into short recipes (just run, just test-rule, just retry, just build)
    -> this is the primary interface documented for day-to-day use; only host dependency is `just` itself
  - Image ships with: Python, trafilatura, readability-lxml, httpx, markdownify,
    python-slugify, playwright (+ browser binaries), openpyxl
  - Volumes:
      ./input       -> URL list files (csv/md/xlsx)
      ./output      -> written Markdown output (config: --output-dir, default ./output)
      ./domains.yml -> per-domain override rules (editable without rebuilding image)
      ./state       -> run-state.json / run-report.md per input file
  - Only rebuild the image when dependencies change (requirements.txt edits).
    Editing domains.yml or dropping in a new input file needs no rebuild.
```

### Pipeline (per URL)

```
1. Input parsing       -> normalized, deduped URL list
2. Fetch               -> httpx (static) -> Playwright fallback if content looks thin
3. Per-domain pre-clean -> strip known-junk selectors/patterns for this domain (domains.yml)
4. Generic extraction  -> trafilatura (primary) -> readability-lxml (fallback)
5. Quality flagging    -> heuristics decide needs_review: true/false
6. Markdown conversion -> markdownify + YAML frontmatter
7. Write               -> <output-dir>/<domain>/<slug>.md
8. Run report          -> append status (done/failed/flagged) to run-report.md
```

## Components

### 1. Input Parsing

- Supports `.csv`, `.xlsx` (via `openpyxl`/`pandas`), and `.md`.
- Markdown parsing must handle the existing convention seen in `raw/pinterest-recipe-links.md`: `- [ ] https://...` and `- [x] https://...` checkbox-list URLs, as well as bare URLs and `[text](url)` links.
- Normalization: strip `utm_*` and other tracking query params, strip whitespace, dedupe exact and normalized duplicates, validate with `urlparse` and drop garbage rows before fetch.

### 2. Fetching

- `httpx` for static HTML by default. Real `User-Agent`, explicit timeouts on every request (never unbounded).
- Per-domain rate limiting to avoid hammering any one site.
- After extraction (step 4), if the resulting content is suspiciously short (below a word-count threshold), the URL is automatically retried once using Playwright (headless browser) to handle JS-rendered pages, then re-run through extraction. No upfront per-URL configuration needed to decide "does this site need JS."

### 3. Per-Domain Override Rules (`domains.yml`)

This is the primary mechanism for driving manual review toward zero over time, since the same domains recur across many URLs.

- Applied to raw HTML *before* generic extraction runs.
- Schema:
  ```yaml
  halfbakedharvest.com:
    strip_selectors:
      - "iframe[src*='html-load.com']"
      - ".newsletter-signup-banner"
    strip_text_patterns:
      - "Sign up for my newsletter"
  ```
  Note what's deliberately *not* in this example: the Instacart shopping links attached to each ingredient. Those are useful (a legitimate "buy these ingredients" service the user wants to keep using), not noise — a rule should target the specific junk element (the ad iframe, the newsletter banner), never a blanket "strip all links" pattern that would take out wanted content along with it.
- Versioned in git alongside the rest of the tool — history of what was added, when, and (via commit message) why.
- Workflow for adding a new rule (documented step by step in README, using the real Half Baked Harvest example as the worked walkthrough):
  1. Notice recurring junk in an output file for domain X.
  2. Add or extend the `domains.yml` entry for X.
  3. Test the rule against a single known-bad URL: `just test-rule <url>` — prints before/after diff of the extracted content, no batch run needed.
  4. Once satisfied, the rule applies automatically to every future URL on that domain.
- `domains.yml` documentation is a first-class deliverable — not an afterthought. Must include the full schema reference, the worked example above, and the test-rule command.

### 4. Generic Extraction

- `trafilatura` extracts main content + available metadata (title, author, publish date) from the cleaned HTML.
- If trafilatura's output is empty or below a minimum length threshold, fall back to `readability-lxml`.
- This stage is domain-agnostic; it's the safety net for domains without (or before) a custom rule.

### 5. Quality Flagging

Runs after extraction, before writing to disk. Cheap heuristics computed per file:

- Word count below threshold (extraction likely failed or page was mostly non-content)
- Leftover `<iframe>`/`<script>` residue that survived extraction (a strong signal of ad/tracking cruft, not a signal about legitimate content links like Instacart's — this heuristic checks for raw script/iframe tags specifically, not link density, so it won't flag wanted shopping links)
- Suspiciously repeated phrases/blocks (a sign of un-stripped repeated widgets, e.g. a newsletter banner appearing 3 times on one page)

Any file tripping a threshold gets `needs_review: true` in its frontmatter and is listed separately (with the specific reason) in `run-report.md`. This turns "read every file" into "read the 3-5 flagged files."

### 6. Markdown Conversion & Frontmatter

- `markdownify` converts cleaned HTML to Markdown (headings, lists, tables, code blocks preserved).
- Images: **kept as remote URLs in v1** (not downloaded). See Future Extensions for the planned local-image pass.
- Frontmatter schema (YAML), consistent across all sources:
  ```yaml
  ---
  title: "..."
  source: "https://..."
  domain: "halfbakedharvest.com"
  author: "..."          # if available
  published: "..."        # if available
  scraped: "2026-09-07"
  needs_review: false
  review_reason: ""       # populated only if needs_review is true
  ---
  ```

### 7. Output Layout

- Output root is a config flag (`--output-dir`, default `./output`), never hardcoded — switching to write directly into `raw/` later (per user's stated future intent) is a one-line change, not a redesign.
- Structure: `<output-dir>/<domain-name>/<slug>.md` — one file per URL, folder per source domain (mirrors the existing `raw/` convention of folder-per-source).
- `run-report.md` per run: summary counts (succeeded/failed/flagged) plus a per-URL line with status and, for flagged/failed, the reason.

### 8. Batching & State

- Designed for 10-50 URLs per run (user's stated preferred batch size), with the workflow well-documented in the README:
  1. Split your master URL list into batches of 10-50.
  2. Run batch 1.
  3. Review `run-report.md`, spot-check flagged files.
  4. Run batch 2, etc.
- `run-state.json` (per input file) tracks `pending / done / failed / flagged` per URL. Re-running the same input file automatically skips URLs already marked `done`.
- `docker compose run scraper retry --failed-only` re-attempts only failed/flagged URLs from the last run against that input file (exposed as `just retry <input-file>`).

### 9. CLI

Raw `docker compose run` commands work directly, but the primary interface is a `justfile` (https://github.com/casey/just) that wraps them into short, memorable recipes — this is what the README teaches and what the day-to-day workflow uses.

```bash
just init                         # one-time setup: docker compose build + create ./input, ./output, ./state dirs + seed a blank domains.yml if missing
just up                           # docker compose up -d (starts any long-lived services, e.g. if a Playwright browser container runs standalone)
just down                         # docker compose down
just run urls.csv                 # docker compose run scraper run --input input/urls.csv --output-dir output --batch-size 25
just run urls.csv 10              # optional batch-size override
just test-rule <url>              # docker compose run scraper test-rule <url>
just retry urls.csv               # docker compose run scraper retry --failed-only --input input/urls.csv
just build                        # docker compose build (only needed after dependency changes)
```

`just init` is the documented first command in the README (fresh clone → `just init` → ready to run). `just up`/`just down` manage the compose stack's lifecycle explicitly, which matters if the architecture ends up with a standalone long-lived service (e.g. a persistent Playwright browser container reused across runs rather than started fresh per invocation) rather than pure one-shot `docker compose run` calls — this decision is deferred to the implementation plan, but the recipes exist either way since `down` is also the correct way to clean up dangling containers/volumes if a run gets interrupted.

`just` is a single dependency to install on the host (`brew install just`), and is the only command surface documented for day-to-day use — the underlying `docker compose run ...` invocations stay available as an escape hatch but aren't the primary teaching surface in the README.

## Future Extensions (seams built in now, not implemented)

- **Local image download**: a separate pass that walks existing output `.md` files, downloads referenced images into an `attachments/` folder next to each file, and rewrites links — without re-scraping. Enabled by a `download_images` config flag at the conversion stage (step 6), so it's a config toggle, not an architecture change.
- **Direct `raw/` output**: change `--output-dir` to point at `raw/`; no code changes needed since folder-per-domain structure is already what `raw/` uses today.
- **Structured recipe-schema extraction**: JSON-LD Recipe detection as an optional pre-step for sites that have it, feeding directly into the existing wiki recipe frontmatter — deferred, kept generic for now since recipes are only one of many source types.
- **LLM-based cleanup fallback**: for pages that still get flagged after per-domain rules + generic extraction, an optional (paid, opt-in) pass that sends the flagged content to an LLM for cleanup, rather than requiring a manual fix every time.

## Testing

- Unit tests for: URL normalization/dedup, markdown checkbox-list URL parsing, per-domain rule application, quality-flagging heuristics (using the real Half Baked Harvest file as a fixture — before/after known-junk removal).
- Integration test: a small fixture batch (3-5 saved HTML pages, including the Half Baked Harvest example) run through the full pipeline, asserting output files exist, frontmatter is well-formed, and the known junk patterns are absent from the final Markdown.
