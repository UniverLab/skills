# Workflow Design Skill

**Workflow orchestration skill** for turning multi-step work into reusable Canopy graphs with real MCP tooling constraints.

---

## Overview

This skill helps an agent turn a user goal into a structured Canopy workflow made of:

- ordered specs
- `agent` nodes
- `check` nodes
- `gate` nodes
- routing edges between nodes

It is designed for **generic workflows**, not a single hardcoded coding pipeline, and it keeps the plan grounded in the workflow tools that actually exist.

---

## What It Does

The skill guides an agent to:

1. understand the user goal and constraints
2. split the work into atomic specs
3. choose an appropriate graph pattern per spec
4. persist the workflow through Canopy MCP tools
5. summarize the result before execution

It also supports refining existing workflows by inspecting first, then either extending the graph safely or recreating it when the current toolset cannot mutate the shape directly.

---

## When This Skill Triggers

Use it when the user asks to:

- plan or create a workflow
- orchestrate several agents in background
- define checkpoints and approvals
- build developer/reviewer/verifier/committer flows
- convert a manual process into a Canopy graph
- update or refine a stored workflow
- add retries, approvals, or pass/fail routing around agent work

---

## Skill Structure

```
workflow-design/
├── SKILL.md              # Main skill definition
├── README.md             # This file
├── LICENSE               # MIT License
└── references/
    ├── workflow-patterns.md
    └── mcp-tool-playbook.md
```

---

## MCP Tooling Covered

Creation flow:

- `workflow_create`
- `workflow_add_spec`
- `workflow_add_node`
- `workflow_add_edge`
- `workflow_get`

Refinement flow:

- `workflow_list`
- `workflow_get`
- extend with `workflow_add_spec`, `workflow_add_node`, or `workflow_add_edge` when the change is append-only
- recreate as a new workflow when the graph shape must change in ways the toolset cannot mutate directly

Execution flow:

- `workflow_run`
- `workflow_pause`
- `workflow_continue`

---

## Usage

Reference it from agent configuration:

```yaml
skills:
  - name: workflow-design
    path: skills/workflow-design/SKILL.md
    triggers:
      - "create workflow"
      - "plan workflow"
      - "background pipeline"
      - "orchestrate agents"
      - "checkpoint flow"
```

---

## References

- **workflow-patterns.md** — examples of reusable graph shapes
- **mcp-tool-playbook.md** — recommended create/extend/replace flow with the current MCP toolset

---

## License

MIT © Jheison Martinez

---

## Version

1.2 (2026-05-13)
