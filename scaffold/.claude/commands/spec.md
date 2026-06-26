---
description: Write one infrastructure-agnostic spec per module from the system map. Run all modules in parallel, or pass a module ID to spec just one.
---

Read `outputs/04-system-map.md`. If that file is missing, tell the user to run
`/decompose` first.

**Usage:**
- `/spec` — spec every module in the system map (run spec-writer subagents in parallel)
- `/spec M03` — spec just the module with ID M03

For each target module, invoke the `spec-writer` subagent with the module name
and ID. Pass enough context in the invocation that the agent knows which module
it's writing for.

Spec-writer agents are independent and can run in parallel — launch them all at
once rather than sequentially when processing multiple modules.

Each spec-writer should produce `outputs/specs/<module-id>-<module-name>.spec.md`.

When all agents complete, report:
- Which spec files were written
- Any modules where the agent flagged insufficient discovery data
- Any open questions that must be resolved before the spec can go to build

**Reminder:** specs must be infrastructure-agnostic. If any spec references a
concrete database or cloud provider, send it back to the spec-writer for revision.
