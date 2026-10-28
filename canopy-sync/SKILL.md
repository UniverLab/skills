---
name: canopy-sync
description: >
  Use this skill when the Canopy MCP server is available and you are about to
  act on a shared workspace: writing files, running builds or tests,
  installing dependencies, or committing. The old multi-agent sync protocol is
  retired; this skill covers what replaced it: session_list,
  scheduled_send_create, and intelligence_upsert/search.
license: MIT
metadata:
  author: jheison.martinez
  version: "2.0"
  framework: Canopy
  category: canopy-tooling
  requires: canopy
  last_updated: "2026-10-28"
---

# Canopy Coordination: Sessions and Memory, Not Sync

The multi-agent sync protocol is retired. Mission declaration, workspace
status, broadcast channels and their four MCP tools no longer exist — calling
them fails. What remains is durable: past missions and chatter stay in the
database, `intelligence_get_context` surfaces them, and the person's TUI
renders them.

Coordination is not a "multi-agent only" feature. Solo sessions still leave
history; parallel sessions still need to find each other and each other's
findings.

This is a **tooling skill**: it only applies when the Canopy tools are
present. Behavior rules live in `execution-mindset`.

---

## What replaced the sync tools

| You used to | Do this now |
|---|---|
| See who else is working | `session_list` — every live session with its id, workdir, name and whether a TUI is attached |
| Tell another agent to act later | `scheduled_send_create` — schedule a prompt for a session instead of polling it |
| Broadcast a milestone or status | `intelligence_upsert` a fact/session summary, so every future session reads it |
| Read others' declarations | `intelligence_get_context` at session start, `intelligence_search` for a specific topic |

There is no live broadcast and no mission lifecycle to open or close. Never
invent a `sync_*` call, and never wait for a peer's acknowledgement.

---

## Coordination protocol by action

| Action type | Risk | Protocol |
|---|---|---|
| Read files, search, analyze | `low` | Execute directly |
| Run tests, builds, linters | `medium` | Execute; `intelligence_upsert` the outcome if it is worth keeping |
| Write files, modify code | `high` | `intelligence_get_context` for known constraints → check `session_list` for a session in the same workdir → execute → `intelligence_upsert` what changed |
| Install deps, schema changes, commits | `breaking` | Same as `high`, and say it plainly in the upsert so the next session sees the migration/commit constraint |

Two sessions in the same workdir at once is a conflict risk you manage, not
one the daemon prevents: check `session_list` before a long write, and prefer
the session a change is already assigned to.

### get_tools — your action guide

`get_tools(scope)` returns the right tools + protocol for any situation:

- `get_tools(scope="session_start")` — what to do first
- `get_tools(scope="file_write", path="...")` — what applies before modifying
- `get_tools(scope="test_run")` — verification guidance
- `get_tools(scope="close_session")` — wrap up properly (this is where the
  session summary is written)
- `get_tools(scope="multi_agent")` — the coordination toolkit

---

## Core Rules

1. **Protocol by action, not by count.** The history matters even when you're
   alone.
2. **Non-blocking.** Nothing you call waits for a peer. Act on last-known
   state.
3. **Communicate intent, not implementation.** Upserts explain *what* and
   *why*, not *how*.
4. **Close the session with a summary.** `get_tools(scope="close_session")`
   tells you to `intelligence_upsert` the session summary — that is the
   replacement for the old report-status step.

---

## Anti-patterns

❌ Calling a retired `sync_*` tool, or a hand-rolled broadcast
✅ `session_list` + `intelligence_upsert` / `intelligence_search`

❌ Skipping the upsert because you think you're alone — you don't know who
reads this history next session
✅ Follow the action protocol: read context, act, persist what changed

❌ Fine-grained upserts for every file edit
✅ Coarse, durable entries: "Refactoring database layer, migration 12 drops
the retired tables"
