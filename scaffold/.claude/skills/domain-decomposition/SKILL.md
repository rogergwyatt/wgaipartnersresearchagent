---
name: domain-decomposition
description: Group an application's capabilities into bounded contexts/modules, build the dependency graph, and produce a strangler-fig cut order. Use whenever decomposing a system into modules, defining bounded contexts, planning a migration sequence, or designing the facade/router for incremental replacement. Re-draw clean boundaries rather than mirroring legacy structure.
---

# Domain Decomposition

Turn the feature inventory + data model + API surface into a system map with
clean bounded contexts. The goal is *intended* structure, not a mirror of legacy
table layout or URL paths.

## Inputs

| Source | Location | Required |
|--------|----------|---------|
| Feature inventory | `outputs/01-feature-inventory.md` | yes |
| API catalog | `outputs/02-observed-api.md` | yes |
| Data model | `outputs/03-data-model.md` | yes |
| SME documents | `inputs/sme-docs/` | recommended |

## Output: `outputs/04-system-map.md`

### Module registry

```markdown
| ID | Module | Description | Owns Entities | Exposes API Prefix | Tier |
|----|--------|-------------|---------------|-------------------|------|
| M01 | Identity | Auth, users, sessions, roles | User, Session, Role | /auth, /users | leaf |
| M02 | Billing | Plans, subscriptions, invoices | Plan, Subscription, Invoice | /billing | leaf |
| M03 | Projects | Workspaces, membership, settings | Project, ProjectMember | /projects | mid |
| M04 | Resources | Core domain objects, versioning | Resource, Version | /resources | mid |
| M05 | Notifications | Email, in-app, webhook dispatch | Notification, Webhook | /notifications | leaf |
| M06 | Admin | Impersonation, flags, audit log | AuditLog | /admin | leaf |
```

**Tier**: `leaf` (no downstream module dependencies), `mid` (depends on leaves), `core` (depended on by many)

### Dependency graph

```markdown
## Dependencies

M03 Projects → M01 Identity (owner_id, membership auth check)
M04 Resources → M03 Projects (resource belongs to project)
M04 Resources → M01 Identity (created_by)
M05 Notifications → M01 Identity (send to user)
M05 Notifications → M04 Resources (event triggers)
M06 Admin → all (cross-cutting)
```

Render as adjacency list; optionally include a Mermaid diagram.

### Integration points

For each cross-module dependency: what data flows, what event or call triggers it,
and whether it's synchronous (API call) or async (event/queue).

### Strangler cut order

```markdown
## Strangler Sequence

Phase 1 (leaves, no outbound dependencies):
  1. M01 Identity
  2. M02 Billing
  3. M05 Notifications
  4. M06 Admin

Phase 2 (mid-tier, depends only on Phase 1):
  5. M03 Projects
  6. M04 Resources   ← largest module, cut last within phase

Facade/router: intercept all requests at the reverse proxy layer. Route each
path prefix to new module once live; fall through to legacy for uncut paths.

Anti-corruption layer: during transition, M03 Projects must read Project ownership
from the legacy system until M01 Identity is fully cut. ACL adapter in M03 wraps
the legacy auth check and is deleted after cutover.
```

## Methodology

1. **Start from domain nouns, not URLs** — group entities by which business concept owns their lifecycle
2. **One entity, one owner** — if two modules both write the same entity, that is a boundary violation; resolve by ownership assignment or by splitting the entity
3. **Minimize cross-module synchronous calls** — prefer events for side effects; flag unavoidable sync calls as integration points
4. **Leaf-first cut order** — replace modules with no outbound runtime dependencies first; this limits blast radius of each cutover
5. **Don't replicate URL structure** — legacy `/api/v2/foo/bar` paths are an artifact; define module boundaries from domain logic
6. **Admin and audit are cross-cutting** — treat them as a leaf module with read access to other modules' data, not as a dependency of those modules
7. **Feature flags / entitlements** belong in Identity or Billing, not scattered across domain modules

## Validation checklist

Before finalizing the system map:
- [ ] Every entity in `03-data-model.md` is owned by exactly one module
- [ ] Every feature in `01-feature-inventory.md` is covered by exactly one module
- [ ] Every API endpoint in `02-observed-api.md` is assigned to a module
- [ ] No circular dependencies in the dependency graph
- [ ] Cut order respects dependency ordering (no module cut before its dependencies)
