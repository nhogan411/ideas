# Personal Recipe & Culinary Knowledge Base
### A Karpathy-Style LLM Wiki for Your Kitchen

---

## What This Is

A **local-first, AI-queryable culinary knowledge base** — not a recipe app, but a personal wiki that accumulates everything you know and have saved about cooking. Inspired by Andrej Karpathy's LLM wiki concept: raw source documents get "compiled" by an LLM into organized Markdown files, and those files become the knowledge base you query.

The key insight is that a recipe by itself is just instructions. What makes cooking better is the *why* behind the instructions — the kind of context J. Kenji López-Alt (yes, that's the spelling) embeds throughout his writing on Serious Eats and in *The Food Lab*. This system captures both together.

**You'll be able to ask things like:**
- "I have chicken thighs, lemons, and capers. What should I make?"
- "Why does Kenji say to flip a steak every 30 seconds? What goes wrong if I don't?"
- "What's the difference between how I should cook a ribeye vs a NY strip?"
- "Give me the best technique for caramelizing onions without burning them"

---

## Architecture Overview

```
Sources (raw/)
├── bookmarks/         ← Web-clipped Markdown from browser bookmarks
├── pinterest/         ← Scraped or manually exported pins
├── keep/              ← Google Keep notes exported via Takeout
├── cookbooks/         ← Transcribed or OCR'd cookbook recipes
└── serious-eats/      ← Clipped Kenji articles + technique guides

          ↓  (ingest script + Claude API)

Wiki (wiki/)
├── index.md           ← Auto-maintained master index
├── recipes/           ← Individual recipe files
│   ├── ribeye-steak.md
│   ├── roast-chicken.md
│   └── ...
├── techniques/        ← Technique articles (reverse sear, basting, etc.)
│   ├── searing-meat.md
│   ├── caramelizing-onions.md
│   └── ...
├── ingredients/       ← Ingredient notes (how to shop, store, prep)
│   ├── chicken-thighs.md
│   └── ...
└── sources/           ← Attribution tracking (cookbook citations, URLs)

          ↓  (query via Obsidian + Claude)

You, asking questions
```

**Key principle:** You almost never edit the `wiki/` files directly. The LLM writes them. You interact with the `raw/` folder (adding new content) and the query interface (asking questions).

---

## Components

### 1. Obsidian (Free)
Your "IDE" for the knowledge base. Renders Markdown beautifully, handles wikilinks between pages, and has a plugin ecosystem. You'll install:
- **[Claudian](https://community.obsidian.md/plugins/realclaudian)** — chat directly with your wiki using Claude
- **[Obsidian Web Clipper](https://obsidian.md/clipper)** — browser extension that clips any webpage to Markdown with one click

### 2. A Git Repo
Your Obsidian vault is just a folder of `.md` files. Version-control it with git. This gives you:
- Full history of how your wiki evolved
- Easy backup to GitHub (private repo)
- Ability to rollback if the LLM writes something wrong

### 3. Ingest Scripts
A small collection of shell/Python scripts (you vibe-code these with Claude) that:
- Watch the `raw/` folder for new files
- Call the Claude API to process new content into wiki pages
- Update the master `index.md` and backlinks

### 4. Claude API
Used for two things:
- **Ingestion**: Processing new raw content into structured wiki pages
- **Querying**: Answering your questions using the wiki as context

You can also use OpenAI (GPT-4.1 / GPT-4o), but Claude is currently better at maintaining consistent Markdown style and longer document context.

---

## Data Ingestion — How You Add Content

The model here is simple: **you decide what goes in, the system decides how to organize it.** There's no scraping, no automation of the import itself. You're the curator. The LLM is the librarian.

### The Core Workflow

1. You drop a file into `raw/` — any way you like
2. The ingest script detects the new file, calls Claude, and compiles/updates wiki pages
3. You never touch `wiki/` directly

That's it. The "intelligence" is entirely on the output side.

### How to Get Content Into `raw/`

**From the web (recipes, Serious Eats articles, etc.):**
Install the **[Obsidian Web Clipper](https://obsidian.md/clipper)** browser extension. When you're on a recipe or Kenji article you want to save, click the clipper → it converts the page to clean Markdown and saves it directly to your `raw/` folder. One click per page. This is your primary tool.

**From Pinterest / Google Keep:**
Open the pin or Keep note, follow the link to the actual recipe, and Web Clip it. Pinterest saves from years ago are totally valid candidates — you don't need to be precious about what goes in. The status flags (see below) are what determine whether the system recommends something, not whether it exists in the wiki.

**From physical cookbooks:**
- **Typing it out**: 5-10 minutes per recipe. Add a citation line at the top: `source: "The Food Lab by J. Kenji López-Alt, p. 312"`. Include his technique notes — that's where the value is, not just the ingredient list.
- **Photographing the page**: Drop the image into `raw/`. The ingest script can call Claude's vision API to transcribe it before compiling. Works well for clean cookbook typography.

**From articles you already have open:**
Web Clipper handles this. For Kenji content specifically — his technique articles on Serious Eats ("The Science of Perfect Steak," etc.) are worth clipping in full, not just bookmarking. The commentary is the content.

## Recipe Status Flags

Each recipe page in `wiki/recipes/` gets a small YAML frontmatter block at the top. The three fields that drive recommendations:

```yaml
---
title: "Pan-Seared Ribeye"
cuisine: asian, japanese
status: made          # options: unproven | made | retired
enjoyed: true         # true | false | null (null = made but no verdict yet)
last_made: 2025-11-12
source: "https://www.seriouseats.com/..."
servings: 4
servings_verified: true     # did it actually yield this many?
prep_time: 15         # minutes
cook_time: 20         # minutes
total_time: 35        # minutes — can differ from prep+cook (e.g. rest time, marinating)
time_verified: true         # did the timing seem accurate when you made it?
calories_per_serving: 540
calories_confidence: calculated   # estimated | calculated | verified
protein_per_serving: 48   # grams
carbs_per_serving: 12
fat_per_serving: 22
---
```

### A Note on Calorie and Macro Data

This is worth being honest about: **there is no frictionless path to accurate macros.** Here's the realistic picture:

**Where the numbers come from (in order of accuracy):**

1. **`verified`** — you logged it in MyFitnessPal with your specific ingredients and copied the numbers over. Most accurate, highest friction. Worth doing for recipes you make frequently.

2. **`calculated`** — the ingest script parses the ingredient list and queries the **[USDA FoodData Central API](https://fdc.nal.usda.gov/)** (free, no key required for basic use). It maps "1 lb boneless chicken breast" → USDA nutrition data → calculates per-serving totals. Better than website estimates because it's based on the actual ingredient quantities in the recipe, not what the blogger guessed. Still imperfect — doesn't account for oil absorbed during cooking, water loss, etc.

3. **`estimated`** — Claude makes a reasonable guess when ingredient quantities are too vague for the USDA API to calculate reliably. Ballpark only.

The ingest script always tags macros with whichever confidence level applies. When you query, the system can filter by confidence:

> "Show me low-carb dinners — only ones with verified or calculated macros"

**The "200 calories left for a snack" use case** is worth naming separately: this system can answer "show me snacks under 200 calories" but it doesn't track what you've eaten today — that still lives in MyFitnessPal or wherever. The two tools complement each other rather than one replacing the other. What this system gives you is a searchable library of *your* recipes with macro estimates, so when you're in MFP you can find the right recipe faster.

**Practical recommendation:** Don't try to verify macros upfront. Cook the recipe, and if you happen to log it in MFP anyway, copy the numbers back into the frontmatter. Over time your most-cooked recipes accumulate verified data naturally, with zero extra effort.

```yaml
calories_confidence: estimated   # options: estimated | calculated | verified
```

### Status Values

| Value | Meaning | Recommended by default? |
|-------|---------|------------------------|
| `unproven` | Saved but never cooked | ❌ No — only shown if user asks for new ideas |
| `made` | You've cooked it at least once | Depends on `enjoyed` |
| `retired` | Made it, actively don't want to make it again | ❌ Never |

### The `enjoyed` Field

| Value | Meaning | Recommended? |
|-------|---------|-------------|
| `true` | Would make again | ✅ Yes — first priority |
| `null` | Made it, no strong verdict | ⚠️ Maybe — shown as "you've made this before" |
| `false` | Made it, wouldn't repeat | ❌ No |

### Default Behavior

When you ask for a recommendation, the system:
1. **First** surfaces `status: made` + `enjoyed: true` recipes that match your ingredients
2. **Then** offers `status: made` + `enjoyed: null` as secondary options ("you've made this, no verdict")
3. **Only if you ask** ("show me something new" or "I want to try something different") will it surface `unproven` recipes

You update these flags yourself — but the system needs to make this as frictionless as possible or it won't happen.

### Morning Prompt (via launchd cron)

A `launchd` job runs at ~8am each morning and opens a simple CLI prompt (or minimal local web page) asking what you cooked the night before:

```
Good morning. Did you cook anything last night?

> sesame chicken

Found: "Sesame Ginger Chicken Stir-fry" — is that the one? (y/n)
> y

How was it?
  1. Loved it — would make again
  2. Fine, no strong feelings
  3. Didn't love it
> 1

Recipe says: 4 servings, 35 min total. Seem about right?
  1. Yes, both were accurate
  2. Servings were off (it made more / less)
  3. Timing was off (took longer / shorter)
  4. Both were off
> 3

Timing was off — roughly how long did it actually take?
> closer to 50 minutes

Logged. Status: made ✓  Enjoyed: true ✓  Time updated: ~50 min ✓
```

Fuzzy-matches the recipe from whatever you type — no need for exact titles. The whole interaction is ~15 seconds while the coffee brews. If you didn't cook, you just close the terminal. `launchd` ensures it reliably fires each morning without you having to remember.

The ingest script never overwrites `status` or `enjoyed` once you've set them. It only writes the defaults (`unproven`, `null`) on first ingest.

---

## Ingredient-Based Recommendations

The query interface is designed to handle natural-language ingredient questions with smart gap detection. You say what you have; it tells you what you can make and flags what you're missing.

### Example Interaction

**You ask:**
> "Looking for something Asian to make tonight. I have a pound of chicken breast, soy sauce, garlic, ginger, and sesame oil."

**The system responds:**

> **With what you have:**
> - **Sesame Ginger Chicken Stir-fry** *(made it, enjoyed it)* — you have everything
> - **Soy-Glazed Chicken Thighs** *(made it, enjoyed it)* — calls for thighs, but breast will work with minor time adjustment
>
> **If you have an onion:**
> - **Chicken Teriyaki Bowl** *(unproven, saved from Serious Eats)* — also needs mirin; you could sub with a splash of rice wine + sugar
>
> **Missing one or two things:**
> - **Dan Dan Noodles** — you'd need tahini (or peanut butter) and noodles
> - **Mapo Tofu** — you'd need tofu and doubanjiang

This works because:
1. Every recipe page has a structured ingredient list the LLM can parse
2. The query prompt instructs Claude to: match on what's provided, identify close matches, flag what's missing, and respect status flags
3. Claude is quite good at ingredient substitution reasoning ("thighs vs breast," "mirin sub")

### How "Gap Detection" Gets Prompted

The key is in how you prompt the query. The ingest script writes a system prompt that includes something like:

> *"When recommending recipes, first prioritize those marked enjoyed: true. For each candidate, identify ingredients the user mentioned vs. ingredients required. Flag recipes as: 'you have everything,' 'missing X (easy sub: Y),' or 'missing X (required).' Suggest substitutions where reasonable. Only surface unproven recipes if the user explicitly asks for new ideas or nothing in the proven set matches."*

You don't have to think about this — it's baked into the system prompt once during setup.

---

When the ingest script processes a new raw file, it instructs Claude to:

1. **Create or update a recipe page** in `wiki/recipes/` with:
   - Standardized frontmatter (title, cuisine, servings, time, source URL — plus `status: unproven` and `enjoyed: null` as defaults; the ingest script never overwrites these once you've set them)
   - Structured ingredients list (each ingredient on its own line so the query system can match against them)
   - Instructions
   - Chef's notes / technique tips (extracted from the source)
   - Backlinks to relevant technique pages
   - Source citation with URL

2. **Create or update technique pages** in `wiki/techniques/` that get enriched each time a new source mentions the technique. Over time, `wiki/techniques/searing-meat.md` becomes a rich document that synthesizes everything Kenji, Marcus Samuelsson, and whoever else you've saved has said about searing.

3. **Create or update ingredient pages** with shopping notes, storage tips, and links to recipes using that ingredient.

4. **Update `index.md`** with a brief summary of the new content and links.

---

## Query Interface

Two modes:

### Mode 1: Conversational (via Claudian in Obsidian)
Open Obsidian, open the Claudian panel, ask:
> "I have chicken thighs, a lemon, garlic, and capers. What should I make, and what technique tips should I know?"

Claude reads the relevant wiki pages and synthesizes an answer — citing your actual saved recipes and the technique notes from Kenji or wherever you clipped from.

### Mode 2: CLI (for power use)
A simple shell command that sends your question + relevant wiki context to the Claude API and prints the answer. Useful when you're cooking and just want to type a quick question in Terminal.

```bash
recipe "how do I know when the maillard reaction is actually happening on my steak?"
```

---

## Setup Steps (One-Time)

1. **Install Obsidian** → create a new vault at `~/recipes-wiki/`
2. **Initialize git** in that folder, create private GitHub repo
3. **Create folder structure**: `raw/`, `wiki/`, `.scripts/`
4. **Install Obsidian plugins**: Web Clipper (browser extension), Claudian (Obsidian plugin)
5. **Get Claude API key** from [console.anthropic.com](https://console.anthropic.com) → add a small credit ($20 gets you started)
6. **Write/vibe-code the ingest script** (1-2 hours with Claude's help):
   - Watches `raw/` for new `.md` or image files
   - For images: calls Claude vision API to transcribe first
   - Calls Claude to compile content into wiki pages
   - Writes/updates files in `wiki/`, maintains `index.md`
7. **Seed the wiki**: Clip your 20-30 most-used recipes via Web Clipper, run the ingest script, watch the wiki populate
8. **Test queries** with Claudian to verify the wiki is useful

---

## Costs

### Initial Setup

| Item | Cost |
|------|------|
| Obsidian | Free |
| Obsidian plugins (Claudian, Web Clipper) | Free |
| GitHub private repo | Free |
| Claude API initial credit | $20 (one-time, but won't be all spent at setup) |
| Your time — setup + scripts | ~4-8 hours |
| Your time — initial seeding | ~4-8 hours (depending on how many sources you have) |

**Rough API cost for initial ingest:**
- 300 recipes clipped and processed: ~$8-12 in Claude API calls
- 50 Kenji/Serious Eats technique articles: ~$3-5
- Total initial API spend: **~$15-20** (fits within your initial $20 credit)

### Ongoing Costs

| Item | Cost | Notes |
|------|------|-------|
| Claude API — queries | ~$2-5/month | Assuming 5-15 queries/week, moderate context window |
| Claude API — new ingests | ~$1-2/month | Adding a handful of new recipes/articles per week |
| GitHub | Free | Private repos are free |

**Realistic ongoing cost: $3-8/month** for regular, active use.

**Cost levers:**
- Use `claude-3-5-haiku` for ingestion (cheaper, still excellent at structured Markdown tasks) → cuts ingestion cost ~80%
- Use `claude-opus-4` or `claude-sonnet-4` only for complex synthesis queries → saves money where quality matters less
- Cache the wiki index so repeated queries don't re-read the whole thing

---

## What Makes This Better Than Existing Recipe Apps

| Feature | Paprika / NYT Cooking | This |
|---------|----------------------|------|
| Works offline | ✅ | ✅ |
| Survives site going down | ❌ | ✅ (local files) |
| Ingredient-based search | Basic keyword | Semantic ("I have X, Y, Z") |
| Technique synthesis | ❌ | ✅ (cross-source) |
| "Why" behind instructions | ❌ | ✅ (from Kenji etc.) |
| Your own notes + sources | Limited | ✅ (you own the files) |
| Troubleshooting ("what went wrong") | ❌ | ✅ |
| Cite source cookbooks | ❌ | ✅ |
| Version history | ❌ | ✅ (git) |

---

## Realistic Limitations

- **No mobile app out of the box.** You'd need Obsidian Sync ($4/mo) to get the wiki on your phone. Querying from mobile is clunky unless you build a simple web UI later.
- **Maintenance overhead.** Like Casey Newton found with his journalism wiki, pages can get long and occasionally need compaction. Budget 30 min/month for "wiki health checks."
- **OCR quality varies.** Photographing cookbook pages works reasonably well but occasionally produces errors in ingredient amounts. Always sanity-check transcribed recipes the first time you cook from them.
- **Pinterest is the messiest source.** Don't expect to import your entire Pinterest history cleanly. Be selective.
- **Claude hallucination risk in queries.** Always ask it to cite its sources, and spot-check by opening the linked wiki page. The wiki grounds it significantly, but it can still confabulate.

---

## Natural Evolutions (Later)

Once the core is working:
- **Meal planning agent**: "Plan me 5 dinners for next week using what's in my fridge"
- **Shopping list generation**: From a planned meal, output a grocery list
- **Simple web UI**: A lightweight local Flask/Next.js app so you can query from your phone while cooking
- **Finetune a small model**: Once your wiki is large enough (~1,000+ recipes + tips), you could fine-tune a small local model (Mistral, Llama) to "know" your recipes — eliminating ongoing API costs for queries

---

## Getting Started This Weekend

If you wanted to begin this weekend, the highest-ROI path:

1. **Day 1 (2 hrs):** Install Obsidian, install Web Clipper browser extension, create vault at `~/recipes-wiki/`, init git
2. **Day 1 (1-2 hrs):** Clip your 20 favorite recipes and 3-4 Kenji technique articles using Web Clipper → they land in `raw/`
3. **Day 2 (2-3 hrs):** Ask Claude to help you write the ingest script. Give it this document as context. The script should: watch `raw/` for new files, call Claude API to compile into `wiki/`, maintain `index.md`
4. **Day 2 (1 hr):** Run first ingest, watch the wiki populate, test a query via Claudian

You don't need Pinterest, cookbook photos, or anything else to get value on day 2. Start with the 20 recipes you actually cook and let the system prove itself before adding more.

---

*Document drafted: August 2026*
*Based on: Karpathy LLM wiki concept (April 2026) + Casey Newton's Platformer write-up (August 2026)*
