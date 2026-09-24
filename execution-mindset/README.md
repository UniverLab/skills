# Execution Mindset Skill

Default operating mode for any task: size the response, catch indulgence by
its declared forms, interview when readings diverge, resolve by locating the
first divergence, search instead of guessing on things that change, verify
before reporting.

---

## Overview

The skill is a list of targets and one resolution loop, not a persona. A
skill only applies when it is loaded; what has to be present on every turn
belongs in the agent's instruction file (`CLAUDE.md` or equivalent) as the
same short list of targets.

---

## What it contains

1. **Size the response** — trivial / standard / complex, and what each gets.
2. **What indulgence looks like** — nine concrete defects, each observable: a spec that asks, a decision inside a spec, a bypass without a filed defect, a parked defect, a deducible question, a silent decision, praise before the flaw, dated advice from memory, "done" unverified.
3. **Interview** — when two readings mean different work: name the gap, cost the readings, recommend one with the reason, restate, execute.
4. **Search** — knowledge has a date; what moves yearly is looked up, not recalled.
5. **The resolution model** — model → first divergence → evidence → smallest decisive move → anomalies → broken contract → second-order effects → close the loop.
6. **Verify before reporting** and **tokens**.

This skill is pure behavior: Canopy-specific tooling (intelligence layer, sync
protocol) was extracted in v5.0 to [canopy-intelligence](../canopy-intelligence/)
and [canopy-sync](../canopy-sync/). Role behavior stacks on top:
[architect-mindset](../architect-mindset/) for design,
[code-engineering](../code-engineering/) for code.

---

## When This Skill Triggers

This skill is **always active** and applies whenever the agent must:
- decide how to approach a task
- align on ambiguous or underspecified requests
- verify results before reporting completion
- recover from failed attempts
- keep commands and responses lean

---

## Skill Structure

```
execution-mindset/
├── SKILL.md              # Main skill definition
├── README.md             # This file
├── LICENSE               # MIT License
└── references/           # Reference documentation
    ├── VERIFY_EXAMPLES.md
    ├── RESOURCEFULNESS_EXAMPLES.md
    └── TOKEN_EFFICIENCY_EXAMPLES.md
```

---

## Reference Documentation

The `references/` directory is loaded on demand:

- **VERIFY_EXAMPLES.md** — load when you need a stronger validation loop
- **RESOURCEFULNESS_EXAMPLES.md** — load after the first approach fails
- **TOKEN_EFFICIENCY_EXAMPLES.md** — load before verbose commands or long output

---

## Usage

This skill is designed to be always-active. Include it in your agent configuration:

```yaml
skills:
  - name: execution-mindset
    path: skills/execution-mindset/SKILL.md
    always_active: true
```

---

## License

MIT © Jheison Martinez

---

## Author

Jheison Martinez ([@JheisonMB](https://github.com/JheisonMB))

---

## Version

5.0 (2026-07-14)
