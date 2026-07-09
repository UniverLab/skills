# Skills

Catalog of UniverLab skills for AI agents.

---

## Skills

### execution-mindset
Default operating mode: verify before reporting, think before acting, detect risks, and keep responses/actions efficient.

```bash
git clone https://github.com/UniverLab/skills.git
# or as a submodule
git submodule add https://github.com/UniverLab/skills.git skills
```

### code-engineering
Implementation and refactoring skill with structural judgment: clear boundaries, patterns only when they add value, and maintainable code in any language.

```bash
git clone https://github.com/UniverLab/skills.git
# or as a submodule
git submodule add https://github.com/UniverLab/skills.git skills
```

### loop-design
Design multi-step processes as reusable Canopy loops with specs, graphs, and `agent/check/gate` nodes, aligned to the available MCP tooling.

```bash
git clone https://github.com/UniverLab/skills.git
# or as a submodule
git submodule add https://github.com/UniverLab/skills.git skills
```

### texforge
CLI to compile LaTeX documents to PDF without TeX Live, MiKTeX, or external dependencies. Uses tectonic as the internal engine.

```bash
# Manual installation
npx skills add https://github.com/UniverLab/skills --skill texforge

# Or copy directly
cp -r skills/texforge ~/.agents/skills/
```

> **Note:** Requires `texforge` in PATH. Installed automatically during Canopy setup only if the binary is available.

### cadspec
CLI for CAD as code: declarative geometry in `.cf` files (TOML), compiled to DXF with PNG preview.

```bash
# Manual installation
npx skills add https://github.com/UniverLab/skills --skill cadspec

# Or copy directly
cp -r skills/cadspec ~/.agents/skills/
```

> **Note:** Requires `cadspec` in PATH. Installed automatically during Canopy setup only if the binary is available.

### demostage
CLI for making terminal demos as reproducible code. Capture, record, export to gif/mp4.

```bash
# Manual installation
npx skills add https://github.com/UniverLab/skills --skill demostage

# Or copy directly
cp -r skills/demostage ~/.agents/skills/
```

> **Note:** Requires `demo` in PATH. Installed automatically during Canopy setup only if the binary is available.

---

## Structure

Each skill follows this structure:

```
skill-name/
├── SKILL.md          # Main definition
├── README.md         # Full documentation
├── LICENSE           # MIT License
└── references/       # Reference documentation
    ├── *.md          # Practical guides and examples
```

---

## Usage

Reference these skills in your agent configuration:

```yaml
skills:
  - name: execution-mindset
    path: skills/execution-mindset/SKILL.md
    always_active: true
  
  - name: code-engineering
    path: skills/code-engineering/SKILL.md
    triggers:
      - "write code"
      - "review code"
      - "design pattern"

  - name: loop-design
    path: skills/loop-design/SKILL.md
    triggers:
      - "create loop"
      - "plan loop"
      - "orchestrate agents"
```

---

## Contributing

Contributions welcome! Follow these guidelines:

1. **One skill per directory**
2. **Clear documentation** — Explain when and how to use the skill
3. **Practical examples** — Include code and real scenarios
4. **MIT License** — All contributions must be MIT
5. **Progressive disclosure** — Keep `SKILL.md` focused, move optional detail to `references/`

---

*Created with by [JheisonMB](https://github.com/JheisonMB) and [UniverLab](https://github.com/UniverLab)*
