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

## 5. Commit-on-Green Finish

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
