# Graph Validation — fatal shapes to avoid

Validate BEFORE `loop_run`; the engine is strict and fails hard, often *after*
a node already burned real work. Each rule below broke a real run.

1. **Entry node.** The engine starts a spec at the unique node with no incoming
   edge; a retry cycle leaves none, and the engine then falls back to the node
   with the lowest `position`. Always add the intended start node FIRST
   (`loop_add_node` assigns `position` by insertion order) and treat
   `position` as load-bearing.
2. **Ambiguous edges.** Two edges from the same node matching the same result
   and pointing at different targets abort the loop with
   `"ambiguous outgoing edges"`. Dedupe by `(from_node, condition)` before
   inserting. A `pass` edge and a `fail` edge from the same node are fine.
3. **`config` must be a JSON object.** The schema now enforces it, but verify
   anything created by other routes. A string config dies mid-run with
   `"missing a platform/cli"`.
4. **`check` timeout defaults to 120 s** — far too low for `cargo test`. Set
   `timeout_seconds` explicitly (1500 for build+test gates).
5. **`gate` nodes match against the JSON-serialized previous output**, not
   prose. Gate on a strict token (`"VERDICT: APPROVED"`), never on a word that
   can appear in narration.
6. **The daemon resolves each node's CLI from its own environment**, not your
   shell — under systemd that PATH is minimal. Pin absolute binaries in
   `~/.canopy/config.toml` (re-read per node, no restart needed).
7. **Iteration budget is 5 per (spec, node), reset per spec**
   (`DEFAULT_MAX_ITERATIONS_PER_NODE`, verified in `loop_engine.rs:20` on
   2026-08-27 — earlier revisions of this file said 10, which was wrong and
   made specs look cheaper to retry than they are). A resilience node that
   always reports pass burns all five attempts against the implementer.
   **When the budget runs out the loop ends `failed` with `blocker: null`** and
   the last run showing `pass` — there is no message saying the ceiling was
   hit. Diagnose it by comparing the last run's `iteration` against 5 before
   assuming a crash or a quota death.
8. **`loop_run` freezes the spec list at launch; `loop_continue` re-reads it.**
   Plan mid-run additions around a pause/continue boundary.
9. **Route the implement's success edge with `pass`, never `always`** — an
   `always` edge feeds a dead implement's output into the reviewer, which then
   approves work that does not exist.
10. **Every node needs an exit for the statuses it can actually report.** A
    node that ends `fail` with no `fail` edge terminates the spec with
    `"ended 'fail' with no outgoing edge for that status"`. That is correct for
    a deliberate dead end (a resilience node that gives up *should* stop for a
    human) and a silent trap everywhere else. This is a property of the whole
    graph, not of any one edge, so no single `loop_add_edge` call can catch it:
    check it against the finished shape, or build the graph with `loop_import`,
    where the whole document is in front of you at once.

Read a failure from the runs table before touching anything: the failing node,
its iteration count, and its raw output tell you whether the graph, the
prompt, or the platform is at fault. Recovery flows live in
`mcp-tool-playbook.md` → "Recovery Matrix". More field notes (with the
incidents behind these rules) live in `loop-patterns.md` → "Failure modes".
