---
name: cadforge
description: >
  Use this skill when the user needs cadforge workflows for declarative CAD as code in TOML
  (.cf): creating projects, live-previewing in the browser (serve), compiling to DXF,
  validating layers/constraints, generating PNG/SVG previews for visual verification,
  importing DXF, formatting .cf files, or running watch mode. Activate for prompts
  mentioning cadforge commands (new/init/serve/build/check/layers/preview/schema/fmt/
  watch/import/view), project.toml, .cf primitives, AGENTS.md in a CAD project, or
  CAD-as-code drawing pipelines.
license: MIT
metadata:
  author: jheison.martinez
  version: "0.6.0"
  framework: OpenCode
  category: cli-tool
  last_updated: "2026-06-12"
---

# cadforge skill

cadforge is a Rust CLI + library for CAD as code. Geometry is
declared in `.cf` TOML layers orchestrated by `project.toml`, compiles
deterministically to DXF, and renders faithful previews (real text, measured
dimension labels, hatches) so both humans and AI agents can *see* the drawing.

## When to use

Use this skill when the task involves:
- creating or initializing a cadforge project
- editing `.cf` primitives and validating the project
- live-previewing while editing (`cadforge serve`)
- compiling layers to DXF or generating PNG/SVG previews
- visually verifying edits with entity highlights
- importing an existing DXF into `.cf` layers
- implementing or debugging cadforge CLI/library features

## The vibecoding loop (human watching)

```bash
cadforge new mi-proyecto && cd mi-proyecto
cadforge serve --open          # live browser preview on http://127.0.0.1:4377
# edit .cf files — the browser refreshes ~80ms after every save;
# parse errors appear as an overlay instead of breaking the loop
```

Viewer features:
- scroll = zoom, drag = pan, double-click / `F` = fit, `Esc` = deselect
- **click any entity** → inspector panel with its source TOML block and
  copy buttons (`copy for agent` produces a ready-made targeted-edit prompt)
- layer panel (or keys `1`-`9`): cycle each layer on → ghost → off; ghost is
  ideal for tracing one floor plan over another
- `3D` button (or key `3`): stack the layers in a 3D exploded view

`serve` renders SVG only; run `cadforge build` when you need the DXF.

## The agent feedback loop (autonomous editing)

1. `cadforge schema` — dump the complete `.cf` language reference (markdown).
   Every project scaffolded by `cadforge new` also contains it as `AGENTS.md`.
2. Edit `.cf` files.
3. `cadforge check --json` — machine-readable validation: layers, entity
   counts, constraint violations, `strict` flag. Non-zero exit if strict+violations.
4. `cadforge preview` — writes `preview.png` (faithful render) and
   `preview.meta.json` mapping every entity id to world **and pixel** bounding
   boxes (plus text content). **Look at the image** to verify the result.
5. `cadforge preview --highlight ln-001,tx-002` — re-render with labeled amber
   dashed markers around those ids to confirm an edit landed where intended.
   Unknown ids are ignored silently.

## Core commands

```bash
cadforge new <name>                 # scaffold project + AGENTS.md
cadforge init                       # scaffold into current directory
cadforge serve [--open] [--port N]  # live preview server (default port 4377)
cadforge build [--layer <name>] [--output <file>] [--check]
cadforge check [--json]
cadforge layers [--json]
cadforge schema                     # full .cf reference for agents
cadforge preview [--width <px>] [-H <px>] [--layer <name>]
                 [--format png|svg|all] [--highlight id1,id2]
cadforge fmt [--check]
cadforge watch                      # auto-rebuild DXF on save
cadforge import <file.dxf> [--layer <name>]
cadforge view [--layer <name>]      # open DXF in external viewer
cadforge config set|show            # global defaults (author, units)
```

All project commands accept `--path <dir>` to run outside the project directory.

## .cf format essentials

Primitives: `line`, `polyline`, `rect`, `circle`, `arc`, `text`, `point`,
`dim`, `hatch`, `fill`, `group`, plus construction tools `array` (linear /
polar — spiral stairs, gears, repeated columns) and `mirror`. Run
`cadforge schema` for every attribute — do not guess fields from memory.

```toml
[layer]                 # optional per-file metadata
name = "muros"
color = "#FFFFFF"
line_weight = 0.35

[[line]]
id = "ln-001"           # ids enable hatch boundaries, belongs_to, --highlight
from = [0.0, 0.0]
to = [8.5, 0.0]
weight = 0.50

[[polyline]]
id = "pl-001"
points = [[0.0, 0.0], [5.0, 0.0], [5.0, 3.0], [0.0, 3.0]]
closed = true

[[text]]
position = [4.0, 3.0]
content = "SALA"
size = 0.25             # world units, not points
align = "center"

[[dim]]                 # measured distance is labeled automatically in render
from = [0.0, 0.0]
to = [8.5, 0.0]
offset = -0.8           # sign picks the side
text_size = 0.25        # label height in world units (optional)
precision = 2           # decimals (default 2)
show_units = true       # append project units (default true)

[[hatch]]
boundary = "pl-001"     # id of a closed polyline/rect in the same file
pattern = "ansi31"      # ansi31..34 | solid | none

[[array]]               # polar array: spiral stair / gear teeth
target = "pl-huella"    # or targets = [...]; copies get ids id@1, id@2, …
mode = "polar"          # polar | linear (linear uses offset = [dx, dy])
count = 16              # total instances including the original
center = [0.0, 0.0]
step_angle = 22.5

[[mirror]]              # mirrored copies get ids id@m
targets = ["pl-nave", "ar-puerta"]
axis = [[2.5, 0.0], [2.5, 1.0]]
```

## project.toml essentials

```toml
[project]
name = "mi-proyecto"
units = "m"             # used in dimension labels
strict = false          # true: constraint violations fail the build

[layers]                # order = draw order (later on top)
muros = { file = "muros.cf", locked = false }
cotas = { file = "cotas.cf", locked = false }

[constraints]           # optional
cotas.parent = "muros"          # child bbox inside parent bbox
cotas.belongs_to = "muros"      # children reference parent ids via belongs_to
```

## Gotchas

- `[[polyline]]` uses `points`, not `vertices`. Coordinates are floats, Y grows up.
- `preview` height short flag is `-H` (`-h` is help). `--width/-H` are a fit box;
  the image keeps the content aspect ratio.
- `text.size` is in world units (e.g. `0.25` for a 25cm-tall label at units = m),
  not font points.
- `cadforge check` validates only; `build --check` is equivalent. Neither writes DXF.
- Generated artifacts (`output.dxf`, `preview.png`, `preview.svg`,
  `preview.meta.json`) are gitignored — never hand-edit them.
- Text rendering uses an embedded DejaVu Sans Mono: previews are deterministic and
  work in containers with no system fonts.
- `serve` watches `.cf`/`.toml` only and renders SVG; it does not write DXF.
- `[[array]]`/`[[mirror]]` expand at build time; generated copies have ids
  like `base@3` / `base@m`. To change all copies, edit the base entity or the
  array/mirror block — generated ids do not exist in the source files.
- 2D geometry only (plan views) — no 3D kernel; the viewer's 3D mode is a
  visual stack of 2D layers. For constraints/import edge cases, check
  `cadforge-spec.md` in the repo before promising behavior.
