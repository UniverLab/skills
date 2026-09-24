# graph-reviewer

**Audit skill** for reviewing a Canopy graph before it spends real quota — the
graph, the specs it will execute, and how it allocates expensive models.

Part of the graph-orchestration family:

- [canopy-graph-design](../canopy-graph-design/) — how to build a graph
- **graph-reviewer** — how to audit one before running it

## Why it exists

A wrong graph does not fail cheaply. It fails after twenty minutes of agent
time, on the third spec, having already spent the budget it was meant to
protect. Every check in this skill comes from a graph that shipped work or a
graph that burned quota.

## What it covers

- **Graph** — one entry, nothing unreachable, `pass`/`fail` on every agent
  node, balanced lock/unlock across escalation paths.
- **Specs** — decides rather than asks, one repository each, seven sections,
  what-and-how over history, no unsourced claims.
- **Model economy** — where the expensive model sits, what one rejection
  costs, `resume`, commit rights, context groups, pinned skills.
- **Prompts** — finishing the turn, continuing from work on disk, carrying
  context across hops, self-termination hazards.
- **Timeouts and failure modes** — quota versus rejection, honest handling of
  a bare timeout.

## Contents

- [SKILL.md](SKILL.md) — the audit and the report format

## License

MIT — see [LICENSE](LICENSE).
