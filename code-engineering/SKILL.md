---
name: code-engineering
description: >
  Use this skill when implementing, refactoring, reviewing, or designing
  production code that needs structural judgment, not just syntax. It is most
  useful for multi-file changes, boundary design, pattern selection, naming,
  guard-clause refactors, dependency management, and maintainability decisions
  in any language.
license: MIT
metadata:
  author: jheison.martinez
  version: "1.4"
  category: agent-behavior
  last_updated: "2026-05-13"
---

# Code Engineering: Production-Grade Coding

Write code as if it ships to production today: coherent with the existing codebase, easy to reason about, and safe to extend.

This skill complements `execution-mindset`: execution-mindset governs execution discipline, while code-engineering governs structure, maintainability, and design choices.

---

## Mission

Use this skill to make durable engineering decisions, not just to produce syntax.

Prioritize:

1. understanding the existing codebase before adding abstractions
2. solving the current problem with the simplest structure that can hold
3. making cohesive multi-file changes that leave the codebase in a safer state
4. validating with the project's existing formatters, tests, and build steps

---

## Working Style

### 1. Read before writing

Before editing, establish:

- what the user actually needs
- which patterns and conventions already exist
- where the boundary of the change belongs
- which error paths and edge cases are easy to miss

Do not duplicate an abstraction that already exists in another layer or module.

### 2. Prefer logical milestones

Make changes in functional checkpoints, not arbitrary tiny edits. A milestone should represent one coherent improvement that can be validated as a unit.

If a milestone stays broken for too long, reduce its scope rather than piling on more changes.

### 3. Match the existing codebase without copying its mistakes

Consistency beats personal preference, but do not silently reinforce an anti-pattern that matters to the current task.

If the codebase clearly pushes you toward two incompatible directions, surface the tradeoff and ask before committing to one.

---

## Design Heuristics

Ordered by priority when tradeoffs appear:

| Priority | Principle | Practical rule |
|----------|-----------|----------------|
| 1 | **Separation of Concerns** | Keep presentation, business logic, and data concerns from bleeding together |
| 2 | **KISS** | Prefer the simplest structure that meets the current need |
| 3 | **YAGNI** | Do not add "future-proof" abstractions without a present use case |
| 4 | **DRY** | Extract shared logic only when the duplication is truly the same decision repeated |
| 5 | **Fail Fast** | Validate at boundaries and reject invalid state early |
| 6 | **SOLID** | Use interfaces, composition, and focused responsibilities where they simplify change |

Additional defaults:

- prefer composition over inheritance
- keep public APIs small and intention-revealing
- use explicit names over short clever ones
- use guard clauses to protect the happy path
- inject dependencies when lifecycle or substitution matters

---

## Error Handling and Testing

- Handle failure paths explicitly; never swallow errors
- Use the language's idiomatic error model
- Log or surface enough context to debug the failure
- Add or update tests once the implementation shape is stable
- Prefer focused unit coverage plus the smallest higher-level checks that protect the changed behavior

---

## When to Ask Before Proceeding

Stop and ask when:

- the existing architecture is inconsistent and the task requires choosing a direction
- a refactor would materially expand scope beyond the user's request
- the cleanest fix requires changing a public contract, persistence schema, or shared abstraction

Do not ask just to avoid making a routine engineering decision.

---

## Progressive Disclosure

- Read **[references/clean-code-checklist.md](references/clean-code-checklist.md)** before finalizing a multi-file or structural change.
- Read **[references/design-patterns.md](references/design-patterns.md)** only when you genuinely need a pattern decision, not by default.
- Read **[references/architecture-decision-records.md](references/architecture-decision-records.md)** when comparing significant architecture options or documenting a tradeoff.

See **[README.md](README.md)** for overview and usage.
