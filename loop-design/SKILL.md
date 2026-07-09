---
name: loop-design
description: >
  Use this skill when the user wants to turn a recurring or multi-step process
  into a reusable Canopy loop, inspect or reshape an existing loop
  graph, or coordinate background agent/check/gate flows with approvals and
  retries. Prefer reusable graph patterns over one-off pipelines, and build
  against the MCP loop tools that actually exist in the environment.
license: MIT
metadata:
  author: jheison.martinez
  version: "1.7"
  framework: Canopy
  category: loop-orchestration
  last_updated: "2026-07-09"
---

# Loop Design

Design generic loops for Canopy as editable graphs, not as one-off scripts.

This skill exists for planning and authoring **background loops** that users can later inspect, edit, and run from Canopy.

---

## Mission

Translate a user goal into:

1. ordered specs
2. a graph per spec
3. the right mix of `agent`, `check`, and `gate` nodes
4. a persisted loop built with MCP tools

The output must stay generic enough that different users can plug in different CLIs, models, prompts, and verification commands.

---

## When This Skill Triggers

Use this skill when the user asks for any of the following:

- create or plan a loop
- build a background pipeline
- chain multiple agents with checkpoints
- orchestrate developer/reviewer/verifier/committer flows
- create reusable specs for implementation, bug fixing, refactors, docs, or ops
- convert a manual process into a Canopy loop graph
- refine an existing loop graph
- turn a repeated manual process into a reusable loop
- add approvals, retries, or verification gates around agent work

Do **not** reduce everything to one fixed "coding pipeline" unless the user explicitly wants that exact pattern.

---

## Core Rules

### 1. Model loops as graphs, not lists

Each spec is an ordered work unit, but inside each spec you should think in graph form:

- `agent` — produces or analyzes work
- `check` — verifies with commands or deterministic checks
- `gate` — decides the next route based on prior output

If the loop needs iteration, use edges and gates. If it is linear, keep it simple.

### 2. Prefer explicit specs with narrow intent

Good spec boundaries:

- "Implement auth domain service"
- "Add API handlers and validation"
- "Run integration checks and finalize"

Bad spec boundaries:

- "Build the whole app"
- "Do all backend work"

### 3. Preserve genericity

Never assume:

- one CLI platform
- one model vendor
- one review style
- one commit strategy

Any node may use Copilot, OpenCode, Kimi, local MCP-driven agents, or other supported CLIs. Design prompts and checks accordingly.

### 4. Ask before irreversible behavior

Before you create or run a loop, clarify:

- whether creation should stay draft or also run
- what verification commands are authoritative
- whether commits are allowed inside the loop
- whether blockers should pause or hard-fail the loop

### 4b. A loop does not have to be run by hand

A loop can be scheduled to fire on its own, the same way background agents can. `loop_create`/`loop_update` accept an optional `trigger` — a loop is not limited to `loop_run` on demand:

- **manual** (default) — the loop only starts when someone calls `loop_run`. Clearing a trigger (or omitting it on create) keeps this mode.
- **cron** — fires on a 5-field cron expression, evaluated in the same **local wall-clock** frame as agent triggers (not UTC).
- **watch** — fires when a file or directory changes (create/modify/delete/move, or "all"), with a debounce window.

Ask the user which mode fits before creating the loop: a one-off migration is almost always manual; a recurring maintenance sweep (nightly cleanup, "re-run docs sync whenever the schema file changes") is a candidate for cron or watch. See **[references/mcp-tool-playbook.md](references/mcp-tool-playbook.md)** for the exact `trigger` fields.

### 5. Reuse patterns, adapt prompts

Reuse graph patterns from the reference docs, but adapt:

- node prompts
- CLI/model selection
- verification commands
- retry / fail routing
- blocker behavior

---

## Loop Construction Playbook

### Step 1 — Understand the target outcome

Extract:

- final deliverable
- hard constraints
- allowed tools/CLIs
- required validations
- whether specs can run in parallel

### Step 2 — Split into specs

Specs should be:

- ordered
- independently understandable
- small enough to validate

### Step 3 — Pick a graph shape per spec

Typical choices:

- **linear:** `agent -> check`
- **review loop:** `agent -> review -> gate -> agent`
- **verify loop:** `agent -> check -> gate -> agent`
- **finish gate:** `agent/check -> gate(pass_route=next_spec)`
- **gated implement:** `implement --pass--> check -> reviewer/committer`
  (see pattern 7 in the reference — the shape that stops a dead implement from
  reaching the reviewer)
