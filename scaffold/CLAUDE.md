# WG AI Partners — Research Agent (Project Guide)

A Claude Code toolkit for incrementally replacing a SaaS application using the
Strangler Fig pattern, working from black-box analysis (no source code).

## The one rule: specs are infrastructure-agnostic
No skill, agent, command, or spec may name a concrete database engine or cloud
provider. Specs describe *logical* capabilities ("relational store", "object
storage", "managed queue", "identity provider"). The only file that names real
technology is `infrastructure-profile.yaml`, which binds logical capabilities to
concrete services at build time. This is what lets one set of specs target
Postgres-on-AWS or MySQL-on-GCP with no rework.

## Layout
- `.claude/skills/`   — analysis methodology (loaded on demand)
- `.claude/agents/`   — isolated subagents that run each stage in a clean context
- `.claude/commands/` — slash commands that drive the pipeline
- `inputs/`           — raw client artifacts. NEVER COMMITTED (see Privacy).
- `outputs/`          — generated inventory, system map, and per-module specs

## Pipeline

**With a live app URL (no pre-collected notes):**
1. `/crawl <url>` -> ui-crawler walks the live app, writes output 01
2. `/discover`    -> discovery agent reads HAR + DB exports, writes outputs 02-03
                     (skips 01 if already written by /crawl)
3. `/decompose`   -> architect agent writes the system map (04) + strangler order
4. `/spec`        -> spec-writer produces one spec per module under outputs/specs/
5. build          -> Claude Code builds each module in strangler order

**With pre-collected artifacts in inputs/:**
1. `/discover`  -> discovery agent reads `inputs/`, writes outputs 01-03
2. `/decompose` -> architect agent writes the system map (04) + strangler order
3. `/spec`      -> spec-writer produces one spec per module under outputs/specs/
4. build        -> Claude Code builds each module in strangler order, binding the
                   infrastructure profile

## Discovery uses triangulation, not guessing
DB exports are ground truth for schema; HAR is ground truth for API contracts;
UI confirms what's user-facing; SMEs supply hidden business rules. Tag every
finding with its source(s) and a confidence level; emit anything un-triangulated
as an open SME question rather than a guess.

## Modernization stance
Target is "same scope, modernized" — capture *intended* behavior and clean module
boundaries. Do not replicate legacy quirks or bugs in the specs.

## Privacy (non-negotiable)
`inputs/` routinely contains live PII, auth tokens, and session cookies. It is
gitignored. Scrub artifacts before they enter the repo. Never paste raw HAR or DB
dumps into committed files.
