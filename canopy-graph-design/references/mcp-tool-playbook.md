# MCP Tool Playbook

Recommended tool order for creating and refining Canopy graphs with the currently available MCP tools.

---

## Create a New Loop

### The default: one call 🔴

**`graph_import {document, workdir}` builds an entire graph in a single call.**
The document is what `graph_export` returns: name, description, nodes (with full
prompts and commands) and edges. Edges reference nodes **by name**, not by id,
which is what makes it portable. Validation is **all-or-nothing** — if any part
is rejected, nothing is written.

Prefer it. Building a 9-node graph by hand is ~28 calls, each re-sending the
conversation; the same graph as a document is one. And a malformed document
fails loudly and atomically, where hand-built graphs fail *silently*: a missing
`fail` edge is a property of the whole shape that no individual `graph_add_edge`
call can see (graph-validation rule 10).

- `graph_export {graph_id, with_models}` produces the document. It **strips
  `platform`/`model` by default** so a shared design does not pin the recipient
  to a harness they may not have; pass `with_models: true` when exporting your
  own graph to restore later. `graph_import`'s response lists every agent node
  left without a platform.
- Ids, workdir, specs and run state are never exported. The entry point is
  implicit: the node with no incoming edges.
- Import always creates a **new** graph; it never updates one. A name collision
  gets a numeric suffix, and the response says which name was used.
- `prompt_template` text rides inside the document, but a node's
  `prompt_preset` rides **by name**. `graph_import` therefore refuses the whole
  document, naming every preset it cannot resolve, unless
  `<canopy_dir>/prompts/<name>.md` exists on this machine or the name is a
  builtin (`implementer`, `reviewer`, `resilience`). Before importing on a
  different machine, copy the custom preset there — or carry its text as
  `prompt_template`.

### Building by hand (when there is no document yet)

1. `graph_create` — set `trigger` here if the graph should fire on its own (see below); omit it for a manual-only graph
2. `graph_add_node` for each node — insertion order sets `position`, and `position` is load-bearing (graph-validation rule 1)
3. `graph_add_edge` for routing
4. `graph_get` to verify the final shape against `references/graph-validation.md`
5. `graph_export` it once it works, so the next one is a single call
6. `graph_preflight {graph_id}` — probes every distinct platform+model the graph references, before a real run spends anything on a harness that cannot answer. Confirm a `broken` verdict with `agent_probe {platform, model}` on its own before rewiring anything: a pair preflight calls broken can answer fine when probed alone.
7. summarize the graph to the user
8. `graph_run` only after explicit approval or direct instruction — not needed at all if a cron/watch trigger will fire it

Specs are **not** part of the graph any more: they live in the standalone
backlog (`spec_create`) and reach the graph through a queue — see the Queues
section below. `graph_add_spec` binds a spec to the graph itself and is the older
shape; prefer `spec_create` + `queue_*`.

---

## Setting a Trigger (cron / watch / manual)

`graph_create` and `graph_update` both take an optional `trigger` object (`GraphTriggerParams`). Omit it on create for a manual loop; omit it on update to leave the current trigger untouched.

| Field | Applies to | Notes |
|---|---|---|
| `kind` | all | `"cron"`, `"watch"`, or `"manual"`. `"manual"` (or empty) clears any existing trigger. |
| `schedule` | cron | 5-field cron expression, required when `kind = "cron"`. Evaluated in local wall-clock time, same as agent triggers. |
| `path` | watch | Absolute path to a file or directory, required when `kind = "watch"`. |
| `events` | watch | List of `"create"`, `"modify"`, `"delete"`, `"move"`, or `"all"`. Required (non-empty) when `kind = "watch"`. |
| `debounce_seconds` | watch | Optional, default `2`. |
| `recursive` | watch | Optional, default `false`. |

Example — nightly cron graph:

```json
{
  "name": "nightly-docs-sync",
  "workdir": "/home/user/project",
  "trigger": { "kind": "cron", "schedule": "0 2 * * *" }
}
```

