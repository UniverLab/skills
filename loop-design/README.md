# Loop Design Skill

**Loop orchestration skill** for turning multi-step work into reusable Canopy graphs with real MCP tooling constraints.

---

## Overview

This skill helps an agent turn a user goal into a structured Canopy loop made of:

- ordered specs
- `agent` nodes
- `check` nodes
- `gate` nodes
- routing edges between nodes

It is designed for **generic loops**, not a single hardcoded coding pipeline, and it keeps the plan grounded in the loop tools that actually exist.

---

## What It Does

The skill guides an agent to:

1. understand the user goal and constraints
2. split the work into atomic specs
3. choose an appropriate graph pattern per spec
4. persist the loop through Canopy MCP tools
5. summarize the result before execution

It also supports refining existing loops by inspecting first, then either extending the graph safely or recreating it when the current toolset cannot mutate the shape directly.

---

## When This Skill Triggers

Use it when the user asks to:

- plan or create a loop
- orchestrate several agents in background
- define checkpoints and approvals
- build developer/reviewer/verifier/committer flows
- convert a manual process into a Canopy graph
- update or refine a stored loop
- add retries, approvals, or pass/fail routing around agent work

---

## Skill Structure

```
loop-design/
├── SKILL.md              # Main skill definition
├── README.md             # This file
├── LICENSE               # MIT License
└── references/
    ├── loop-patterns.md
    └── mcp-tool-playbook.md
```

---

## MCP Tooling Covered

Creation flow:

- `loop_create`
- `loop_add_spec`
- `loop_add_node`
- `loop_add_edge`
- `loop_get`

Refinement flow:

- `loop_list`
- `loop_get`
- extend with `loop_add_spec`, `loop_add_node`, or `loop_add_edge` when the change is append-only
- recreate as a new loop when the graph shape must change in ways the toolset cannot mutate directly

Execution flow:

- `loop_run`
- `loop_pause`
- `loop_continue`

---

## Usage

Reference it from agent configuration:

```yaml
skills:
  - name: loop-design
    path: skills/loop-design/SKILL.md
    triggers:
      - "create loop"
      - "plan loop"
      - "background pipeline"
      - "orchestrate agents"
      - "checkpoint flow"
```

---

## References

- **loop-patterns.md** — examples of reusable graph shapes
- **mcp-tool-playbook.md** — recommended create/extend/replace flow with the current MCP toolset

---

## License

MIT © Jheison Martinez

---

## Version

1.2 (2026-05-13)