- **resilience branch:** add `implement --fail--> resilience` so a failed
  implement is *triaged* instead of silently ending the spec (pattern 8)

**Route the implement's success edge with `pass`, never `always`.** An `always`
edge sends a *failed* implement (crash, quota exhaustion) straight to the reviewer,
which then approves work that does not exist. This is not hypothetical: it marked
specs `completed` with no commit behind them.

### Step 4 — Persist via MCP tools

Use the Canopy loop tools, in order when creating:

1. `loop_create`
2. `loop_add_spec`
3. `loop_add_node`
4. `loop_add_edge`
5. `loop_get` to verify the final shape

When refining existing loops, inspect with `loop_list` / `loop_get` first.

The toolset supports in-place mutations:

- `loop_update` — rename, redescription, or move workdir
- `loop_update_spec` — rename, change description, reorder, toggle parallelizable
- `loop_update_node` — rename, change kind, replace config, reposition
- `loop_update_edge` — change routing condition (pass/fail/always)

Use these when the change is surgical. Use `loop_create` + rebuild when the graph shape changes drastically.

Runtime tools:

- `loop_run` — start execution
- `loop_pause` — halt after current node finishes
- `loop_continue` — resume with retry_current_node or skip_next_spec
- `loop_complete_node` — report pass/fail from inside a node
- `loop_report_blocker` — pause because a node needs human intervention

Only call `loop_run` after the user has the chance to review the graph, unless they explicitly asked to launch it immediately.

---

## Graph Validation — fatal shapes to avoid

The engine executes a spec by walking its edge graph, and it is **strict**: several
malformed shapes don't degrade gracefully — they abort the whole loop, sometimes
before a single node runs. Validate every spec against these before `loop_run`.
Field lessons that motivated this section are in
**[references/loop-patterns.md](references/loop-patterns.md)** under "Failure modes".

### 1. Exactly one resolvable entry node per spec

The start node is resolved topologically: the node with **no incoming edge**. Two
traps:

- **Retry cycles hide the entry.** A back-edge like `review --fail--> implement`
  gives the implement node an incoming edge, so a naive `implement <-> review`
  cycle has **zero** source nodes → the spec cannot start (historically:
  `"Spec has no entry node"`, failing the loop instantly with no runs and
  `started_at == completed_at`). The engine now falls back to the
  **lowest-`position`** node in that case, so **node `position` is meaningful** —
  make the intended start `position: 1`. Don't rely on insertion order.
- **Multiple sources are also rejected** (`"multiple entry nodes"`). If a spec has
  two disconnected starts, that's a modelling error — split it or connect them.

### 2. No ambiguous outgoing edges

When a node finishes, the engine selects the next edge whose condition matches the
result (`pass`|`fail`, and `always` matches both). If the matching edges point at
**different targets**, it aborts with `"ambiguous outgoing edges"` — fatal, and it
happens *after* the node already did its work.

- Two edges that are byte-identical (same `to_node` **and** condition) are now
  deduped rather than fatal. Don't lean on it: still guard against inserting the
  same edge twice when building in a batch.
- Never emit `always` + `pass` from one node to **different** targets — both match
  a pass, and that is the genuine ambiguity the engine rejects.
- A `pass` edge and a `fail` edge from the same node are fine: disjoint conditions.

### 3. Node config: the defaults that bite

- **`config` must be a JSON object.** The MCP schema now declares
  `"type": "object"` and serde rejects anything else, so a JSON-*encoded string*
  fails at parse time. Historically it was stored as a string and the node died
  mid-run with `"missing a platform/cli"`.
- **`check` nodes default to `timeout_seconds: 120`.** A real test suite blows
  through that. Set it explicitly (`1500` is a sane floor for a Rust workspace) or
  every check dies by timeout.
- **`gate` nodes match against the JSON-serialized previous output**, not a clean
  string. `output_contains: "APPROVED"` collides with prose like *"not approved"*.
  Use a strict token the reviewer must emit verbatim, e.g. `"VERDICT: APPROVED"`.
- Required keys by kind: `agent` → `platform`; `check` → `command`;
  `gate` → `value`.

### 4. Every node's platform must resolve **on the daemon**

Agent nodes spawn their CLI from the **canopy daemon's** environment (often a
systemd user service), *not* your interactive shell. A `platform` whose binary is
on your `$PATH` but not the daemon's will fail to spawn mid-run.

