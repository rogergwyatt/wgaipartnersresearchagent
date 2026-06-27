---
name: data-model-reconstruction
description: Reconstruct the domain data model (entities, fields, relationships) for a black-box SaaS app, using DB exports as ground truth and cross-checking against UI forms and API payloads. Use whenever building a data model from database exports or a data dictionary, inferring schema, or mapping entities and relationships of an existing system.
---

# Data Model Reconstruction

DB exports in `inputs/db-exports/` are schema ground truth. Combine with API
payloads (what's active) and UI forms (what's user-facing) to classify every
field. Ambiguous semantics go to SME questions — do not guess business meaning
from column names alone.

## Inputs

| Source | Location | Role |
|--------|----------|------|
| DB exports / schema dumps | `inputs/db-exports/` | Ground truth for entities, fields, types, constraints, indexes |
| API catalog | `outputs/02-observed-api.md` | What fields are actually exchanged at runtime |
| Feature inventory | `outputs/01-feature-inventory.md` | What entities map to user-visible concepts |
| SME documents | `inputs/sme-docs/` | Business semantics, lifecycle rules, soft-delete conventions |

## Output: `outputs/03-data-model.md`

### Entity summary table

```markdown
| Entity | Table(s) | Description | Status |
|--------|----------|-------------|--------|
| Project | projects | Top-level workspace container | active |
| LegacyExport | legacy_exports | Pre-2021 export format | suspected-dead |
```

### Per-entity detail block

```markdown
#### Project (`projects`)

**Description:** Represents a user's top-level workspace. All resources belong to a Project.

**Fields**

| Column | Type | Nullable | UI-facing | API-exchanged | Notes |
|--------|------|----------|-----------|---------------|-------|
| id | uuid | no | no | yes | PK |
| name | varchar(255) | no | yes | yes | |
| owner_id | uuid | no | no | yes | FK → users.id |
| plan_tier | enum | no | no | yes | free\|pro\|enterprise |
| deleted_at | timestamp | yes | no | no | Soft delete — confirm with SME |
| legacy_ref | varchar | yes | no | no | Never seen in API — suspected dead |

**Relationships**
- belongs to `User` (owner_id)
- has many `ProjectMember` (join table: project_memberships)
- has many `Resource`

**Indexes observed:** `(owner_id)`, `(deleted_at)` (partial, where null)

**Open questions:**
- Is `deleted_at` a soft-delete pattern applied consistently, or only on Project?
- What triggers `plan_tier` changes — billing webhook or manual admin action?
```

## Methodology

1. **Start from schema, not API** — enumerate all tables first; then classify each as active / suspected-dead / join-table / audit-log
2. **Active/dead classification**:
   - **active**: field appears in at least one API payload or UI form
   - **suspected-dead**: exists in DB, not in any API payload or UI form observed; flag for SME confirmation before omitting from specs
   - **audit-only**: timestamps, created_by, updated_by — capture but call out as cross-cutting
3. **Relationships**: infer from FK constraints in schema; cross-check by following join patterns visible in API response nesting
4. **Soft-delete detection**: `deleted_at`, `is_deleted`, `status='archived'` — note the pattern and confirm with SME whether it is consistent
5. **Multi-tenancy**: identify the tenant discriminator (org_id, account_id, workspace_id) and note which tables do/don't carry it
6. **Enum values**: extract from schema CHECK constraints or application-level enums if visible in exports; list all observed values

## Trailing sections

- `## Entity Relationship Summary` — a short prose description of the top-level entity graph (3–8 entities) suitable for a README
- `## Suspected-Dead Fields` — consolidated list for SME confirmation before specs are written
- `## Open SME Questions`
