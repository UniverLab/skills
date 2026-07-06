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
  version: "1.5"
  framework: Canopy
  category: loop-orchestration
  last_updated: "2026-07-05"
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