- Prefer an **absolute `binary` path** in the CLI registry (`~/.canopy/config.toml`)
  for any CLI installed outside standard locations, or ensure the daemon's `PATH`
  includes it. The registry is re-read per node execution, so a binary-path fix is
  picked up **without restarting the daemon**.
- An empty `model` is tolerated by some CLIs (falls back to their default), but be
  explicit when you can.

### 5. The iteration budget

Each node may run at most `DEFAULT_MAX_ITERATIONS_PER_NODE` (10) times within a
spec; exceeding it fails the spec with `"exceeded max iterations"`. The budget is
**reset when a spec starts fresh** and only carried over when *resuming* a spec
left in `Running`. So a spec killed repeatedly by an external cause (CLI quota)
no longer dies permanently after ten attempts.

A `check --fail--> implement` ping-pong is therefore bounded, not infinite.

### 6. Reading a failure & recovering

Two failure signatures, and they mean opposite things:

- **Structural:** loop `status = failed` but **0 specs failed**, no runs recorded,
  `started_at == completed_at`. That's a graph/spawn error, not agent work — read
  `~/.canopy/daemon.log` for the exact `bail!` message.
- **Honest:** loop `failed` with a spec actually marked `failed`. A node genuinely
  failed (e.g. the CLI hit its session limit). This is what a well-shaped graph
  produces, and it is cheap to recover from.

**Relaunching:** `loop_run` refuses `completed`/`failed` loops
(`"cannot be resumed yet"`), and `loop_continue` only acts on **paused** loops.
Reset the loop's status (and the affected spec's) and run again — **completed specs
are skipped**, so prior work is never redone.

**Scheduled resume without polling:** `loop_schedule_autorun { loop_id, at }` fires
a loop **once** at an exact time; `agent_schedule_enable { id, at }` does the same
for a disabled background agent. Both fold into the scheduler's wakeup calculation,
so nothing polls. This is the right answer when a node dies on a quota that resets
at a known hour — far better than a recurring cron that retries blindly and burns
the iteration budget.

Note the asymmetry: the `"cannot be resumed yet"` guard lives in the **MCP handler**
for `loop_run`, not in the engine. A scheduled trigger (`autorun_at`, cron) calls
the engine directly and will happily resume a `failed` loop.

**Pre-flight checklist before `loop_run`:** per spec, exactly one entry (or a clear
lowest-`position` start) · no duplicate/overlapping outgoing edges per condition ·
every `config` an object with its kind's required key · `check` timeouts raised
above the 120 s default · every referenced platform resolvable by the daemon.

### 7. `run_loop` freezes the spec list

`run_loop` reads the spec list **once** at launch. A spec added to a running loop
will not execute in that pass: let it finish, reset the loop to `draft`, and
`loop_run` again — the completed specs are skipped and only the new one runs.

---

## Authoring Guidelines

### Agent nodes

Agent node configs should be explicit:

```json
{
  "platform": "copilot",
  "model": "gpt-5.4",
  "prompt_template": "{{spec_content}}\n\n{{previous_feedback}}",
  "timeout_minutes": 30
}
```

Prefer prompts that:

- state the spec goal clearly
- include expected outputs
- mention constraints
- tell the node when to report blocker vs fail

### Check nodes

Check nodes should encode real verification, not vague "looks good" logic.

Good examples:

- `cargo fmt --check && cargo test && cargo clippy --all-targets -- -D warnings`
- `npm test`
- `pytest tests/auth -q`
- domain-specific smoke checks

### Gate nodes

Use gates when routing depends on semantics, not just exit codes.

Examples:

- continue only when output contains `APPROVED`
- send back to implementer if review found unresolved issues
- advance to next spec when a checkpoint summary confirms completion

---

## Summary Contract

Before finishing, present a concise summary containing:

1. loop name
2. each spec in order
3. graph per spec
4. CLIs/models/checks chosen
5. what still requires user confirmation before `loop_run`

---

## Progressive Disclosure

- Read **[references/loop-patterns.md](references/loop-patterns.md)** when choosing a graph shape for a spec.
- Read **[references/mcp-tool-playbook.md](references/mcp-tool-playbook.md)** immediately before creating, extending, or replacing a loop through MCP tools.

---

## Reference Documentation

- **[references/loop-patterns.md](references/loop-patterns.md)** — reusable graph patterns
- **[references/mcp-tool-playbook.md](references/mcp-tool-playbook.md)** — tool order and editing flow

See **[README.md](README.md)** for overview and usage.