Example — clearing a trigger back to manual via `graph_update`:

```json
{ "graph_id": "...", "trigger": { "kind": "manual" } }
```

A fireable graph (cron/watch) will not re-trigger itself while it is already `Running` or `Paused` — no need to guard against overlapping runs when designing the graph.

---

## Refine an Existing Loop

1. `graph_list` or `graph_get`
2. inspect current specs, nodes, edges, and statuses
3. choose the lowest-risk path:
   - **surgical mutation** with `graph_update`, `graph_update_spec`, `graph_update_node`, or `graph_update_edge` for targeted changes
   - **extend** with `graph_add_spec`, `graph_add_node`, or `graph_add_edge` when the change is additive
   - **recreate** when the graph shape must change drastically
4. `graph_get` again
5. summarize the effective diff or migration path

### Available Mutation Tools

| Tool | What it changes |
|---|---|
| `graph_update` | name, description, workdir |
| `graph_update_spec` | name, description, position, parallelizable |
| `graph_update_node` | name, kind, config, position |
| `graph_update_edge` | condition (`pass`/`fail`/`error`/`always`/`route`) and, since CB11, the target node |
| `graph_delete_edge` | removes one edge by id |
| `graph_delete_node` | removes a node **and cascades to every edge naming it** — and, if that node is an
ensemble's entry or exit, to the ensemble row itself (CB52 made the members and join go with it instead of
being orphaned) |
| `graph_copy_node` | duplicates a node's config into this or another graph, optionally wiring it and shallow-merging `config_overrides` |

**Retargeting an edge is `graph_delete_edge` + `graph_add_edge`.** Earlier
revisions of this file said no delete tools existed and that any topology change
forced recreating the graph — that has not been true for a while, and following
it means recreating graphs for nothing.

Both delete tools are **rejected while the graph is running** (pause first), and
`graph_delete_node` refuses the graph's entry node.

Editing a node's `config` while a graph runs **is** safe: the graph is
snapshotted per spec, so the change takes effect on the next one. Changing the
*topology* of a running graph is not.

`graph_add_spec` while a graph runs **is** safe too: bound specs are read
live at every advance, so a spec bound mid-run joins the running dispatch in
position order — after the current spec's position in its place, or right
after the current spec if bound earlier. The run never reports completion
while a bound spec is still pending: it fails naming the spec in
`graph_failed`, and `graph_continue` runs it.

---

## Mutation Heuristics

### Prefer surgical mutation when:

- a single node, spec, or edge needs a config tweak or rename
- the graph topology stays the same
- you want to preserve run history tied to the graph ID

### Prefer extend-in-place when:

- the graph identity should remain stable
- the new behavior can be added without deleting or rewriting existing graph structure
- users already rely on the current graph ID and shape

### Prefer recreate when:

- the graph graph shape changed drastically
- most specs are being replaced
- a clean new draft is easier to reason about

---

## Safety Rules

- do not auto-run after create unless the user explicitly asked for execution
- do not assume one CLI or one model provider
- do not insert commit nodes unless the user explicitly allows graph-managed commits
- preserve strong verification nodes for code graphs
- use `graph_report_blocker` when a node needs human input — do not hard-fail silently
- use `graph_pause` + `graph_continue` for graceful retry/skip flows

## Runtime Lifecycle

