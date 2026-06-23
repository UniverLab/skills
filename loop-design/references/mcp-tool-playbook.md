# MCP Tool Playbook

Recommended tool order for creating and refining Canopy loops with the currently available MCP tools.

---

## Create a New Loop

1. `loop_create`
2. `loop_add_spec` for each ordered spec
3. `loop_add_node` for each node inside the spec
4. `loop_add_edge` for routing
5. `loop_get` to verify the final shape
6. summarize the graph to the user
7. `loop_run` only after explicit approval or direct instruction

---

## Refine an Existing Loop

1. `loop_list` or `loop_get`
2. inspect current specs, nodes, edges, and statuses
3. choose the lowest-risk path:
   - **surgical mutation** with `loop_update`, `loop_update_spec`, `loop_update_node`, or `loop_update_edge` for targeted changes
   - **extend** with `loop_add_spec`, `loop_add_node`, or `loop_add_edge` when the change is additive
   - **recreate** when the graph shape must change drastically
4. `loop_get` again
5. summarize the effective diff or migration path

### Available Mutation Tools

| Tool | What it changes |
|---|---|
| `loop_update` | name, description, workdir |
| `loop_update_spec` | name, description, position, parallelizable |
| `loop_update_node` | name, kind, config, position |
| `loop_update_edge` | condition (pass/fail/always) |

---

## Mutation Heuristics

### Prefer surgical mutation when:

- a single node, spec, or edge needs a config tweak or rename
- the graph topology stays the same
- you want to preserve run history tied to the loop ID

### Prefer extend-in-place when:

- the loop identity should remain stable
- the new behavior can be added without deleting or rewriting existing graph structure
- users already rely on the current loop ID and shape

### Prefer recreate when:

- the loop graph shape changed drastically
- most specs are being replaced
- a clean new draft is easier to reason about

---

## Safety Rules

- do not auto-run after create unless the user explicitly asked for execution
- do not assume one CLI or one model provider
- do not insert commit nodes unless the user explicitly allows loop-managed commits
- preserve strong verification nodes for code loops
- use `loop_report_blocker` when a node needs human input — do not hard-fail silently
- use `loop_pause` + `loop_continue` for graceful retry/skip flows

## Runtime Lifecycle

| Action | Tool |
|---|---|
| Start execution | `loop_run` |
| Halt after current node | `loop_pause` |
| Retry current node | `loop_continue(action="retry_current_node")` |
| Skip to next spec | `loop_continue(action="skip_next_spec")` |
| Node reports success/failure | `loop_complete_node` |
| Node blocked, needs human | `loop_report_blocker` |
