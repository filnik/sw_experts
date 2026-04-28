# Agile and Lean

## Profile

### What It Is
Agile is a set of values and principles for iterative, incremental software development that emphasizes collaboration, adaptability, and delivering working software frequently. Lean applies manufacturing efficiency principles (Toyota Production System) to software — eliminating waste, optimizing flow, and delivering value faster.

### Origin and Evolution
- Lean manufacturing (Toyota, 1950s-80s) → Lean software development (Poppendieck, 2003)
- Agile Manifesto (2001) — 17 signatories, 4 values, 12 principles
- Scrum (Schwaber & Sutherland), XP (Beck), Kanban (Anderson) as frameworks
- SAFe, LeSS for scaling agile to large organizations
- Current: post-agile pragmatism, Shape Up (Basecamp), continuous delivery over sprints

### Key Authors and References
- **Agile Manifesto signatories** — Beck, Fowler, Cockburn, et al. (2001)
- **Ken Schwaber & Jeff Sutherland** — Scrum Guide
- **Mary & Tom Poppendieck** — *Lean Software Development*
- **David Anderson** — *Kanban: Successful Evolutionary Change*

### Key Concepts at a Glance
| Agile Values | Description |
|-------------|-------------|
| Individuals and interactions | Over processes and tools |
| Working software | Over comprehensive documentation |
| Customer collaboration | Over contract negotiation |
| Responding to change | Over following a plan |

| Lean Principles | Description |
|----------------|-------------|
| Eliminate waste | Remove anything that doesn't deliver value |
| Build quality in | Prevent defects, don't inspect for them |
| Deliver fast | Short cycle times, small batches |
| Optimize the whole | System-level thinking, not local optimization |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Deliver value early and often** — Working software delivered frequently is the primary measure of progress.
2. **Embrace change** — Welcome changing requirements, even late in development. Agile processes harness change for competitive advantage.
3. **Eliminate waste** — Anything that doesn't deliver value to the customer is waste: unnecessary features, handoffs, delays, defects.
4. **Feedback loops** — Short iterations with customer feedback drive the right product. Longer feedback loops increase risk.
5. **Sustainable pace** — Agile promotes sustainable development. Sprints should be marathons at a steady pace, not death marches.

### When to Use vs. When to Avoid

**Agile/Lean works well when:**
- Requirements are evolving or uncertain
- Customer feedback can be incorporated regularly
- Cross-functional teams can self-organize
- Rapid delivery is valued over upfront planning

**Traditional approaches may fit when:**
- Requirements are truly fixed and well-understood
- Regulatory environments require extensive upfront documentation
- Projects with hard contractual deliverables and dates

---

## Frameworks and Methodologies

### 1. Scrum — step-by-step
1. Product Owner maintains prioritized Product Backlog
2. Sprint Planning: team selects work for 1-2 week Sprint
3. Daily Standup: 15-min sync on progress and blockers
4. Team works on Sprint Backlog items
5. Sprint Review: demonstrate working software to stakeholders
6. Sprint Retrospective: reflect on process improvements
7. Repeat

### 2. Kanban — step-by-step
1. Visualize workflow on a board (To Do → In Progress → Done)
2. Set Work-in-Progress (WIP) limits per column
3. Measure flow metrics: cycle time, throughput, lead time
4. Identify and resolve bottlenecks (where work queues)
5. Continuously improve process based on metrics
6. Pull work when capacity is available (no sprints needed)

---

## How to Apply

### Decision Checklist
- [ ] Is value being delivered incrementally (not big-bang)?
- [ ] Are feedback loops short (days/weeks, not months)?
- [ ] Is WIP limited to maintain flow?
- [ ] Are retrospectives driving real improvements?
- [ ] Is the team empowered to self-organize?

### Implementation Patterns

**Kanban Board with WIP Limits:**
```
| Backlog | Todo (3) | In Progress (2) | Review (2) | Done |
|---------|----------|-----------------|------------|------|
| Story E | Story D  | Story B         | Story A    | ...  |
|         | Story C  |                 |            |      |

WIP limits prevent overload:
- If "In Progress" is at limit (2), no new work starts
- Team must finish or help with current work first
- This surfaces bottlenecks visibly
```

### Common Mistakes
1. **Cargo cult agile** — Following Scrum ceremonies without understanding the principles
2. **No retrospective action** — Having retrospectives but never implementing improvements
3. **Sprint as mini-waterfall** — Analysis sprint → dev sprint → test sprint is not agile
4. **No WIP limits** — Starting everything, finishing nothing
5. **Story points as velocity targets** — Points are for estimation, not performance measurement

### Integration with Other Topics
- **Team Topologies** — Agile teams aligned with team topology principles
- **Technical Debt** — Agile backlog includes technical debt items
- **CI/CD** — Continuous delivery is the technical enabler of agile
- **Architecture Decision Records** — Lightweight documentation compatible with agile
- **Evolutionary Architecture** — Architecture that evolves with agile delivery
- **Technical Leadership** — Servant leadership in agile teams

---

## Resources

### Essential Reading
- Agile Manifesto (agilemanifesto.org)
- *Lean Software Development* — Mary & Tom Poppendieck
- *Scrum Guide* — Schwaber & Sutherland (free, scrumguides.org)
- *Kanban* — David Anderson

### Online Resources
- Mountain Goat Software (Mike Cohn) — Agile resources
- Kanbanize blog on Kanban practices
- Shape Up (basecamp.com/shapeup) — alternative to Scrum

### Practice Exercises
1. Set up a Kanban board with WIP limits for a team
2. Run a sprint retrospective using the Start/Stop/Continue format
3. Measure cycle time and identify the biggest bottleneck
4. Convert a large project plan into an incremental delivery roadmap
5. Practice splitting user stories into independently deliverable slices
