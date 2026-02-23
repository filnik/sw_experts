# Design Patterns (GoF and Beyond)

## Profile

### What It Is
Design patterns are reusable solutions to commonly occurring problems in software design. They provide a shared vocabulary and proven templates for structuring code, enabling developers to communicate design decisions efficiently.

### Origin and Evolution
- Inspired by Christopher Alexander's architectural patterns (*A Pattern Language*, 1977)
- Formalized by the Gang of Four (GoF): Gamma, Helm, Johnson, Vlissides (*Design Patterns*, 1994)
- Extended by patterns communities: POSA, enterprise patterns (Fowler), domain patterns (Evans)
- Modern evolution: functional patterns, reactive patterns, cloud-native patterns

### Key Authors and References
- **Gang of Four** — *Design Patterns: Elements of Reusable Object-Oriented Software* (1994)
- **Martin Fowler** — *Patterns of Enterprise Application Architecture* (2002)
- **Eric Evans** — *Domain-Driven Design* (2003)
- **Frank Buschmann et al.** — *Pattern-Oriented Software Architecture* (POSA) series

### Key Concepts at a Glance
| Category | Patterns | Purpose |
|----------|----------|---------|
| Creational | Factory, Builder, Singleton, Prototype | Object creation mechanisms |
| Structural | Adapter, Decorator, Facade, Proxy, Composite | Object composition |
| Behavioral | Strategy, Observer, Command, State, Chain of Responsibility | Object interaction |
| Beyond GoF | Repository, Unit of Work, Specification, CQRS | Enterprise/domain patterns |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Program to an interface, not an implementation** — Depend on abstractions to allow flexible substitution of concrete behaviors
2. **Favor composition over inheritance** — Compose objects to get new functionality rather than inheriting from base classes
3. **Encapsulate what varies** — Identify aspects of your application that vary and separate them from what stays the same
4. **Strive for loosely coupled designs** — Minimize dependencies between interacting objects
5. **A pattern is a proven solution, not a goal** — Apply patterns to solve problems, don't force problems into patterns

### When to Use vs. When to Avoid

**Use when:**
- A recognized problem matches a known pattern
- The team benefits from shared vocabulary
- The pattern simplifies the design (not complicates it)
- Multiple implementations of a behavior are needed

**Avoid when:**
- A simpler solution exists (YAGNI)
- The pattern adds layers without clear benefit
- Only one implementation will ever exist
- The language provides built-in constructs (e.g., closures replace Strategy in many cases)

---

## Frameworks and Methodologies

### 1. Pattern Selection — step-by-step
1. Identify the design problem (creation, structure, or behavior)
2. List the forces: what varies, what's fixed, what constraints exist
3. Match forces to candidate patterns
4. Evaluate trade-offs (complexity vs. flexibility)
5. Implement the simplest pattern that resolves the forces
6. Document the decision (ADR if significant)

### 2. Pattern Refactoring — step-by-step
1. Identify code smells (long switch, parallel hierarchies, feature envy)
2. Map smell to candidate pattern (switch → Strategy, nested ifs → State)
3. Extract the varying behavior into its own abstraction
4. Implement pattern incrementally (one refactoring step at a time)
5. Verify behavior preservation with tests

---

## How to Apply

### Decision Checklist
- [ ] Is there a genuine design problem, or am I pattern-hunting?
- [ ] Does the pattern reduce complexity or increase it?
- [ ] Does the team understand this pattern?
- [ ] Are there simpler alternatives (function, closure, module)?
- [ ] Will this pattern need to evolve, and does it support that?

### Implementation Patterns

**Strategy Pattern:**
```python
from typing import Protocol

class PricingStrategy(Protocol):
    def calculate(self, base_price: float) -> float: ...

class RegularPricing:
    def calculate(self, base_price: float) -> float:
        return base_price

class DiscountPricing:
    def calculate(self, base_price: float) -> float:
        return base_price * 0.8

class Order:
    def __init__(self, pricing: PricingStrategy):
        self.pricing = pricing

    def total(self, base_price: float) -> float:
        return self.pricing.calculate(base_price)
```

**Builder Pattern:**
```typescript
class QueryBuilder {
  private table: string = '';
  private conditions: string[] = [];
  private ordering: string[] = [];

  from(table: string): this {
    this.table = table;
    return this;
  }

  where(condition: string): this {
    this.conditions.push(condition);
    return this;
  }

  orderBy(field: string): this {
    this.ordering.push(field);
    return this;
  }

  build(): string {
    let query = `SELECT * FROM ${this.table}`;
    if (this.conditions.length) query += ` WHERE ${this.conditions.join(' AND ')}`;
    if (this.ordering.length) query += ` ORDER BY ${this.ordering.join(', ')}`;
    return query;
  }
}
```

### Common Mistakes
1. **Pattern obsession** — Applying patterns everywhere regardless of need ("golden hammer")
2. **Wrong pattern** — Using Singleton when a simple module-level instance suffices
3. **Over-abstracting** — Creating pattern infrastructure for a single concrete case
4. **Ignoring language idioms** — Using verbose OOP patterns in languages with first-class functions
5. **Singleton abuse** — Using Singleton as a disguised global variable, making testing impossible

### Integration with Other Topics
- **SOLID Principles** — Patterns are implementations of SOLID; Strategy implements OCP, Observer implements DIP
- **Clean Architecture** — Patterns structure the inner layers (use cases, entities)
- **Domain-Driven Design** — Repository, Specification, Factory are DDD building blocks
- **Refactoring** — Patterns are common refactoring targets ("refactor to patterns")
- **Test-Driven Development** — Well-patterned code is inherently more testable

---

## Resources

### Essential Reading
- *Design Patterns: Elements of Reusable Object-Oriented Software* — GoF
- *Head First Design Patterns* — Freeman & Robson (accessible introduction)
- *Patterns of Enterprise Application Architecture* — Martin Fowler
- *Refactoring to Patterns* — Joshua Kerievsky

### Online Resources
- Refactoring Guru (refactoring.guru/design-patterns)
- Source Making (sourcemaking.com/design_patterns)
- Martin Fowler's Catalog of Patterns

### Practice Exercises
1. Implement a notification system using Observer pattern
2. Refactor a complex object creation into a Builder
3. Replace a type-code switch with Strategy pattern
4. Implement a chain of validators using Chain of Responsibility
5. Create a decorator stack for HTTP middleware
