---
name: texforge
description: >
  Compile LaTeX to PDF with the texforge CLI, without TeX Live or MiKTeX. The full skill now ships inside
  the texforge binary: run `texforge skill` to print it. Use when the user mentions texforge, compiling
  .tex files, LaTeX templates, project.toml, or spell-checking and linting a LaTeX document.
license: MIT
metadata:
  author: jheison.martinez
  version: "2.0"
  category: cli-tool
  last_updated: "2026-11-17"
---

# texforge

This skill now lives in the texforge binary, so it always matches the installed version.

1. Print it and follow it: `texforge skill`
2. Not installed yet? Get texforge (Linux, macOS and Windows installers, binaries and `cargo install`):
   https://github.com/UniverLab/texforge/releases/latest — install steps in
   https://github.com/UniverLab/texforge#install

Releases before 1.0 do not have the `skill` command; update with `texforge update`
or reinstall from the link above, then run `texforge skill`.

This file is deprecated and no longer carries the command reference. Do not copy
instructions from older versions of it.
