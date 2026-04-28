# Team Topologies

## Profile

### What It Is
Team Topologies is a framework for organizing business and technology teams for fast flow of work. It defines four fundamental team types and three interaction modes that enable autonomous, loosely coupled teams aligned with the software architecture (Conway's Law).

### Origin and Evolution
- Conway's Law (1967): "Organizations design systems that mirror their communication structures"
- Inverse Conway Maneuver: design teams to produce the desired architecture
- Team Topologies formalized by Matthew Skelton and Manuel Pais (2019)
- Builds on Spotify model, DevOps principles, and organizational design research
- Current: widely adopted for structuring engineering organizations

### Key Authors and References
- **Matthew Skelton & Manuel Pais** — *Team Topologies* (2019)
- **Melvin Conway** — Conway's Law (1967)
- **Ruth Malan** — Organizational architecture
- **Nicole Forsgren** — *Accelerate* (team structures and performance)

### Key Concepts at a Glance
| Team Type | Purpose |
|-----------|---------|
| Stream-aligned | Delivers value in a specific business domain; the primary team type |
| Platform | Provides self-service internal capabilities to stream-aligned teams |
| Enabling | Helps stream-aligned teams overcome obstacles and adopt new practices |
| Complicated-subsystem | Manages technically complex components requiring specialist knowledge |

| Interaction Mode | Description |
|-----------------|-------------|
| Collaboration | Two teams working closely together (temporary) |
| X-as-a-Service | One team provides a service consumed by another |
| Facilitating | One team helps another improve a capability |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Optimize for flow** — Organize teams to minimize handoffs, waiting, and dependencies. Fast flow of change from idea to production.
2. **Conway's Law is real** — Your architecture will mirror your team structure. Design teams to produce the architecture you want.
3. **Cognitive load matters** — Teams have limited cognitive capacity. Don't overload teams with too many responsibilities or domains.
4. **Stream-aligned teams are primary** — Most teams should be stream-aligned (delivering business value). Other types exist to support them.
5. **Team interactions should evolve** — Collaboration → X-as-a-Service is a natural evolution. Start collaborating, then establish clear interfaces.

### When to Use vs. When to Avoid

**Use Team Topologies when:**
- Scaling engineering organization (20+ engineers)
- Teams feel overloaded or have too many dependencies
- Architecture doesn't match desired team autonomy
- Delivery speed is slowing due to coordination overhead

**Simpler structures for:**
- Small organizations (< 15 engineers)
- Single-product companies with one team
- Early-stage startups where everyone does everything

---

## Frameworks and Methodologies

### 1. Team Design — step-by-step
1. Map current teams and their responsibilities
2. Identify cognitive load per team (too many domains? too many services?)
3. Define business streams (customer journeys, product areas)
4. Create stream-aligned teams around business streams
5. Identify shared capabilities that could become platform teams
6. Define interaction modes between teams
7. Evolve: review and adjust quarterly

### 2. Platform Team Design — step-by-step
1. Identify repeated work across stream-aligned teams (CI/CD, infrastructure, auth)
2. Extract into a platform team with clear internal product (self-service)
3. Treat internal platform as a product: documentation, SLOs, support channels
4. Platform should reduce cognitive load on stream-aligned teams
5. Measure: are stream-aligned teams faster because of the platform?

---

## How to Apply

### Decision Checklist
- [ ] Is each team's cognitive load manageable (2-3 services, 1 domain)?
- [ ] Are stream-aligned teams able to deliver end-to-end without waiting?
- [ ] Does the platform team provide self-service capabilities?
- [ ] Are interaction modes explicitly defined between teams?
- [ ] Is the team structure aligned with the desired architecture?

### Implementation Patterns

**Team Mapping:**
```
Stream-Aligned Teams:
  - Orders Team: owns order service, payment integration, order notifications
  - Catalog Team: owns product catalog, search, recommendations
  - Customer Team: owns registration, profiles, preferences

Platform Team:
  - Developer Platform: CI/CD pipelines, Kubernetes platform, monitoring, secrets
  - Data Platform: data pipelines, analytics infrastructure, feature store

Enabling Team:
  - SRE Enablement: helps stream teams adopt SRE practices, then moves on

Complicated-Subsystem:
  - ML Team: owns recommendation engine and ML infrastructure
```

### Common Mistakes
1. **Too many team types** — Most teams should be stream-aligned. Platform and enabling are supporting roles.
2. **Platform as gatekeeper** — Platform should enable self-service, not be a bottleneck for approvals
3. **Ignoring cognitive load** — "The team can handle 15 services" — no, they can't
4. **Static team structure** — Teams and interactions should evolve as the organization and product change
5. **Conway's Law ignored** — Team structure that doesn't match desired architecture creates constant friction

### Integration with Other Topics
- **Microservices** — Service boundaries should align with team boundaries
- **Modular Monolith** — Modules map to team ownership
- **Domain-Driven Design** — Bounded contexts inform team boundaries
- **Agile and Lean** — Agile teams structured per Team Topologies principles
- **Platform Engineering** — Platform team is a key Team Topologies concept
- **Technical Leadership** — Leaders design team structures

---

## Resources

### Essential Reading
- *Team Topologies* — Matthew Skelton & Manuel Pais
- *Accelerate* — Forsgren, Humble, Kim
- Conway's Law original paper

### Online Resources
- teamtopologies.com — Official resources and patterns
- Team Topologies Academy (online courses)
- ThoughtWorks Technology Radar — Team Topologies recommendation

### Practice Exercises
1. Map your current team structure to Team Topologies types
2. Assess cognitive load for each team (domains, services, technologies)
3. Design a platform team charter with self-service offerings
4. Define explicit interaction modes between your teams
5. Apply the Inverse Conway Maneuver: restructure teams to match desired architecture
