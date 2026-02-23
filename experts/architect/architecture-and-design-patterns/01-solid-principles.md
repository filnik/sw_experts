# SOLID Principles

## Profile

### What It Is
SOLID is an acronym for five object-oriented design principles that promote maintainable, flexible, and understandable software. They guide developers in creating systems that are easy to extend, refactor, and test.

### Origin and Evolution
- Coined by Robert C. Martin (Uncle Bob) in the early 2000s, though individual principles emerged earlier
- The acronym was introduced by Michael Feathers
- Originally OOP-focused, now widely applied to functional and modular architectures
- Evolved from academic principles to industry standard practices

### Key Authors and References
- **Robert C. Martin** — *Agile Software Development: Principles, Patterns, and Practices* (2002)
- **Bertrand Meyer** — Open/Closed Principle origin (*Object-Oriented Software Construction*, 1988)
- **Barbara Liskov** — Liskov Substitution Principle (*Data Abstraction and Hierarchy*, 1987)
- **Robert C. Martin** — *Clean Architecture* (2017)

### Key Concepts at a Glance
| Principle | Focus | One-liner |
|-----------|-------|-----------|
| SRP | Cohesion | One reason to change |
| OCP | Extension | Open for extension, closed for modification |
| LSP | Substitutability | Subtypes must be substitutable |
| ISP | Granularity | No client forced to depend on unused methods |
| DIP | Decoupling | Depend on abstractions, not concretions |

---

## Core Philosophy

### The 5 Fundamental Principles

1. **Single Responsibility Principle (SRP)** — A class/module should have one, and only one, reason to change. Group things that change for the same reason; separate things that change for different reasons.

2. **Open/Closed Principle (OCP)** — Software entities should be open for extension but closed for modification. Achieve through abstractions, polymorphism, and composition rather than editing existing code.

3. **Liskov Substitution Principle (LSP)** — Objects of a supertype should be replaceable with objects of a subtype without altering the correctness of the program. Subtypes must honor the contract of their parent types.

4. **Interface Segregation Principle (ISP)** — No client should be forced to depend on interfaces it does not use. Prefer many small, focused interfaces over one large general-purpose interface.

5. **Dependency Inversion Principle (DIP)** — High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.

### When to Use vs. When to Avoid

**Use when:**
- Building systems expected to evolve over time
- Working in teams where multiple developers touch the same codebase
- Designing libraries or frameworks consumed by others
- Codebases that require extensive testing

**Avoid over-application when:**
- Prototyping or building throwaway code
- Very small scripts or utilities (< 100 lines)
- Applying principles creates more complexity than the problem warrants
- Performance-critical inner loops where indirection has measurable cost

---

## Frameworks and Methodologies

### 1. SRP Decomposition — step-by-step
1. Identify the actors (stakeholders) that might request changes
2. Group responsibilities by actor
3. Extract each responsibility into its own class/module
4. Connect them through a facade or coordinator if needed
5. Verify: each module has exactly one reason to change

### 2. Dependency Inversion via Ports & Adapters — step-by-step
1. Identify the high-level policy (business logic)
2. Define abstractions (interfaces/protocols) the policy needs
3. Implement concrete adapters for each abstraction
4. Wire dependencies at the composition root (main/startup)
5. Verify: high-level modules import only abstractions, never concretions

---

## How to Apply

### Decision Checklist
- [ ] Does each class/module have a single reason to change?
- [ ] Can I extend behavior without modifying existing code?
- [ ] Can subtypes replace their parent types transparently?
- [ ] Are interfaces small and client-specific?
- [ ] Do high-level modules depend on abstractions?

### Implementation Patterns

**SRP Example:**
```python
# Bad: UserService handles auth, profile, and notifications
class UserService:
    def authenticate(self, credentials): ...
    def update_profile(self, data): ...
    def send_notification(self, message): ...

# Good: separated by responsibility
class AuthService:
    def authenticate(self, credentials): ...

class ProfileService:
    def update_profile(self, data): ...

class NotificationService:
    def send_notification(self, message): ...
```

**DIP Example:**
```python
# Bad: high-level depends on low-level
class OrderService:
    def __init__(self):
        self.repo = MySQLOrderRepository()  # concrete dependency

# Good: depend on abstraction
class OrderService:
    def __init__(self, repo: OrderRepository):  # abstraction
        self.repo = repo
```

### Common Mistakes
1. **Over-splitting with SRP** — Creating classes with a single method each; SRP means one *reason to change*, not one method
2. **Violating LSP with exceptions** — Subclass throws exceptions parent doesn't; breaks substitutability
3. **God interfaces** — One interface with 20 methods that no single implementor needs entirely
4. **Mechanical DIP** — Creating interfaces for every class even when there's only one implementation and no testing need
5. **Ignoring OCP pragmatism** — Not every class needs to be extensible; apply where change is likely

### Integration with Other Topics
- **Clean Architecture** — SOLID principles are the foundation of Clean Architecture's dependency rules
- **Hexagonal Architecture** — Ports embody ISP and DIP; adapters embody OCP
- **Domain-Driven Design** — Bounded contexts naturally align with SRP at the module level
- **Test-Driven Development** — SOLID code is inherently more testable
- **Refactoring** — SOLID violations are common refactoring targets

---

## Resources

### Essential Reading
- *Agile Software Development: Principles, Patterns, and Practices* — Robert C. Martin
- *Clean Architecture* — Robert C. Martin
- *Design Principles and Design Patterns* — Robert C. Martin (paper)

### Online Resources
- Martin Fowler's blog on SOLID and design principles
- Uncle Bob's Clean Coder blog
- SOLID principles explained with real-world examples (various community posts)

### Practice Exercises
1. Take a 200-line class and decompose it following SRP
2. Refactor a switch statement to use OCP (Strategy pattern)
3. Find an LSP violation in an inheritance hierarchy and fix it
4. Split a fat interface into role-specific interfaces
5. Invert a concrete dependency using constructor injection
