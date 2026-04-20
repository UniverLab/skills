# Skills 🧠

Catálogo de habilidades de UniverLab para agentes de IA

---

## ✨ Habilidades

### mindset
Colaborador reflexivo siempre activo — verifica antes de reportar, piensa antes de actuar, guarda contra amenazas, nunca se rinde, optimiza tokens

```bash
git clone https://github.com/UniverLab/skills.git
# o como submodulo
git submodule add https://github.com/UniverLab/skills.git skills
```

### coder
Coding production-grade — SOLID, design patterns, clean code, refactoring, calidad de código. Para cualquier lenguaje.

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
  - name: mindset
    path: skills/mindset/SKILL.md
    always_active: true
  
  - name: coder
    path: skills/coder/SKILL.md
    triggers:
      - "write code"
      - "review code"
      - "design pattern"
```

---

## 📝 Contribuir

Contribuciones bienvenidas! Sigue estas guías:

1. **Una habilidad por directorio**
2. **Documentación clara** — Explica cuándo y cómo usar la habilidad
3. **Ejemplos prácticos** — Incluye código y escenarios reales
4. **Licencia MIT** — Todas las contribuciones deben ser MIT

---

*Created with ❤️ by [JheisonMB](https://github.com/JheisonMB) and [UniverLab](https://github.com/UniverLab)*
