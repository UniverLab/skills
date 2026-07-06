# Loop Patterns

Reusable graph patterns for Canopy loops.

---

## 1. Feature Delivery Loop

Use when implementing a new feature with review and verification.

```text
implementer --always--> verifier --pass--> reviewer --pass--> finish_gate
                              \--fail-----------------------> implementer
reviewer --fail---------------------------------------------> implementer
finish_gate(pass_route=next_spec)
```

Best for:

- feature work
- larger refactors
- multi-step backend/frontend changes

---

## 2. Focused Bugfix Loop

Use when the loop has a strong deterministic check.

```text
fixer --always--> targeted_test --pass--> finish_gate
                         \--fail--------> fixer
finish_gate(pass_route=next_spec)
```

Best for:

- regressions
- failing test repairs
- crash fixes

---

## 3. Review-Driven Authoring

Use when quality of reasoning matters more than a shell check.

```text
author --always--> reviewer --pass--> approval_gate
                      \--fail--------> author
approval_gate(pass_route=next_spec, fail_route=author)
```

Best for:

- docs
- prompts
- architecture proposals
- migrations needing explicit approval language

---

## 4. External Intake to Internal Execution

Use when a first node extracts or discovers tasks.

```text
extractor --pass--> planner --pass--> implementer --always--> verifier
                                           ^                    |
                                           |-------fail---------|
```

Best for:

- reading tasks from issue trackers
- processing backlog files
- consuming MCP-fed work queues

---

## 5. Multi-Platform Cross-Check

Use when you want a second CLI/model family to catch what the first one missed, instead of a same-vendor reviewer marking its own work.

```text
implementer(claude/opus) --always--> crosscheck(opencode/qwen) --pass--> finish_gate
                                            \--fail-------------------> implementer(claude/opus)
finish_gate(pass_route=next_spec) --fail--> implementer(codex)
```

- `implementer` — `agent` node on one platform/model (e.g. `platform: "anthropic", model: "claude-sonnet-5"`).
- `crosscheck` — `check`/`agent` node on a **different** platform (e.g. `platform: "opencode", model: "qwen3-coder"`), reviewing the implementer's diff against the spec.
- `finish_gate` — routes on the cross-check verdict; on repeated failure, escalate to a third platform (e.g. `platform: "codex"`) rather than looping the same pair forever.

Best for:

- security-sensitive changes where a same-vendor blind spot is a real risk
- verifying that a fix isn't overfit to one model's interpretation of the spec
- setups where the user has multiple CLI subscriptions and wants them cross-validating each other

Rules:

- never assume the cross-check platform matches the implementer's — that defeats the purpose
- keep the cross-check prompt narrow (diff + spec + "find what's wrong"), not a re-implementation
- cap escalation depth (e.g. 1 cross-check retry, then 1 third-platform escalation) so the loop can't spin forever between two disagreeing models

---

## 6. Commit-on-Green Finish

Use only when the user explicitly allows commits inside the loop.

```text
implementer -> verifier -> reviewer -> commit_ready_gate
commit_ready_gate(pass)-> committer -> next_spec
commit_ready_gate(fail)-> implementer
```

Rules:

- commit node must not run before verification
- commit prompts should include commit policy and checkpoint expectations
- if commits are not allowed, replace `committer` with a summary/report node
