---
description: Build the system map and strangler cut order from the discovery outputs.
---

Delegate to the `architect` subagent.

Before delegating, verify that all three discovery outputs exist:
- `outputs/01-feature-inventory.md`
- `outputs/02-observed-api.md`
- `outputs/03-data-model.md`

If any are missing, tell the user to run `/discover` first.

The architect agent should:
1. Read outputs 01–03 (and any SME docs in `inputs/sme-docs/`)
2. Write `outputs/04-system-map.md` with:
   - Module registry (bounded contexts, owned entities, API prefix, tier)
   - Dependency graph
   - Integration points
   - Strangler cut sequence (phased, leaf-first)
   - Facade/router design
   - Anti-corruption layer plan

When the agent completes, present to the user:
- The proposed module list with tiers
- The phased cut order
- Any architectural decisions that carry uncertainty and should be reviewed
  before proceeding to `/spec`

Usage: `/decompose`
