# canopy-sync

Tooling skill for coordinating alongside other Canopy sessions.

The old sync protocol (mission declaration, workspace status, broadcast and
report-status tools) is retired. This skill teaches what replaced it:
`session_list` to see every live session, `scheduled_prompts` to wake one
later, and `intelligence_upsert` / `intelligence_search` to persist and find
shared findings. Durable history stays readable through
`intelligence_get_context` and the TUI's activity panel.

**Requires:** the Canopy MCP server in the session (`requires = "canopy"` in
`skills.toml`). Without those tools this skill is inert — don't install it
globally on hosts that never run Canopy.

Extracted from `execution-mindset` v4.0, which mixed behavior rules with
Canopy tooling. Behavior lives in [execution-mindset](../execution-mindset/);
the intelligence layer lives in [canopy-intelligence](../canopy-intelligence/).

## Contents

- [SKILL.md](SKILL.md) — the replacement table, the action/risk protocol
  table, core rules, and anti-patterns.

## License

MIT — see [LICENSE](LICENSE).
