---
description: Run black-box discovery — produce the feature inventory, observed API surface, and data model from inputs/. Skips the UI crawl if outputs/01-feature-inventory.md already exists (e.g. from /crawl).
---

Delegate to the `discovery` subagent.

**Before delegating, check what already exists:**

- If `outputs/01-feature-inventory.md` exists (written by `/crawl`), tell the
  discovery agent to skip the feature-inventory step and use that file as-is.
- Check that the remaining inputs exist:
  - `inputs/har/` — at least one .har file → needed for `02-observed-api.md`
  - `inputs/db-exports/` — at least one schema or export file → needed for `03-data-model.md`
- If either directory is empty, warn the user that the corresponding output will
  be thin and confidence will be low, then proceed with what's available.

The discovery agent should write whichever of these don't already exist:
1. `outputs/01-feature-inventory.md` (skip if already present)
2. `outputs/02-observed-api.md`
3. `outputs/03-data-model.md`

When the agent completes, summarize to the user:
- Which outputs were written vs. already present
- Feature count and confidence breakdown (high / medium / low)
- Endpoint count
- Entity count and how many are suspected-dead
- All open SME questions, numbered, ready to paste into a client interview doc

Usage: `/discover`
