# Coder Skill

**Production-grade coding principles** for writing clean, maintainable code.

---

## Overview

The Coder skill ensures Claude writes code as if it ships to production today. It focuses on SOLID principles, design patterns, clean code practices, and architectural decisions — complementing the Mindset skill which handles agent behavior.

---

## Key Principles

### Design Principles (by priority)

1. **SoC** — Separate concerns into layers
2. **DRY** — One source of truth
3. **YAGNI** — Build only what's needed now
4. **KISS** — Simplest solution that works
5. **Fail Fast** — Validate at boundaries
6. **SOLID** — Structural backbone
7. **Law of Demeter** — Don't reach through objects

### Code Quality

- **Naming** — Explicit, readable names (no acronyms)
- **Size limits** — Methods ≤ 30 lines, ≤ 3 nesting levels
- **Guard clauses** — Fail fast with negative conditions
- **Error handling** — Explicit, no swallowed errors
- **Testing** — Unit tests after functionality works

---

## When This Skill Triggers

This skill activates when:
- Writing code in any language
- Reviewing code
- Designing code architecture
- Selecting design patterns
- Refactoring existing code
- Making architectural decisions

---

## Skill Structure

```
coder/
├── SKILL.md              # Main skill definition
├── README.md             # This file
├── LICENSE               # MIT License
└── references/           # Reference documentation
    ├── design-patterns.md
    ├── clean-code-checklist.md
    └── architecture-decision-records.md
```

---

## Reference Documentation

The `references/` directory contains detailed guides:

- **design-patterns.md** — Decision trees for pattern selection
- **clean-code-checklist.md** — Pre-submission verification checklist
- **architecture-decision-records.md** — ADR templates and examples

---

## Usage

Include in your agent configuration for coding tasks:

```yaml
skills:
  - name: coder
    path: skills/coder/SKILL.md
    triggers:
      - "write code"
      - "review code"
      - "design pattern"
      - "refactor"
      - "architecture"
```

---

## Design Pattern Decision Trees

### Object Creation
```
Need to create objects?
├─ One global instance? → Singleton (prefer DI)
├─ Multiple implementations? → Factory Method
├─ Families of related objects? → Abstract Factory
├─ Complex object? → Builder
└─ Cloning existing? → Prototype
```

### Structure
```
Need to organize objects?
├─ Incompatible interfaces? → Adapter
├─ Abstraction varies? → Bridge
├─ Tree hierarchy? → Composite
├─ Add behavior dynamically? → Decorator
├─ Simplify subsystem? → Facade
├─ Memory optimization? → Flyweight
└─ Control access? → Proxy
```

### Behavior
```
Need to manage communication?
├─ Notify state changes? → Observer
├─ Swap algorithms? → Strategy
├─ Encapsulate operations? → Command
├─ Behavior changes with state? → State
├─ Algorithm skeleton? → Template Method
├─ Request through handlers? → Chain of Responsibility
├─ Reduce object communication? → Mediator
├─ Capture/restore state? → Memento
├─ Traverse collection? → Iterator
├─ Add operations to hierarchy? → Visitor
└─ Interpret expressions? → Interpreter
```

---

## License

MIT © Jheison Martinez

---

## Author

Jheison Martinez ([@JheisonMB](https://github.com/JheisonMB))

---

## Version

1.2 (2026-05-05)
