# MCP Tool Playbook

Recommended tool order for creating and refining Canopy workflows with the currently available MCP tools.

---

## Create a New Workflow

1. `workflow_create`
2. `workflow_add_spec` for each ordered spec
3. `workflow_add_node` for each node inside the spec
4. `workflow_add_edge` for routing
5. `workflow_get` to verify the final shape
6. summarize the graph to the user
7. `workflow_run` only after explicit approval or direct instruction

---

## Refine an Existing Workflow

1. `workflow_list` or `workflow_get`
2. inspect current specs, nodes, edges, and statuses
3. choose the lowest-risk path:
   - extend the existing workflow with `workflow_add_spec`, `workflow_add_node`, or `workflow_add_edge` when the change is append-only
   - recreate a replacement workflow when the graph shape must change in ways the current toolset cannot mutate directly
4. `workflow_get` again
5. summarize the effective diff or migration path

---

## Mutation Heuristics

### Prefer extend-in-place when:

- the workflow identity should remain stable
- the new behavior can be added without deleting or rewriting existing graph structure
- users already rely on the current workflow ID and shape

### Prefer recreate when:

- the workflow graph shape changed drastically
- most specs are being replaced
- a clean new draft is easier to reason about

---

## Safety Rules

- do not auto-run after create unless the user explicitly asked for execution
- do not assume one CLI or one model provider
- do not insert commit nodes unless the user explicitly allows workflow-managed commits
- preserve strong verification nodes for code workflows
