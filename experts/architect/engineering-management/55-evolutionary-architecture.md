# Evolutionary Architecture

## Profile

### What It Is
Evolutionary Architecture supports guided, incremental change across multiple dimensions. Instead of designing the "perfect" architecture upfront, it embraces the reality that requirements, technologies, and understanding evolve over time, using fitness functions to ensure architectural characteristics are preserved as the system changes.

### Origin and Evolution
- Reaction to big-design-up-front (BDUF) and its failures in dynamic environments
- Formalized by Neal Ford, Rebecca Parsons, and Patrick Kua (*Building Evolutionary Architectures*, 2017)
- Builds on agile principles applied to architecture
- Fitness functions as automated architectural governance
- Current: architecture fitness functions in CI/CD, living architecture documentation

### Key Authors and References
- **Neal Ford, Rebecca Parsons, Patrick Kua** — *Building Evolutionary Architectures* (2017, 2nd ed. 2023)
- **Martin Fowler** — "Sacrificial Architecture" and evolutionary design articles
- **ThoughtWorks** — Evolutionary architecture practices

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Fitness Function | Automated test that verifies an architectural characteristic |
| Guided Change | Architecture evolves intentionally, not accidentally |
| Last Responsible Moment | Defer decisions until you have enough information |
| Sacrificial Architecture | Build to learn, then replace when understanding improves |
| Incremental Change | Small, reversible architectural changes over time |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Architecture is not a phase — it's continuous** — Architecture decisions happen throughout the project, not just at the beginning.
2. **Fitness functions protect qualities** — Automated checks ensure performance, security, modularity, and other architectural properties are maintained as the code evolves.
3. **Defer decisions to the last responsible moment** — Don't choose a database, messaging system, or deployment strategy before you need to. Gather information first.
4. **Make change safe and reversible** — Small, incremental changes with fast feedback are safer than big architectural rewrites.
5. **Evolvability is an architectural characteristic** — The ability to change IS a feature. Design for it explicitly.

### When to Use vs. When to Avoid

**Evolutionary approach when:**
- Requirements are uncertain or changing
- Technology landscape is evolving
- Team is learning the domain
- Long-lived systems that must adapt over years

**More upfront design when:**
- Hard constraints (hardware, embedded, safety-critical)
- Regulatory requirements define architecture
- Integration with fixed external systems

---

## Frameworks and Methodologies

### 1. Fitness Function Design — step-by-step
1. Identify key architectural characteristics (performance, security, modularity, scalability)
2. For each characteristic, define a measurable metric
3. Write automated tests that verify the metric
4. Run fitness functions in CI/CD (fail the build if violated)
5. Review and update fitness functions as architecture evolves
6. Examples: ArchUnit tests, performance benchmarks, dependency checks

### 2. Incremental Architecture Migration — step-by-step
1. Document current architecture and pain points
2. Define target architecture direction (not final state)
3. Identify the smallest meaningful step toward the target
4. Implement the step with fitness functions protecting existing qualities
5. Evaluate: did this improve things? Adjust direction if needed.
6. Repeat

---

## How to Apply

### Decision Checklist
- [ ] Are architectural characteristics protected by fitness functions?
- [ ] Are decisions deferred to the last responsible moment?
- [ ] Is the architecture evolving incrementally (not big-bang rewrites)?
- [ ] Are fitness functions running in CI/CD?
- [ ] Is the team reviewing architecture decisions regularly?

### Implementation Patterns

**Architecture Fitness Functions:**
```python
# Modularity fitness function — no circular dependencies
def test_no_circular_module_dependencies():
    deps = analyze_module_dependencies("src/modules/")
    cycles = find_cycles(deps)
    assert cycles == [], f"Circular dependencies found: {cycles}"

# Performance fitness function — API latency
def test_api_latency_under_slo():
    results = run_load_test(endpoint="/api/orders", rps=100, duration=60)
    assert results.p99_latency_ms < 200, f"P99 latency {results.p99_latency_ms}ms exceeds 200ms SLO"

# Dependency fitness function — no banned imports
def test_domain_layer_has_no_infrastructure_imports():
    violations = check_imports(
        from_pattern="src/domain/**/*.py",
        banned_patterns=["sqlalchemy", "redis", "requests", "fastapi"],
    )
    assert violations == [], f"Domain layer imports infrastructure: {violations}"
```

### Common Mistakes
1. **BDUF disguised as evolutionary** — Planning the entire target architecture upfront defeats the purpose
2. **No fitness functions** — "Evolutionary" without automated governance is just entropy
3. **Big-bang migrations** — Evolutionary means incremental; large rewrites are the opposite
4. **Ignoring the "last responsible moment"** — Either deciding too early (waste) or too late (crisis)
5. **Fitness functions that don't fail** — If they never fail, they're not protecting anything

### Integration with Other Topics
- **ADRs** — Document architectural evolution decisions
- **Modular Monolith** — Enables evolutionary extraction to microservices
- **Technical Debt** — Evolutionary architecture manages debt through incremental improvement
- **CI/CD** — Fitness functions run in the pipeline
- **Agile** — Evolutionary architecture is agile applied to architecture
- **Clean Architecture** — Provides structure that enables evolution

---

## Resources

### Essential Reading
- *Building Evolutionary Architectures* (2nd ed.) — Ford, Parsons, Kua
- *Fundamentals of Software Architecture* — Richards & Ford
- Martin Fowler — "Sacrificial Architecture" article

### Online Resources
- ThoughtWorks Technology Radar
- ArchUnit documentation (archunit.org)
- Neal Ford's conference talks on evolutionary architecture

### Practice Exercises
1. Write 3 fitness functions for an existing project (modularity, performance, dependency)
2. Identify an architectural decision that should be deferred
3. Plan an incremental migration from monolith to modular structure
4. Add ArchUnit-style tests to your CI/CD pipeline
5. Document an architectural evolution path with milestones
