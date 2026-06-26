---
description: Run black-box discovery — produce the feature inventory, observed API surface, and data model from inputs/.
---

Delegate to the `discovery` subagent.

Before delegating, check that inputs exist:
- `inputs/ui-notes/` — at least one file (UI walkthrough notes)
- `inputs/har/` — at least one .har file
- `inputs/db-exports/` — at least one schema or export file

If any input directory is empty, warn the user that the corresponding output will
be thin and confidence will be low, then proceed with what's available.

The discovery agent should:
1. Read everything in `inputs/`
2. Write `outputs/01-feature-inventory.md`
3. Write `outputs/02-observed-api.md`
4. Write `outputs/03-data-model.md`

When the agent completes, summarize to the user:
- Feature count and confidence breakdown (high / medium / low)
- Endpoint count
- Entity count and how many are suspected-dead
- All open SME questions, numbered, ready to paste into a client interview doc

Usage: `/discover`
