# Module Spec: {{MODULE_NAME}} ({{MODULE_ID}})

> **Infrastructure-agnostic.** This spec names logical capabilities only.
> Concrete bindings (database engine, cloud provider, etc.) are in
> `infrastructure-profile.yaml` and resolved at build time.

---

## 1. Overview

**Module ID:** {{MODULE_ID}}  
**Tier:** {{leaf | mid | core}}  
**Depends on:** {{list of module IDs, or "none"}}  
**Depended on by:** {{list of module IDs, or "none"}}

One-paragraph description of what this module does and why it exists as a
distinct bounded context.

---

## 2. Features

List every feature from `outputs/01-feature-inventory.md` assigned to this
module. For each feature:

```
### {{FEATURE_ID}} — {{Feature Name}}

**Description:** What the user can do.

**Business rules:**
- Rule 1 (e.g., "A project must have exactly one owner at all times")
- Rule 2

**Roles/permissions:** Which roles can invoke this feature.

**Edge cases / constraints:** Any non-obvious behavior observed.
```

---

## 3. API Contracts

List every endpoint from `outputs/02-observed-api.md` assigned to this module.

```
### {{ENDPOINT_ID}} {{METHOD}} {{PATH}}

**Purpose:** One sentence.

**Auth:** {{required | none | optional}} — {{mechanism, e.g., "Bearer token validated by Identity module"}}

**Request**

Headers:
- `Authorization: Bearer <token>` (when auth required)

Query params:
- `page` (integer, default 1)
- `per_page` (integer, default 25, max 100)

Body (JSON):
```json
{
  "name": "string (required, max 255)",
  "description": "string (optional)"
}
```

**Response 200**
```json
{
  "id": "uuid",
  "name": "string",
  "created_at": "iso8601"
}
```

**Error responses**
| Status | Condition | Body shape |
|--------|-----------|------------|
| 400 | Validation failure | `{"errors": [{"field": "name", "message": "..."}]}` |
| 401 | Missing or invalid token | `{"error": "unauthorized"}` |
| 403 | Insufficient role | `{"error": "forbidden"}` |
| 404 | Resource not found | `{"error": "not_found"}` |
| 422 | Business rule violation | `{"error": "...", "detail": "..."}` |
```

---

## 4. Data Schema

List every entity owned by this module from `outputs/03-data-model.md`.
Use logical types — never a concrete database type.

```
### {{EntityName}}

**Store:** relational store  
**Description:** One sentence on what this entity represents.

| Field | Type | Nullable | Notes |
|-------|------|----------|-------|
| id | uuid | no | Primary key |
| name | text | no | Max 255 chars |
| owner_id | uuid | no | FK → Identity module user |
| plan_tier | enum(free, pro, enterprise) | no | |
| deleted_at | timestamp | yes | Soft-delete marker |
| created_at | timestamp | no | Set on insert, never updated |
| updated_at | timestamp | no | Updated on every write |

**Constraints:**
- `name` must be unique within the same owning account
- `deleted_at` IS NULL filter applied to all list queries

**Indexes required:**
- `(owner_id)` — supports list-by-owner queries
- `(deleted_at)` partial where null — supports soft-delete filtering
```

---

## 5. Infrastructure Binding

List only the logical capabilities this module requires. No concrete names.

```yaml
requires:
  - relational store       # entity persistence
  # - object storage       # uncomment if module handles file uploads
  # - managed queue        # uncomment if module emits async events
  # - cache                # uncomment if low-latency reads required
  - identity provider      # token validation (via Identity module, not direct)
  # - secret store         # uncomment if module manages credentials
```

---

## 6. Dependencies

Cross-module interactions this module initiates.

```
| Dependency | Type | What is called / emitted | When |
|------------|------|--------------------------|------|
| Identity module | sync call | Token validation endpoint | On every authenticated request |
| Notifications module | async event | `project.created` event | When a project is created |
```

---

## 7. Non-functional Requirements

Only call out requirements that are non-obvious or constrain the build.

- **Throughput:** {{e.g., "List endpoint must return within 200ms at p99 for up to 10,000 records — cache hint"}}
- **Data retention:** {{e.g., "Soft-deleted records retained for 90 days then hard-deleted"}}
- **Idempotency:** {{e.g., "POST /resources is idempotent on client-supplied idempotency-key header"}}
- **Rate limiting:** {{e.g., "Observed 429 responses at 100 req/min per token — enforce same limit"}}

---

## 8. Open Questions

Any gap that must be resolved before build begins. Format for a client interview.

| # | Question | Context | Blocking? |
|---|----------|---------|-----------|
| 1 | {{Question text}} | {{Why it matters}} | {{yes / no}} |

If none: **none — spec is complete for build.**
