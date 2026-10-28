# canopy-capabilities

Orientation skill: answers "what can I do here?" for a Canopy workspace.

A capability MAP (memory, background agents, graphs, sessions, TUI, RAG, spec
backlogs) with starter ideas and a need→capability table — deliberately not a
static tool inventory. It teaches the agent to discover the live surface
(`get_tools`, `canopy --help`, `harness` `models`, `node` `list`) so the skill
doesn't rot when the tool surface evolves.

**Requires:** the Canopy MCP server (`requires = "canopy"` in `skills.toml`).

Companions: [canopy-intelligence](../canopy-intelligence/),
[canopy-graph-design](../canopy-graph-design/).

## License

MIT — see [LICENSE](LICENSE).
