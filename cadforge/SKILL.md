---
name: cadforge
description: >
  Use this skill when the user needs cadforge workflows for declarative 2D CAD in TOML
  (.cf): creating projects, compiling to DXF, checking layers/geometry, formatting .cf
  files, generating previews, or running watch mode. Activate for prompts mentioning
  cadforge commands (new/init/build/check/layers/preview/fmt/watch), project.toml,
  .cf primitives, or Architecture as Code drawing pipelines.
license: MIT
metadata:
  author: jheison.martinez
  version: "0.3.0"
  framework: OpenCode
  category: cli-tool
  last_updated: "2026-06-02"
---

# cadforge skill

cadforge is a Rust CLI + library for Architecture as Code in 2D CAD. The project
uses `.cf` TOML layers plus `project.toml`, compiles to DXF, and can render PNG preview
artifacts for agent workflows.

## When to use

Use this skill when the task involves:
- creating or initializing a cadforge project
- editing `.cf` primitives and validating the project
- compiling layers to DXF or generating preview assets
- implementing or debugging cadforge CLI/library features

## Default command flow

```bash
cadforge new mi-proyecto
cd mi-proyecto
cadforge fmt
cadforge check
cadforge build
cadforge preview
```

For iterative work:

```bash
cadforge watch
```

## Core commands

```bash
cadforge new <name>
cadforge init
cadforge build [--layer <name>] [--output <file>]
cadforge check
cadforge layers
cadforge preview [--width <px> --height <px>] [--layer <name>]
cadforge fmt [--check]
cadforge watch
```

## .cf primitives currently supported

Files use TOML with array-of-tables for primitives:

```toml
[layer]
name = "muros"
color = "#FFFFFF"

[[line]]
id = "ln-001"
from = [0.0, 0.0]
to = [8.5, 0.0]
weight = 0.50

[[rect]]
id = "rc-001"
origin = [1.0, 1.0]
width = 3.5
height = 4.0

[[circle]]
id = "ci-001"
center = [4.0, 3.0]
radius = 0.5

[[arc]]
id = "ac-001"
center = [2.0, 2.0]
radius = 0.9
from_angle = 0.0
to_angle = 90.0

[[polyline]]
id = "pl-001"
points = [[0, 0], [5, 0], [5, 3], [0, 3]]
closed = true

[[text]]
id = "tx-001"
position = [4.0, 3.0]
content = "SALA"
size = 0.2

[[dim]]
id = "dm-001"
from = [0, 0]
to = [5, 0]
offset = 0.5
```

`line`, `polyline`, `rect`, `circle`, `arc`, `text`, `point`, `dim`, `hatch`, `fill`, `group`

## project.toml

```toml
[project]
name = "mi-proyecto"
scale = "1:100"
units = "m"

[layers]
muros = { file = "muros.cf", locked = false }
puertas = { file = "puertas.cf", locked = false }
mobiliario = { file = "mobiliario.cf", locked = false }
cotas = { file = "cotas.cf", locked = false }
```

## Gotchas

- `.cf` uses `points` in `[[polyline]]` (not `vertices`).
- `cadforge check` validates and reports; it does not generate DXF output.
- `cadforge preview` writes `preview.png` and a metadata JSON sidecar.
- The current implementation is centered on 2D primitives; avoid assuming full 3D kernel flows.
- For constraints/import/viewer roadmap items, consult the project spec before promising behavior.
