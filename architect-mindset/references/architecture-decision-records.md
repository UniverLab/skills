# Architecture Decision Records (ADR)

Templates and examples for documenting architectural decisions.

---

## What is an ADR?

An Architecture Decision Record (ADR) captures an important architectural decision along with its context and consequences.

**Benefits:**
- Preserves decision rationale over time
- Helps onboard new team members
- Prevents rehashing old decisions
- Provides audit trail for compliance

---

## ADR Template

```markdown
# [Short title is a few words]

## Status

[Proposed | Accepted | Deprecated | Superseded by [ADR-0005](0005-example.md)]

## Context

[Describe the problem space, the forces at play, including technological, political, social, and project local. These forces are probably in tension and should be called out as such. The language in this section is value-neutral.]

## Decision

[This is the meat of the ADR. Describe the response to these forces. It is written in full sentences, with active voice. "We will ..."]

## Consequences

[Describe the resulting context, after applying the decision. All consequences should be listed here, not just the "positive" ones.]

## Alternatives Considered

[What other options were considered? Why were they rejected?]

## Related

- [ADR-0001](0001-example.md) — Previous decision this builds on
- [ADR-0003](0003-example.md) — Related decision

## Notes

[Any additional context, links to discussions, etc.]
```

---

## Example ADRs

### ADR-0001: Use Microservices Architecture

**Status:** Accepted

**Context:**
- Monolithic application becoming difficult to maintain
- Team growing from 5 to 20 developers
- Need independent deployment of features
- Different scaling requirements for different components

**Decision:**
- Adopt microservices architecture
- Each service owns its own database
- Services communicate via REST APIs
- Use API Gateway pattern for client communication

**Consequences:**
- **Positive:** Independent deployment, technology diversity, better scaling
- **Negative:** Increased operational complexity, distributed transaction challenges
- **Neutral:** Need to implement service discovery, monitoring, logging

**Alternatives Considered:**
1. **Modular Monolith** — Easier to start but harder to scale team
2. **Serverless** — Too early in the serverless ecosystem
3. **SOA** — More enterprise-focused than needed

---

### ADR-0002: Use PostgreSQL for Primary Database

**Status:** Accepted

**Context:**
- Need relational database with strong ACID guarantees
- Require JSON support for flexible schemas
- Team familiar with SQL
- Budget constraints

**Decision:**
- Use PostgreSQL 14+ as primary database
- Use JSONB for flexible data storage
- Implement connection pooling
- Use migrations for schema changes

**Consequences:**
- **Positive:** Strong community, good performance, JSON support
- **Negative:** Operational overhead for backups and scaling
- **Neutral:** Need to monitor query performance

**Alternatives Considered:**
1. **MySQL** — Similar but less advanced JSON support
2. **MongoDB** — No ACID transactions at the time
3. **CockroachDB** — Too expensive for current budget

---

### ADR-0003: Use React for Frontend

**Status:** Accepted

**Context:**
- Need modern frontend framework
- Team has JavaScript experience
- Require good ecosystem and community
- Need component-based architecture

**Decision:**
- Use React 18+ with TypeScript
- Use functional components with hooks
- Use Context API for state management
- Use React Router for navigation

**Consequences:**
- **Positive:** Large ecosystem, good performance, strong typing
- **Negative:** Learning curve for hooks, bundle size concerns
- **Neutral:** Need to implement testing strategy

**Alternatives Considered:**
1. **Vue.js** — Simpler but smaller ecosystem
2. **Angular** — More opinionated, steeper learning curve
3. **Svelte** — Newer, less proven

---

## ADR Workflow

### 1. Propose
- Create ADR with "Proposed" status
- Describe context and alternatives
- Present to team for discussion

### 2. Review
- Team discusses pros and cons
- Consider alternatives
- Assess consequences
- May request changes

### 3. Decide
- Team reaches consensus
- Update status to "Accepted"
- Implement the decision

### 4. Maintain
- Update ADR if new information emerges
- Mark as "Deprecated" if no longer relevant
- Point at it from the specs and status files it governs

---

## Where decisions live

**Not in the repository.** A repo's `docs/` is user documentation; a folder of
ADRs next to the code drifts from it, gets published by accident, and is read
by nobody at decision time. Decisions, their context and the alternatives go
to the **knowledge layer** — with Canopy, `intelligence_upsert` as a decision
node linked to the project and to the facts it rests on (see the
`canopy-intelligence` skill). What stays in the repo is at most a pointer:

- a spec carries the decision as settled (the what and the how), never the
  debate;
- a status or roadmap entry names the knowledge node id;
- a code comment states the invariant the decision protects, not its history.

The template above is the *shape* of the knowledge node: title, status,
context, decision, consequences, alternatives. Superseding a decision is a new
node that links to the old one, which is marked superseded — never an edit
that erases what was decided before.

---

## ADR Best Practices

### Do's
- **Be concise** — Focus on key information
- **Be specific** — Avoid vague language
- **Document alternatives** — Show you considered options
- **Update status** — Keep ADRs current
- **Point at it** — specs, status files and code comments name the decision node; they do not copy it

### Don'ts
- **Don't over-document** — Not every decision needs an ADR
- **Don't be dogmatic** — ADRs can be changed
- **Don't duplicate** — Reference existing documentation
- **Don't neglect** — Review ADRs periodically

---

## When to Write an ADR

✅ **Write an ADR when:**
- Decision has long-term impact
- Multiple reasonable alternatives exist
- Decision affects multiple teams
- Significant architectural change
- Compliance or regulatory requirements

❌ **Don't write an ADR when:**
- Decision is trivial or temporary
- Only one obvious solution exists
- Decision affects only one component
- Implementation detail, not architectural

---

## ADR Maintenance

### Review Schedule
- **Quarterly** — Review all active ADRs
- **Annually** — Archive deprecated ADRs
- **On change** — Update related ADRs

### Versioning
- Use semantic versioning for ADR format
- Don't version individual ADRs (they're immutable)
- Create new ADR if decision changes significantly

---

## The Rule

> Document architectural decisions as you make them.
> Keep ADRs concise, current, and connected to code.
> Review periodically to ensure they still make sense.
> When in doubt, write it down — future you will thank you.
