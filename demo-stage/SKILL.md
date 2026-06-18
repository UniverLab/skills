---
name: demo-stage
description: >
  CLI tool for making terminal demos as reproducible code ("Demos as Code"). Use this skill whenever the
  user wants to record, normalize, validate or export a terminal demo, author a demo.toml score, produce a
  cast/html/gif/mp4 of a CLI session, or compose a terminal pane next to a browser/PDF pane. Activate when
  the user mentions "demo record", "demo normalize", "demo check", "demo export", demo.toml, macro.raw.toml,
  recording a CLI demo, asciinema/gif/mp4 of a terminal, DemoStage, or a multi-scene terminal+browser demo.
license: MIT
metadata:
  author: jheison.martinez
  version: "0.1"
  framework: OpenCode
  category: cli-tool
  last_updated: "2026-06-18"
---

# DemoStage CLI (`demo`)

Turns terminal demos into reproducible engineering. Records a session as **events**, normalizes human
imperfections into a clean **score** (`demo.toml`), and compiles that score to `cast`, `html`, `gif` or
`mp4`. A demo becomes a version-controlled, re-runnable file. Heavy tools are auto-provisioned
tectonic-style: `mp4` fetches a managed **ffmpeg** and browser panes fetch **Chromium** on first use (a
system install is used if present).

## Installation

```bash
# Linux / macOS
curl -fsSL https://get.univerlab.org/demo-stage | sh
# or from source
cargo install --path .
```

## The loop

```
demo record ──> macro.raw.toml ──> demo normalize ──> demo.toml ──> demo check ──> demo export
```

| Command | Does |
|---|---|
| `demo record [-o macro.raw.toml] [--idle-timeout-ms N] [--shell S]` | Capture an interactive PTY session. Stop with `exit`/Ctrl-D or on idle. |
| `demo normalize [in] [-o demo.toml] [--seed N] [--typing-ms 80] [--salt-ms 15]` | Prune typos, humanize typing, trim idle → clean score. |
| `demo check [demo.toml]` | Static validation. Exit 0 valid, 1 invalid. |
| `demo export [demo.toml] --target <cast\|html\|gif\|mp4> [-o PATH]` | Compile the score. |

You can skip record/normalize and **author `demo.toml` by hand**, then `check` + `export`.

## demo.toml (the DSL)

```toml
[demo]
name = "my-demo"
output_dir = "./dist"

[typing]            # optional: humanized typing for human_salt steps
base_ms = 80
salt_ms = 15
seed = 42           # set for reproducible output

[layout]
width = 1280
height = 720
fps = 15
background = "#0b0f14"

  [[layout.panes]]
  id = "console"
  type = "terminal"   # "terminal" | "browser"
  x = 0
  y = 0
  width = 1280
  height = 720
  font_size = 16

[[timeline]]
action = "focus"
pane = "console"

[[timeline]]
action = "type"
text = "cargo build --release"
human_salt = true

[[timeline]]
action = "keypress"
key = "enter"          # enter, tab, esc, up/down/left/right, ctrl+c, …

[[timeline]]
action = "wait_for_stdout"
match = "Finished"

[[timeline]]
action = "terminate"
```

Timeline actions: `focus` (set active pane), `type` (`text`, `human_salt?`), `keypress` (`key`), `wait`
(`duration_ms`), `wait_for_stdout` (`match`, `pane?`), `scroll` (browser: `direction`, `duration_ms`),
`terminate`.

## Multi-scene (terminal + browser)

Add a `browser` pane (a PDF viewer or live web preview) beside the terminal; `gif`/`mp4` composite both.
`cast`/`html` are text-only and reject browser panes.

```toml
  [[layout.panes]]
  id = "preview"
  type = "browser"
  x = 640
  y = 0
  width = 640
  height = 720
  url = "file:///tmp/out/main.pdf"   # required for browser panes

[[timeline]]
action = "focus"
pane = "preview"
[[timeline]]
action = "scroll"
direction = "down"
duration_ms = 3000
```

## Notes for agents

- Always `demo check` a score before `demo export`; fix every reported problem.
- For a clean recording, DemoStage forces `PS1='$ '` during export, so demos never leak `user@host`.
- `cast`/`html`/`gif` are pure-Rust and offline. `mp4` auto-fetches ffmpeg; browser panes auto-fetch
  Chromium (or use a system install). If a download is blocked, install the tool via the package manager.
- Scrolling Chrome's built-in **PDF** viewer is best-effort; for reliable PDF paging prefer a web preview or
  page fragments. Web pages scroll normally.
- Use `[typing].seed` whenever the output must be byte-stable (CI, golden files).
