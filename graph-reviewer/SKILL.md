---
name: loop-reviewer
description: >
  Use this skill to review a Canopy loop before it spends real quota: its
  graph, the specs it will execute, and how it allocates expensive models.
  Apply it when a loop is about to run for the first time, after reshaping a
  graph, when a queue has been assembled or extended, or when a loop is
  burning budget without finishing work. It audits; it does not implement.
license: MIT
metadata:
  author: jheison.martinez
  version: "1.0"
  framework: Canopy
  category: loop-orchestration
  last_updated: "2026-09-01"
---

# Loop Reviewer: Audit Before It Costs

A loop that is wrong does not fail cheaply. It fails after twenty minutes of
agent time, on the third spec, having already spent the budget you were
protecting. Every finding in this skill comes from a loop that shipped work
and a loop that burned quota, so review against it before running, not after.

You **audit**. You do not fix the graph, edit specs, or write code. Your
deliverable is a report.

---

## What You Are Given

A loop id, and usually a queue id. Read them through the MCP surface:
`loop_get` for the graph, `queue_list` and `spec_list` for the work. Large
results are saved to a file whose path the tool returns — read that file in
chunks until you have all of it.

**Never read the SQLite database directly.** If the surface cannot answer
something, that is itself a finding: report the missing capability instead of
working around it.

---

## 1 · The Graph

Structural defects are cheap to find and expensive to hit.

- **Exactly one entry point** — one node with no incoming edges.
- **No unreachable node.** Walk from the entry; anything unvisited is dead.
- **Every agent node has an outgoing edge for `pass` and for `fail`.** A
  missing `fail` edge kills the spec mid-run with a blocker that names the
  node and nothing else. This has happened.
- **Every terminal is deliberate.** A node with no outgoing edges must be the
  intended end of a spec, not an oversight.
- **Paired operations are balanced.** If the graph locks a repository, every
  path that reaches the committer must unlock, and every completed path must
  restore the lock. Check the escalation paths, not just the happy one.
- **A node that only reports `pass` is a decision, not a bug** — a resilience
  node whose `fail` means "stop for a human" is correct. Confirm it is
  deliberate rather than assuming either way.

---

## 2 · The Specs

Apply these to every spec the loop will execute. Quote the offending text.

- **A spec decides; it does not ask.** Any spec leaving a choice to the
  implementer is defective. Look for *decide*, *choose*, *pick one*,
  *whichever*, *either … or*, *to be decided*, or a requirement written as an
  open question. The implementer has less context than the author and will
  either guess or spend a review cycle asking.
  - **The exception:** a spec may instruct the executor to *establish a fact*
    or to *decide a narrow technical detail that only becomes knowable while
    implementing*. That is legitimate when the spec bounds it — a default, a
    criterion, or a tie-break. Flag those separately as bounded open points,
    and say whether the bound is actually there.
- **One repository per spec.** A loop run has a single workdir and a single
  committer. A spec whose work lives partly in another repository cannot be
  completed: the reviewer sees the deliverable outside the tree it can commit
  and refuses, correctly, every time. Flag any spec referencing paths or work
  outside the loop's workdir.
- **Seven sections present**: Objective, Functional Requirements,
  Non-Functional Requirements, Constraints, Guidelines, In Scope, Out of Scope.
- **What and how, not history.** A spec carries the decision, not the story of
  how it was reached. Narration and quoted dialogue belong in the knowledge
  store.
- **One language across the queue**, and the one the executing agents expect.
- **No unverified assumption stated as fact.** A file path, symbol, line
  number or behaviour asserted flatly, with no measurement or citation behind
  it, will be trusted and propagated. Flag claims that read as invented.
- **Acceptance is checkable.** Prefer a criterion someone can run over a
  description of intent.

---

## 3 · Model Economy

This is where loops actually fail, and the part most reviews skip.

- **Where does the expensive model sit?** It should receive work that has
  already passed a deterministic gate and a cheaper reviewer. An expensive
  model reading raw first-draft output is the most common waste.
- **What does one rejection cost?** If the expensive model writes a
  change-request that a different model applies, and then re-reviews it, each
  objection costs **two** expensive invocations and loses context between
  them. A node that reviews *and repairs* costs one. Measured on a real
  queue: the split arrangement grew 1 → 2 → 4 → 6 calls per spec; merging
  them held it at exactly 1.
- **Is `resume` set on the expensive node?** Without it, every bounce
  re-reads the diff from cold and pays for context it already had.
- **Does the expensive node hold commit rights it does not need?** Committing
  is mechanical. Spend the expensive budget on judgment.
- **Are cheap bounces staying cheap?** Gate failures and cheap-reviewer
  change-requests should return to a free implementer, never escalate.
- **Context groups**: specs touching the same files should share a group, so
  each resumes the previous one's warm session instead of re-reading the repo.
- **Pinned skills**: cheap nodes benefit from a role skill at no cost. The
  expensive node usually does not — it needs the diff, not instructions on
  how to reason, and every appended token is quota on the node you are
  protecting.

---

## 4 · Prompts

- **Does each prompt say how to finish?** A weak model that produces correct
  work and never reports it counts as a failure and the work is redone. If
  the harness requires a completion call, the prompt must demand it
  explicitly, and tell the agent to stop once the checks are green.
- **Does the implementer check whether the work is already done?** After a
  timeout or a restart, finished work often sits on disk. A prompt that does
  not say "read the tree first and continue from it" produces a rewrite.
- **Is context that must survive several hops carried by something durable?**
  If feedback only reaches the next node, anything the graph needs three hops
  later has to live in a file or be re-derived. Check that whatever the design
  assumes is actually available where it is read.
- **Are the destructive prohibitions present** where they matter — no
  committing from a non-committer, no pushing, no history rewriting?
- **Does any node risk killing its own run?** A loop working on a tool whose
  binary reconciles running state can have an agent terminate itself by
  building or launching that binary. If the loop targets such a repository,
  the prompts must forbid it.

---

## 5 · Timeouts and Failure Modes

- **Are timeouts realistic for the work?** A node timing out twice with good
  work on disk is a budget problem, not a capability problem — and the
  distinction changes the fix.
- **Is quota exhaustion distinguishable from a real rejection?** They must not
  take the same edge. A quota failure routed back to the implementer redoes
  work that was never wrong.
- **Is a bare timeout classified honestly?** It is not a recognised transient
  failure. Treating it as one sends the work straight back into the same wall.

---

## The Report

Order by severity, and quote evidence.

- **BLOCKING** — will fail or waste budget as written: graph defects,
  undecided specs, cross-repository specs.
- **EXPENSIVE** — will run, but costs more than it should: model placement,
  double-invocation review loops, missing `resume`, absent groups.
- **NEEDS REWRITE** — structural spec problems: missing sections, history
  instead of requirements, unsourced claims.
- **BOUNDED OPEN POINTS** — deliberate, with their bound named. Say whether
  the bound is adequate.
- **CLEAN** — a plain list.

Close with: how many specs and nodes were audited, how many blocking, and the
**single item you would fix first**, with the reason.

Do not pad. A false finding costs as much attention as a missed one, and
erodes trust in the next report.
