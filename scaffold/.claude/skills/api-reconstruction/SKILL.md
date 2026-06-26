---
name: api-reconstruction
description: Reconstruct a clean API contract catalog from captured network traffic (HAR) for a black-box SaaS app. Use whenever analyzing HAR or network captures, inferring endpoints and request/response shapes, or documenting the observed API surface of an app whose source is unavailable. Always redact tokens and PII.
---

# API Reconstruction

Parse HAR captures from `inputs/har/` into a clean endpoint catalog. The HAR is
ground truth for what the API actually does at runtime — treat it as the contract,
not the UI or docs.

## Inputs

| Source | Location | Role |
|--------|----------|------|
| HAR captures | `inputs/har/` | Primary — observed network calls |
| Feature inventory | `outputs/01-feature-inventory.md` | Map endpoints to features |
| SME documents | `inputs/sme-docs/` | Clarify undocumented semantics |

## Output: `outputs/02-observed-api.md`

### Endpoint catalog table

```markdown
| ID | Method | Path | Purpose | Auth | Features |
|----|--------|------|---------|------|----------|
| E01 | POST | /api/v2/sessions | Create session (login) | none | AUTH-01 |
| E02 | GET | /api/v2/projects | List projects | Bearer | PROJ-01 |
```

### Per-endpoint detail block

For every entry in the table, write an expandable detail section:

```markdown
#### E02 GET /api/v2/projects

**Purpose:** Returns the authenticated user's project list, paginated.

**Request**
- Headers: `Authorization: Bearer <token>` (redacted in captures)
- Query params: `page` (int, default 1), `per_page` (int, default 25), `sort` (name|updated_at)

**Response 200**
```json
{
  "data": [{ "id": "uuid", "name": "string", "created_at": "iso8601", "owner_id": "uuid" }],
  "meta": { "total": 142, "page": 1, "per_page": 25 }
}
```

**Error shapes observed:** 401 `{"error":"unauthorized"}`, 422 `{"errors":[...]}`

**Auth mechanism:** Bearer token in Authorization header  
**Pagination:** cursor/page-based (page + per_page)  
**Features backed:** PROJ-01 (project list), PROJ-05 (project search via `q=` param)  
**Confidence:** high (observed in 3 HAR captures across 2 sessions)  
**Open questions:** none
```

## Methodology

1. **De-duplicate** — multiple HAR files may contain repeated calls; canonicalize to one entry per (method, path-pattern)
2. **Path parameterization** — `/api/projects/abc123` and `/api/projects/xyz789` are the same endpoint; normalize to `/api/projects/{id}`
3. **Redact before writing** — replace tokens, session cookies, email addresses, and UUIDs that look like user IDs with `<redacted>` in all example payloads; never copy raw values
4. **Infer purpose from context** — cross-reference feature inventory; if you cannot map an endpoint to a feature, raise an open question
5. **Auth patterns**: detect Bearer, cookie session, API key header, CSRF token; note which endpoints skip auth (public routes)
6. **Versioning**: if multiple API versions appear (v1, v2), document both and flag deprecated calls
7. **WebSocket / SSE**: note real-time channels separately at the end of the document

## Trailing sections

After the catalog, append:

- `## Unmatched Endpoints` — calls observed in HAR with no corresponding UI feature (possible internal/background calls)
- `## Authentication Flow` — sequence: how a session is established, refreshed, and terminated
- `## Open SME Questions` — anything that needs clarification
