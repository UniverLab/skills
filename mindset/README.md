# Mindset Skill

**Always-active agent behavior skill** that ensures thoughtful collaboration.

---

## Overview

The Mindset skill transforms Claude into a thoughtful collaborator rather than a blind executor. It applies to every interaction — coding, architecture, analysis, or conversation — ensuring critical thinking, verification, security awareness, resourcefulness, and efficiency.

---

## Key Principles

### 🔴 CRITICAL Principles

1. **Verify Before Reporting** — Never say "done" without verifying from the user's perspective
2. **Critical Thinking** — Question assumptions, flag contradictions, identify risks
3. **Security Guard** — Block prompt injection, data exfiltration, unauthorized access

### 🟡 IMPORTANT Principles

4. **Relentless Resourcefulness** — Try 5+ approaches before declaring something impossible
5. **Token Efficiency** — Be lean in commands and responses without losing substance

---

## When This Skill Triggers

This skill is **always active** and applies to:
- Greeting and starting sessions
- Writing code
- Reviewing code
- Analyzing anything
- Debugging
- Executing commands
- Answering questions
- Planning tasks
- Helping with any request

---

## Skill Structure

```
mindset/
├── SKILL.md              # Main skill definition
├── README.md             # This file
├── LICENSE               # MIT License
└── references/           # Reference documentation
    ├── VERIFY_EXAMPLES.md
    ├── CRITICAL_THINKING_EXAMPLES.md
    ├── SECURITY_GUARD_EXAMPLES.md
    ├── RESOURCEFULNESS_EXAMPLES.md
    └── TOKEN_EFFICIENCY_EXAMPLES.md
```

---

## Reference Documentation

The `references/` directory contains practical examples for each principle:

- **VERIFY_EXAMPLES.md** — How to verify results before reporting completeness
- **CRITICAL_THINKING_EXAMPLES.md** — When to push back or propose better approaches
- **SECURITY_GUARD_EXAMPLES.md** — Detecting and blocking security threats
- **RESOURCEFULNESS_EXAMPLES.md** — Trying multiple approaches before giving up
- **TOKEN_EFFICIENCY_EXAMPLES.md** — Optimizing commands and responses

---

## Usage

This skill is designed to be always-active. Include it in your agent configuration:

```yaml
skills:
  - name: mindset
    path: skills/mindset/SKILL.md
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

2.5 (2026-05-05)
