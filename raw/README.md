# Raw Recipe Source Material

Unprocessed recipe clippings and link lists, gathered as raw input for the [`recipe-wiki`](../recipe-wiki/) project. Clipped primarily via the Obsidian Web Clipper, one source site per subdirectory. This is intentionally messy/unstructured — the structuring work happens downstream when the [`scraper`](../scraper/) and LLM ingest pipeline process it into the wiki.

## Structure

Each subdirectory is named after the source site/blog and contains one Markdown file per clipped recipe:

| Source Directory | Recipes |
|---|---|
| `101 Cooking for Two/` | Baked Parmesan Green Beans |
| `A Family Feast/` | Whoopie Pies |
| `Adventures of a Nurse/` | Better than Take Out Beef and Broccoli |
| `All Recipes/` | Cajun Chicken and Sausage Gumbo; Creole Seasoning Blend |
| `Big Oven/` | Blue Apron Cumin Sichuan Peppercorn Sauce |
| `Bon Appetit/` | Parmesan-Roasted Cauliflower |
| `BS In The Kitchen/` | Caramelized Onion & Mushroom Brie Grilled Cheese |
| `Half Baked Harvest/` | Better Than Takeout Sweet Thai Basil Chicken; Crockpot Creamy Coconut Chicken Tikka Masala |
| `My Fitness Pal/` | Baked Honey Mustard Chicken |
| `Serious Eats/` | Singapore Rice Noodles |

These subdirectories don't have their own READMEs — they're just raw clipped files, not projects.

Additional loose files at this level:

| File | Contents |
|---|---|
| [`pinterest-recipe-links.md`](pinterest-recipe-links.md) | Raw list of recipe links saved from Pinterest, not yet clipped/processed. |
| [`pinterest-recipe-canonical-links.md`](pinterest-recipe-canonical-links.md) | The same Pinterest links resolved to their canonical source URLs (de-linked from Pinterest redirects). |

## Note

The current clipping workflow (manual, one URL at a time via Obsidian Web Clipper) produces Markdown that still contains boilerplate (ad iframes, newsletter banners, etc.) requiring manual cleanup — see [`scraper/html-to-md-scraper-design-spec.md`](../scraper/html-to-md-scraper-design-spec.md) for the planned fix.
