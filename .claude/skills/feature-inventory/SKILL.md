---
name: feature-inventory
description: Catalog every user-facing capability of a black-box SaaS app from a UI walkthrough and screen notes. Use this whenever building or updating a feature inventory of an existing application, mapping what a system does before decomposition, or when the user mentions feature mapping, capability cataloging, or "what does this app do". Always tag each feature with its source(s) and a confidence level.
---

# Feature Inventory

Produce a structured catalog of what the existing system does, walking the UI
screen by screen. This is the starting truth surface — everything discovered later
(API, data model) is checked against it.

## Inputs

| Source | Location | Role |
|--------|----------|------|
| UI walkthrough notes | `inputs/ui-notes/` | Primary — what users see and do |
| HAR captures | `inputs/har/` | Cross-check — confirm features are actually wired |
| SME documents | `inputs/sme-docs/` | Hidden rules, roles, permissions, edge cases |
| DB exports | `inputs/db-exports/` | Signals unused/legacy features via dead columns |

## Output: `outputs/01-feature-inventory.md`

Structure the output as a flat table followed by per-area narrative sections.

### Top-level table

```markdown
| ID | Area | Feature | Description | Sources | Confidence | Open Questions |
|----|------|---------|-------------|---------|------------|----------------|
| F01 | Auth | Email/password login | ... | ui, har | high | |
| F02 | Auth | SSO via Google | ... | ui | low | Is this GA or behind a flag? |
```

- **ID**: sequential, prefix with area abbreviation (`AUTH-01`, `BILL-03`, …)
- **Area**: logical grouping (Auth, Billing, Reporting, Admin, …)
- **Sources**: one or more of `ui` / `har` / `db` / `sme`
- **Confidence**: `high` (triangulated ≥2 sources) / `medium` (single source, credible) / `low` (inferred, unconfirmed)
- **Open Questions**: anything that cannot be confirmed — emit as a question, not a guess

### Per-area narrative

After the table, write one subsection per area:
- Describe the user journey through that area
- Call out non-obvious business rules (e.g., "trial accounts cannot export")
- List all open SME questions for the area

## Methodology

1. **Walk the UI top-down** — nav bar → primary flows → settings → admin panels → edge-case screens
2. **Don't infer from HAR alone** — if a network call exists but no UI surface was observed, mark confidence low and raise a question
3. **Dead features**: if DB has a column `legacy_export_format` with no HAR call and no UI reference, note it as candidate-dead and flag for SME confirmation
4. **Permissions/roles**: capture every distinct role visible in the UI (admin, viewer, billing-only, …) as its own inventory entry under an "Access Control" area
5. **Pagination, sorting, filtering** count as features — list them explicitly

## Confidence escalation

If a feature is `low` confidence and no SME document clarifies it, add it to a
trailing `## Open SME Questions` section at the end of the output with the
question phrased for a client interview: "Does the bulk-export feature apply to
free-tier accounts, or is it billing-gated?"
