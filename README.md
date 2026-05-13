# Skills 🧠

Catálogo de habilidades de UniverLab para agentes de IA

---

## ✨ Habilidades

### execution-mindset
Modo operativo por defecto: verifica antes de reportar, piensa antes de actuar, detecta riesgos y mantiene respuestas/acciones eficientes.

```bash
git clone https://github.com/UniverLab/skills.git
# o como submodulo
git submodule add https://github.com/UniverLab/skills.git skills
```

### code-engineering
Skill de implementacion y refactor con criterio estructural: limites claros, patrones solo cuando aportan, y codigo mantenible en cualquier lenguaje.

```bash
git clone https://github.com/UniverLab/skills.git
# o como submodulo
git submodule add https://github.com/UniverLab/skills.git skills
```

### workflow-design
Disena procesos multi-step como workflows reutilizables de Canopy con specs, grafos y nodos `agent/check/gate`, ajustados al tooling MCP disponible.

```bash
git clone https://github.com/UniverLab/skills.git
# o como submodulo
git submodule add https://github.com/UniverLab/skills.git skills
```

---

## 📦 Estructura

Cada habilidad sigue esta estructura:

```
skill-name/
├── SKILL.md          # Definición principal
├── README.md         # Documentación completa
├── LICENSE           # Licencia MIT
└── references/       # Documentación de referencia
    ├── *.md          # Guías prácticas y ejemplos
```

---

## 🔧 Uso

Referencia estas habilidades en tu configuración de agente:

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

  - name: workflow-design
    path: skills/workflow-design/SKILL.md
    triggers:
      - "create workflow"
      - "plan workflow"
      - "orchestrate agents"
```

---

## 📝 Contribuir

Contribuciones bienvenidas! Sigue estas guías:

1. **Una habilidad por directorio**
2. **Documentación clara** — Explica cuándo y cómo usar la habilidad
3. **Ejemplos prácticos** — Incluye código y escenarios reales
4. **Licencia MIT** — Todas las contribuciones deben ser MIT
5. **Progressive disclosure** — Mantén `SKILL.md` enfocado y mueve detalle opcional a `references/`

---

*Created with ❤️ by [JheisonMB](https://github.com/JheisonMB) and [UniverLab](https://github.com/UniverLab)*
