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
   - **surgical mutation** with `workflow_update`, `workflow_update_spec`, `workflow_update_node`, or `workflow_update_edge` for targeted changes
   - **extend** with `workflow_add_spec`, `workflow_add_node`, or `workflow_add_edge` when the change is additive
   - **recreate** when the graph shape must change drastically
4. `workflow_get` again
5. summarize the effective diff or migration path

### Available Mutation Tools

| Tool | What it changes |
|---|---|
| `workflow_update` | name, description, workdir |
| `workflow_update_spec` | name, description, position, parallelizable |
| `workflow_update_node` | name, kind, config, position |
| `workflow_update_edge` | condition (pass/fail/always) |

---

## Mutation Heuristics

### Prefer surgical mutation when:

- a single node, spec, or edge needs a config tweak or rename
- the graph topology stays the same
- you want to preserve run history tied to the workflow ID

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
- use `workflow_report_blocker` when a node needs human input — do not hard-fail silently
- use `workflow_pause` + `workflow_continue` for graceful retry/skip flows

## Runtime Lifecycle

| Action | Tool |
|---|---|
| Start execution | `workflow_run` |
| Halt after current node | `workflow_pause` |
| Retry current node | `workflow_continue(action="retry_current_node")` |
| Skip to next spec | `workflow_continue(action="skip_next_spec")` |
| Node reports success/failure | `workflow_complete_node` |
| Node blocked, needs human | `workflow_report_blocker` |