| Action | Tool |
|---|---|
| Start execution | `graph_run` |
| Halt after current node | `graph_pause` |
| Retry current node | `graph_continue(action="retry_current_node")` |
| Skip to next spec | `graph_continue(action="skip_next_spec")` |
| Start a spec at a chosen node (directed run) | `graph_continue(action="run_from_node", node_id, spec_id?)`; the first spec of a `graph_run` can use `start_node_id` instead. Reuses earlier nodes' recorded output; refused while the graph has an active run |
| Re-open a failed/completed graph | `graph_reset` (optionally `{specs:[…]}`; a named spec's execution count is zeroed, a reset without `specs` keeps it), then `graph_run` |
| Resume once at an exact future time | `graph_schedule_autorun(graph_id, at)` |
| Re-enable an agent once at an exact time | `agent_schedule_enable(id, at)` |
| Node reports success/failure | `graph_complete_node` |
| Node blocked, needs human | `graph_report_blocker` |

---

## Recovery Matrix (graph status → how to move it)

Learned from real incidents; each row was needed at least once.

| Loop state | How to recover |
|---|---|
| `draft` / `pending` | `graph_run` |
| `paused` | `graph_continue` (`retry_current_node` or `skip_next_spec`) |
| `running` but the daemon restarted (zombie: no child process behind it) | `graph_pause`, then `graph_continue(retry_current_node)`. Do **not** wait for a scheduled autorun: `is_fireable()` excludes `Running`, so it will never fire. A clean stop or crash that killed a node already leaves the graph `paused` with the spec `interrupted`; `graph_continue(retry_current_node)` then re-runs that same node. |
| `paused` with a blocker from a no-verdict ensemble | Fix the members with `graph_update_ensemble` if they are at fault, then `graph_continue(retry_current_node)`, which re-enters the ensemble without re-running passed nodes. A `graph_run` on a blocked graph is refused. |
| `failed` with blocker `… cannot complete: N bound spec(s) still pending (…)` | The run finished the work it had, but a spec bound mid-run was left pending — nothing was lost. `graph_continue(retry_current_node)` runs it; no reset needed. |
| `failed` | `graph_reset` (clears the failed state, keeps completed specs), then `graph_run` for a single clean launch. `graph_reset` with no `specs` resets only non-completed specs, so `graph_run` resumes at the first pending one — nothing already done is re-implemented. To redo only what failed instead, `graph_continue(action="retry_current_node")` is now accepted on a `failed` graph too: it restarts the failed spec at the node it failed on as a directed run (upstream outputs reused) and clears that spec's execution count; `graph_continue(action="skip_next_spec")` skips the failed spec directly. |
| `completed` | `graph_reset {specs:[…]}` to re-open specific specs, then `graph_run`. |
| `failed` with `blocker: null`, last run `pass` | The per-node **iteration budget ran out** (5 — graph-validation rule 7), not a crash. Check the last run's `iteration` before relaunching: the work is usually nearly done and the reviewer already left a precise list, so finishing it by hand and marking the spec `completed` (`spec_set_status`) costs far less than another five rounds. |
| `failed` in a queue-driven graph, when only the failed member's tail must be redone | `graph_continue(action="run_from_node", node_id=…, spec_id=…)` — see *Restarting one member* below. Omit `spec_id` and the failed member is the default. |
| `failed` on an idea-driven graph (spec-less, run with an `idea`) | `graph_continue(action="retry_current_node")` or `run_from_node` resumes that idea run directly — the graph stores the idea it last ran with, `{{spec_content}}` renders it, and the blank bookkeeping spec's node history (so `{{previous_feedback}}` still carries the failure) is kept. No `spec_id` to name. An idea graph that has never run is refused: start it with `graph_run` passing an `idea`. |
| `failed` with blocker `… failed N time(s) across separate graph executions (limit 3) …` | The per-spec **execution limit** (default 3; genuine verdicts only). `graph_reset {specs:[<spec id>]}` zeroes that spec's count — worktree untouched — then `graph_run`; or give an explicit directive (`graph_run` `start_node_id`, `graph_continue` `run_from_node` / `retry_current_node`), which clears it automatically and replies `Execution limit reset for spec '…'`. Automatic re-runs (autorun, cron) never clear it. |
| `paused` with a blocker listing dirty paths between queue specs | The **between-specs dirty-tree guard** held the queue at the next spec instead of failing it. Ways out: `graph_continue(action="run_from_node", …)` on that spec, clean the tree by hand then `graph_continue(skip_next_spec)`, or `allow_dirty_start: true` via `graph_update`. |

**Prefer `graph_reset` + `graph_run` over `graph_schedule_autorun` for manual recovery.**
`graph_schedule_autorun` auto-resets and resumes when it fires — convenient, but its
resume-and-launch path can double-launch a graph that already has an in-flight run,
which routes a superseded run down the fail edge and (with a resilience node there)
can spin an implement→resilience→implement loop. Use autorun only for the case it is
built for: a quota death converting a stated reset time into one future resume. For
resuming *now*, `graph_reset` then a single `graph_run` avoids the double-launch.

Two asymmetries worth knowing:

- **`graph_run` freezes the spec list at launch; `graph_continue` re-reads it.** Adding
  a spec and then resuming with `graph_continue` picks it up; adding one mid-`graph_run`
  does not.
- A commit-check that keys on a marker file (e.g. `.git/canopy-prev-head`) survives
  daemon/PC restarts pointing at a stale HEAD. Delete the marker in pre-flight, and
  only resume at or before the gates node that rewrites it.

---

## Queues: runtime spec lists the engine runs through a graph

The redesign landed. A graph is now a reusable **graph** (the "team"), and the
specs it runs come from a standalone backlog (`spec_create` / `spec_list`)
collected into a **queue** the graph executes in order:
`graph_run {graph_id, queue_id, workdir}`. Build the graph once (~13 calls), then
point different queues at it across workdirs — the graph's own bound specs are
typically empty when it runs a queue. (`pool` is the old name for the same
thing; `queue_*` tools supersede `graph_pool_*`, and `queue_id` wins over the
deprecated `pool_id` argument.)

| Tool | What it does |
|---|---|
| `queue_create` | create an empty queue |
| `queue_add_spec {queue_id, spec_id, group?}` | append a backlog spec; optional `group` shares a warm session (below) |
| `queue_remove_spec` | drop a spec from the queue |
| `queue_reorder {queue_id, spec_ids}` | total reorder — pass **every** member exactly once, in the desired order; refused if it would split a group's pending members |
| `queue_set_group {queue_id, spec_id, group?}` | set or clear (omit `group`) one member's group in place; the member moves next to its group |
| `queue_list {queue_id?}` | one queue's ordered members, or all queues in summary |

**Warm-context groups.** `group` on `queue_add_spec` makes a spec resume the
session of the previous **successfully-completed** sibling in the same group
instead of analyzing the repo cold — see *Spec Groups* in `SKILL.md`. Group
only an ordered sequence over the same surface; leave unrelated specs ungrouped.
A grouped add lands after the group's last pending member. To (re)group an
existing member, use `queue_set_group` (or `canopy queue group`), not remove and
re-add. `queue_reorder` keeps each group's pending members contiguous and
refuses an order that would split one.

**Membership vs. status.** `queue_*` tools change membership only; a spec's
run status (pending / completed / failed) lives on the spec, so removing and
re-adding a spec does not re-run an already-completed one — `graph_run` still
resumes at the first non-completed member.

**Restarting one member.** `graph_continue(action="run_from_node", node_id=…,
spec_id=…)` on a queue-driven graph restarts the member you name, at the node
you name. The chosen member becomes the graph's current spec: the members
queued before it are marked `skipped`, the members after it run in their usual
order, and a member left `running` by a stale state goes back to `pending` for
a later run. Omit `spec_id` and the default is the running/interrupted member,
else the failed one, else the next pending one. The node you name always runs
first — including the node an ensemble fans out from: that node runs, and
routing then fans out to the ensemble from there. The reply names the same
spec and the same first node the engine will run ("Directed run: spec '…'
starts at node '…'"), so read it before assuming which spec was picked up.

---

## Language

Write spec descriptions and node prompt templates in **English** — models follow
English instructions more reliably, and the reviewer/implementer contract gets
stricter compliance.
