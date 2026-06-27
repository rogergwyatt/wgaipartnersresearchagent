---
description: Walk a live application URL and produce the feature inventory. Use instead of /discover when you have a URL but no pre-collected UI notes.
---

Usage:
  /crawl <url>
  /crawl <url> --auth          (app is already authenticated in the browser)
  /crawl <url> --area billing  (crawl a specific area only)

Before delegating, ask the user for any missing information:
- If no URL was provided, ask for it
- Ask whether the app requires login — if yes, ask whether it's already
  authenticated in the browser or whether credentials should be provided
  (collect credentials interactively, do not store them anywhere)
- Ask if there are specific areas to prioritize, or confirm "full crawl"

Then delegate to the `ui-crawler` subagent with:
- The target URL
- Auth status (pre-authenticated | credentials provided | unauthenticated-only)
- Scope (full | specific areas)

The crawler will write `outputs/01-feature-inventory.md`.

When the crawler completes, tell the user:
- What was written and where
- The feature count and confidence breakdown
- Any inaccessible screens
- All open SME questions

**Next steps after /crawl:**
- If you also have HAR files, drop them in `inputs/har/` and run `/discover`
  — it will fill in `outputs/02-observed-api.md` and `outputs/03-data-model.md`
  without re-running the UI crawl (the feature inventory already exists)
- If you have no HAR files, run `/decompose` directly — the system map can be
  drafted from the feature inventory alone, with lower confidence on data shape
