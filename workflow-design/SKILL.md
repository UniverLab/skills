---
name: workflow-design
description: >
  Use this skill when the user wants to turn a recurring or multi-step process
  into a reusable Canopy workflow, inspect or reshape an existing workflow
  graph, or coordinate background agent/check/gate flows with approvals and
  retries. Prefer reusable graph patterns over one-off pipelines, and build
  against the MCP workflow tools that actually exist in the environment.
license: MIT
metadata:
  author: jheison.martinez
  version: "1.2"
  framework: OpenCode
  category: workflow-orchestration
  last_updated: "2026-05-13"
---

# Workflow Design

Design generic workflows for Canopy as editable graphs, not as one-off scripts.

This skill exists for planning and authoring **background workflows** that users can later inspect, edit, and run from Canopy.

---

## Mission

Translate a user goal into:

1. ordered specs
2. a graph per spec
3. the right mix of `agent`, `check`, and `gate` nodes
4. a persisted workflow built with MCP tools

The output must stay generic enough that different users can plug in different CLIs, models, prompts, and verification commands.

---

## When This Skill Triggers

Use this skill when the user asks for any of the following:

- create or plan a workflow
- build a background pipeline
- chain multiple agents with checkpoints
- orchestrate developer/reviewer/verifier/committer flows
- create reusable specs for implementation, bug fixing, refactors, docs, or ops
- convert a manual process into a Canopy workflow graph
- refine an existing workflow graph
- turn a repeated manual process into a reusable workflow
- add approvals, retries, or verification gates around agent work

Do **not** reduce everything to one fixed "coding pipeline" unless the user explicitly wants that exact pattern.

---

## Core Rules

### 1. Model workflows as graphs, not lists

Each spec is an ordered work unit, but inside each spec you should think in graph form:

- `agent` — produces or analyzes work
- `check` — verifies with commands or deterministic checks
- `gate` — decides the next route based on prior output

If the workflow needs iteration, use edges and gates. If it is linear, keep it simple.

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

Before you create or run a workflow, clarify:

- whether creation should stay draft or also run
- what verification commands are authoritative
- whether commits are allowed inside the workflow
- whether blockers should pause or hard-fail the workflow

### 5. Reuse patterns, adapt prompts

Reuse graph patterns from the reference docs, but adapt:

- node prompts
- CLI/model selection
- verification commands
- retry / fail routing
- blocker behavior

---

## Workflow Construction Playbook

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

Use the Canopy workflow tools, in order when creating:

1. `workflow_create`
2. `workflow_add_spec`
3. `workflow_add_node`
4. `workflow_add_edge`
5. `workflow_get`

When refining existing workflows, inspect with `workflow_list` / `workflow_get` first.

The current toolset does **not** provide in-place `workflow_update*` mutations. Use one of these safe strategies instead:

1. extend the existing workflow with additional specs, nodes, or edges when the desired change is append-only
2. create a replacement workflow when the graph must be reshaped in ways the tools cannot mutate directly
3. summarize the migration path clearly so the user understands whether the old workflow stays in place

Only call `workflow_run` after the user has the chance to review the graph, unless they explicitly asked to launch it immediately.

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

1. workflow name
2. each spec in order
3. graph per spec
4. CLIs/models/checks chosen
5. what still requires user confirmation before `workflow_run`

---

## Progressive Disclosure

- Read **[references/workflow-patterns.md](references/workflow-patterns.md)** when choosing a graph shape for a spec.
- Read **[references/mcp-tool-playbook.md](references/mcp-tool-playbook.md)** immediately before creating, extending, or replacing a workflow through MCP tools.

---

## Reference Documentation

- **[references/workflow-patterns.md](references/workflow-patterns.md)** — reusable graph patterns
- **[references/mcp-tool-playbook.md](references/mcp-tool-playbook.md)** — tool order and editing flow

See **[README.md](README.md)** for overview and usage.
