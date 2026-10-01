# HTML-to-Markdown Scraper

A general-purpose tool that takes a batch of URLs (from a `.csv`, `.xlsx`, or `.md` file) and produces clean, review-ready Markdown files on disk — a scriptable, batch-driven replacement for the Obsidian Web Clipper's one-URL-at-a-time "clip this page" workflow currently used to populate [`raw/`](../raw/).

Deliberately generic: recipe sites are just one of many source types it needs to handle; any recipe-specific structuring happens downstream in the [`recipe-wiki`](../recipe-wiki/) LLM ingest step, not in the scraper itself.

## Files

| File | Contents |
|---|---|
| [`html-to-md-scraper-design-spec.md`](html-to-md-scraper-design-spec.md) | Design spec: purpose, problem statement (current clipper output leaves boilerplate like ad iframes and newsletter banners in the Markdown), and the planned approach to stripping junk while preserving legitimate embedded content. |
