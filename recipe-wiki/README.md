# Recipe Wiki

A local-first, AI-queryable personal culinary knowledge base — not a recipe app, but a Karpathy-style LLM wiki for your kitchen. Raw source documents (see [`raw/`](../raw/)) get "compiled" by an LLM into organized Markdown files capturing not just recipes but the *why* behind techniques (in the style of J. Kenji López-Alt's writing), making the whole collection queryable ("I have chicken thighs, lemons, and capers — what should I make?").

## Files

| File | Contents |
|---|---|
| [`recipe-wiki.md`](recipe-wiki.md) | Full design doc: what the system is, the architecture (raw sources → LLM compilation → queryable wiki), and example queries it should support. |
