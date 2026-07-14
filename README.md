# Skills

Catalog of UniverLab skills for AI agents.

The catalog is organized in three families, designed for **dynamic, on-demand
loading**: skills are narrow and single-purpose so an agent (or a Canopy node
pin) pulls only what the task needs. A Rust-loop skill in a Python session is
context rot — install less, pin per role.

---

## Behavior family

How the agent operates. `execution-mindset` is always on; the other two stack
on top per role.

### execution-mindset
Default operating mode: zero-indulgence clarity, interview when uncertain,
the systemic resolution model (model → locate divergence → smallest decisive
move → close the loop), verification before reporting, and token efficiency.

### architect-mindset
Behavior for design work: boundaries as contracts with violation signals,
design for the process dying mid-step, reversible-vs-expensive decisions,
and specs written for the cheapest executor.

### code-engineering
Behavior for code work: structural judgment, pattern selection with restraint,
maintainable multi-file changes, and debugging as the resolution model applied
to code.

```bash
git clone https://github.com/UniverLab/skills.git
# or as a submodule
git submodule add https://github.com/UniverLab/skills.git skills
```

---

## Canopy tooling family

Only installed where the `canopy` binary exists — these are inert without the
Canopy MCP server.

### canopy-intelligence
The Project Intelligence Layer: pull workspace context at session start,
persist facts/patterns/summaries with `intelligence_upsert`.

### canopy-sync
The action-driven workspace protocol: classify actions by risk and follow the
declare → execute → report sequence.

### loop-design
Design multi-step processes as reusable Canopy loops: the ROLE/WHAT/HOW spec
contract, design-expensive/execute-cheap model selection, graph patterns, and
the MCP tool playbook.

---

## Tool skills

Bound to their CLI; installed automatically during Canopy setup only if the
binary is available.

### texforge
CLI to compile LaTeX documents to PDF without TeX Live, MiKTeX, or external
dependencies. Uses tectonic as the internal engine. Requires `texforge`.

### cadspec
CLI for CAD as code: declarative geometry in `.cf` files (TOML), compiled to
DXF with PNG preview. Requires `cadspec`.

### demostage
CLI for making terminal demos as reproducible code. Capture, record, export
to gif/mp4. Requires `demo`.

```bash
# Manual installation of a single skill
npx skills add https://github.com/UniverLab/skills --skill texforge

# Or copy directly
cp -r skills/texforge ~/.agents/skills/
```

---

## Pinning guide (Canopy loop nodes)

With Canopy's dynamic skills, pin per node role instead of installing
globally:

| Node role | Pin |
|---|---|
| Implementer | `execution-mindset` + `code-engineering` |
| Spec author / loop designer | `architect-mindset` + `loop-design` |
| Reviewer | `execution-mindset` (zero indulgence is the whole job) |
| Any Canopy-workspace session | `canopy-intelligence`, `canopy-sync` |

---

## Structure

Each skill follows this structure:

```
skill-name/
├── SKILL.md          # Main definition
├── README.md         # Full documentation
├── LICENSE           # MIT License
└── references/       # Reference documentation, loaded on demand
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

  - name: architect-mindset
    path: skills/architect-mindset/SKILL.md
    triggers:
      - "design"
      - "architecture"
      - "write specs"

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
6. **Narrow scope** — a skill that only applies next to a binary or MCP server declares it with `requires`

---

*Created with by [JheisonMB](https://github.com/JheisonMB) and [UniverLab](https://github.com/UniverLab)*
