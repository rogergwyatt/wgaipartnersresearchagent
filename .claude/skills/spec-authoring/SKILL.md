---
name: spec-authoring
description: Write an infrastructure-agnostic build spec for a single module, ready for Claude Code to implement. Use whenever turning a system-map module into a spec, writing module or feature specifications, or preparing specs for a build. Never name a concrete database or cloud — reference logical capabilities only.
---

# Spec Authoring

Write one spec per module using `assets/spec-template.md`. Every spec must be
self-contained: a developer (or Claude Code) reading only the spec and the
infrastructure profile should be able to build the module without other context.

## Inputs

| Source | Required |
|--------|---------|
| `outputs/04-system-map.md` — the target module's entry | yes |
| `outputs/01-feature-inventory.md` — features assigned to this module | yes |
| `outputs/02-observed-api.md` — endpoints assigned to this module | yes |
| `outputs/03-data-model.md` — entities owned by this module | yes |
| `infrastructure-profile.yaml` | reference only — do NOT copy values into spec |

## Output: `outputs/specs/<module-id>-<module-name>.spec.md`

Use the canonical format in `assets/spec-template.md`. All sections are required;
write "none" rather than omitting a section.

## The non-negotiable rule

**No concrete technology may appear in a spec.** Use the logical capability names
defined in `infrastructure-profile.yaml`:

| Spec says | Profile maps to |
|-----------|-----------------|
| relational store | postgres, mysql, sqlserver, … |
| object storage | S3, GCS, Azure Blob, … |
| managed queue | SQS, Pub/Sub, Service Bus, … |
| cache | ElastiCache, Memorystore, Redis Cloud, … |
| identity provider | Cognito, Auth0, Entra ID, Keycloak, … |
| secret store | Secrets Manager, Secret Manager, Key Vault, … |

If a spec section needs to mention where data lives, say "relational store" — not
"Postgres table". This is what lets one spec set build on AWS or GCP without edits.

## Methodology

1. **Start from the system-map entry** — confirm scope (owned entities, assigned API prefix, tier, dependencies)
2. **Enumerate endpoints from the API catalog** — each observed endpoint assigned to this module becomes a required API contract; include request/response shapes verbatim (minus PII)
3. **Define the schema from the data model** — only entities owned by this module; use logical types (`uuid`, `text`, `boolean`, `timestamp`, `enum(…)`, `json`)
4. **Business rules first, implementation second** — capture the *what*, not the *how*; leave implementation decisions to the build step
5. **Cross-module calls** — describe integration points as "calls the Identity module's token validation endpoint" — not as a specific URL or SDK call
6. **Error handling** — for each API endpoint, document the error cases observed (from HAR) and the expected response shape
7. **Non-functional requirements** — note if a module requires low-latency (cache hint), high-throughput (queue hint), or large file storage (object storage hint); the builder picks the concrete binding

## Spec completeness checklist

Before finalizing a spec:
- [ ] Every feature assigned to this module in the inventory is covered
- [ ] Every API endpoint assigned to this module has a contract block
- [ ] Every owned entity has a schema block
- [ ] The infra-binding section lists only logical capabilities, no concrete tech
- [ ] Cross-module dependencies are listed in the Dependencies section
- [ ] No open questions remain — all SME questions resolved or escalated
