# Architecture Decision Records

## Profile

### What It Is
Architecture Decision Records (ADRs) are lightweight documents that capture important architectural decisions along with their context, rationale, and consequences. They create an immutable decision log that helps current and future team members understand why the system is built the way it is.

### Origin and Evolution
- Michael Nygard proposed the ADR format (2011)
- Influenced by design rationale research and decision documentation practices
- ADR tools: adr-tools (CLI), Log4brains, various templates
- Modern: ADRs stored in the repository alongside code, reviewed via PRs
- Current: widely adopted as lightweight architecture documentation

### Key Authors and References
- **Michael Nygard** — ADR format and "Documenting Architecture Decisions" article
- **Joel Parker Henderson** — ADR template collection
- **ThoughtWorks** — ADR as recommended practice in Technology Radar

### Key Concepts at a Glance
| Section | Purpose |
|---------|---------|
| Title | Short descriptive title with number |
| Status | Proposed, Accepted, Deprecated, Superseded |
| Context | What is the issue or situation motivating this decision? |
| Decision | What is the change being proposed? |
| Consequences | What are the results (positive, negative, neutral)? |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Document the why, not just the what** — Code shows what was built. ADRs explain why it was built that way and what alternatives were considered.
2. **Lightweight and low-ceremony** — An ADR is a short text file (1-2 pages). If it's longer, it's probably too detailed.
3. **Immutable log** — ADRs are never modified after acceptance. New decisions supersede old ones with references.
4. **Stored with the code** — ADRs live in the repository (e.g., `docs/adr/`), versioned alongside the code they describe.
5. **Write them when the decision is made** — Don't try to document decisions retroactively months later. Capture context while it's fresh.

### When to Use vs. When to Avoid

**Write an ADR when:**
- Choosing between significant alternatives (database, framework, architecture pattern)
- The decision is hard to reverse or expensive to change
- Future developers will ask "why did they do it this way?"
- The team disagrees and needs to document the resolution

**Don't write an ADR for:**
- Trivial decisions (variable naming, formatting choices)
- Decisions already covered by team standards/conventions
- Temporary prototyping decisions

---

## Frameworks and Methodologies

### 1. ADR Creation — step-by-step
1. Identify a decision that needs documentation
2. Number it sequentially (ADR-001, ADR-002, ...)
3. Write Context: what problem/situation led to this?
4. Write Decision: what did we decide and why?
5. Write Consequences: what are the positive and negative results?
6. Submit as PR for review
7. Set status to Accepted when approved

### 2. ADR Lifecycle — step-by-step
1. **Proposed**: ADR written, under review
2. **Accepted**: Team agrees, decision is active
3. **Deprecated**: Decision is outdated but was never explicitly replaced
4. **Superseded by ADR-XXX**: A new ADR replaces this one (link to new one)
5. Keep old ADRs — they're historical context, not living documents

---

## How to Apply

### Decision Checklist
- [ ] Does this decision have significant architectural impact?
- [ ] Were alternatives considered and documented?
- [ ] Is the rationale clear to someone reading this in 2 years?
- [ ] Are consequences (positive and negative) listed?
- [ ] Is the ADR stored in the repository?

### Implementation Patterns

**ADR Template:**
```markdown
# ADR-007: Use PostgreSQL for Order Service

## Status
Accepted (2024-01-15)

## Context
The order service needs a persistent data store. Requirements:
- ACID transactions for order state changes
- Complex queries for reporting
- Team expertise in SQL
- Expected data volume: ~10M orders/year

Alternatives considered:
1. PostgreSQL — mature, ACID, strong SQL, team experience
2. MongoDB — flexible schema, but we have relational data
3. DynamoDB — scalable, but complex transactions, vendor lock-in

## Decision
We will use PostgreSQL (via RDS) for the order service.

## Consequences
**Positive:**
- Team has deep PostgreSQL expertise
- ACID transactions simplify order state management
- Rich SQL ecosystem for reporting and analytics
- Well-understood scaling path (read replicas, partitioning)

**Negative:**
- Vertical scaling limits compared to DynamoDB
- Need to manage schema migrations (Alembic)
- Sharding is complex if we exceed single-node capacity

**Neutral:**
- Standard operational overhead (backups, monitoring, upgrades)
```

**Repository Structure:**
```
docs/
  adr/
    ADR-001-use-microservices.md
    ADR-002-use-kafka-for-events.md
    ADR-003-choose-python-for-backend.md
    ...
    ADR-007-postgresql-for-orders.md
    README.md  # Index of all ADRs
```

### Common Mistakes
1. **Not writing them** — The most common mistake. Decisions are made verbally and forgotten.
2. **Too long** — ADRs should be 1-2 pages. If longer, you're writing a design document.
3. **No alternatives** — Just documenting what was chosen without explaining what was considered
4. **Retroactive documentation** — Writing ADRs months later loses context and rationale
5. **Editing accepted ADRs** — ADRs are immutable; create a new ADR that supersedes

### Integration with Other Topics
- **Evolutionary Architecture** — ADRs document architectural evolution over time
- **Technical Debt** — Deliberate debt should be documented in ADRs
- **Clean Architecture** — Architectural layer decisions documented in ADRs
- **Microservices** — Service boundary decisions are prime ADR candidates
- **Code Review** — ADRs reviewed via PR like code
- **Technical Leadership** — Leaders facilitate ADR creation and review

---

## Resources

### Essential Reading
- Michael Nygard — "Documenting Architecture Decisions" (cognitect.com)
- Joel Parker Henderson — ADR GitHub repository (templates and examples)
- *Design It!* — Michael Keeling (architecture documentation chapter)

### Online Resources
- adr.github.io — ADR resources and tools
- Log4brains — ADR management tool
- ThoughtWorks Technology Radar — ADR recommendation

### Practice Exercises
1. Write an ADR for a recent architectural decision
2. Create an ADR template and directory structure for your project
3. Review and discuss an ADR in a team meeting
4. Write an ADR that supersedes a previous decision
5. Create an ADR index (README) for an existing project
