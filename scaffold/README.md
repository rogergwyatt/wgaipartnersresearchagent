# WG AI Partners — Research Agent

A Claude Code toolkit for incrementally replacing a SaaS application with the
Strangler Fig pattern, from black-box analysis (no source code). It produces an
organizational breakdown of the system and an infrastructure-agnostic spec per
module, ready for Claude Code to build against a client-chosen database and cloud.

## Pipeline
1. `/discover`  — feature inventory, observed API surface, reconstructed data model
2. `/decompose` — system map: bounded contexts, dependencies, strangler cut order
3. `/spec`      — one infrastructure-agnostic spec per module
4. build        — Claude Code builds each module in strangler order

## Layout
    .claude/skills/    analysis methodology (5 skills)
    .claude/agents/    discovery, architect, spec-writer subagents
    .claude/commands/  /discover, /decompose, /spec
    inputs/            raw client artifacts (gitignored)
    outputs/           generated inventory, system map, specs

## Getting started
1. Drop client artifacts into `inputs/` (gitignored — see Privacy).
2. Fill in `infrastructure-profile.yaml`.
3. Run `/discover`, then `/decompose`, then `/spec`.
4. Build each module in strangler order.

## Status
Scaffold. Skill / agent / command bodies are stubs to be fleshed out next.

## Privacy
`inputs/` holds live PII and secrets and is never committed. Scrub before use.
