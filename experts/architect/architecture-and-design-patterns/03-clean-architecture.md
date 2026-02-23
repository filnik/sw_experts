# Clean Architecture

## Profile

### What It Is
Clean Architecture is an architectural approach that organizes code into concentric layers with a strict dependency rule: dependencies always point inward, from low-level details toward high-level policies. The core business logic has zero dependencies on frameworks, databases, or UI.

### Origin and Evolution
- Built on earlier layered architectures: Hexagonal (Cockburn, 2005), Onion (Palermo, 2008)
- Formalized by Robert C. Martin in blog posts (2012) and book (2017)
- Synthesizes ideas from DDD, SOLID, and component design principles
- Widely adopted in enterprise systems, particularly in .NET and JVM ecosystems

### Key Authors and References
- **Robert C. Martin** — *Clean Architecture: A Craftsman's Guide to Software Structure and Design* (2017)
- **Alistair Cockburn** — Hexagonal Architecture (precursor)
- **Jeffrey Palermo** — Onion Architecture (precursor)
- **Mark Seemann** — *Dependency Injection in .NET* (practical application)

### Key Concepts at a Glance
| Layer | Contains | Depends On |
|-------|----------|------------|
| Entities | Business rules, domain objects | Nothing |
| Use Cases | Application-specific business rules | Entities |
| Interface Adapters | Controllers, presenters, gateways | Use Cases, Entities |
| Frameworks & Drivers | DB, web, UI, external services | Everything (outermost) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **The Dependency Rule** — Source code dependencies must point inward only. Inner layers know nothing about outer layers.
2. **Independence of frameworks** — The architecture does not depend on any framework; frameworks are tools, not the architecture.
3. **Testability without externals** — Business rules can be tested without UI, database, web server, or any external element.
4. **Independence of UI** — The UI can change without changing the business rules.
5. **Independence of database** — Business rules are not bound to a specific database; you can swap databases without affecting core logic.

### When to Use vs. When to Avoid

**Use when:**
- Building long-lived systems that will evolve over years
- Business logic is complex and central to the application's value
- Multiple delivery mechanisms (web, CLI, API) share the same logic
- Strong testing requirements exist
- Team size is medium-to-large

**Avoid when:**
- Building CRUD-heavy applications with minimal business logic
- Prototyping or building MVPs where speed is critical
- Small microservices with simple pass-through logic
- The overhead of layers outweighs the benefits

---

## Frameworks and Methodologies

### 1. Layer Decomposition — step-by-step
1. Identify core business entities and their invariants
2. Define use cases as application-specific operations on entities
3. Design ports (interfaces) that use cases need from the outside world
4. Implement adapters that fulfill port contracts
5. Configure the composition root to wire everything together
6. Verify: no import from inner layer references outer layer

### 2. Screaming Architecture — step-by-step
1. Organize top-level directories by business capability, not technical concern
2. Name modules after domain concepts (Orders, Patients, Inventory)
3. Place use cases prominently — they should be the first thing a reader sees
4. Relegate framework code to adapter/infrastructure directories
5. A new developer should understand the domain from the folder structure alone

---

## How to Apply

### Decision Checklist
- [ ] Does the folder structure communicate the domain, not the framework?
- [ ] Can I test use cases without any infrastructure?
- [ ] Do all dependencies point inward?
- [ ] Is the database an implementation detail, not a design driver?
- [ ] Can I replace the web framework without touching business logic?

### Implementation Patterns

**Layer Structure:**
```
src/
  domain/           # Entities, value objects, domain services
    entities/
    value_objects/
    services/
  application/      # Use cases, ports (interfaces)
    use_cases/
    ports/
  infrastructure/   # Adapters, repositories, external services
    persistence/
    messaging/
    http_client/
  presentation/     # Controllers, views, DTOs
    api/
    cli/
```

**Use Case Example:**
```python
class CreateOrder:
    """Application use case — depends only on ports and entities."""

    def __init__(self, order_repo: OrderRepository, event_bus: EventBus):
        self.order_repo = order_repo
        self.event_bus = event_bus

    def execute(self, command: CreateOrderCommand) -> OrderId:
        order = Order.create(
            customer_id=command.customer_id,
            items=command.items,
        )
        self.order_repo.save(order)
        self.event_bus.publish(OrderCreated(order_id=order.id))
        return order.id
```

**Port and Adapter:**
```python
# Port (in application layer)
class OrderRepository(Protocol):
    def save(self, order: Order) -> None: ...
    def find_by_id(self, order_id: OrderId) -> Order | None: ...

# Adapter (in infrastructure layer)
class PostgresOrderRepository:
    def __init__(self, session: Session):
        self.session = session

    def save(self, order: Order) -> None:
        self.session.add(OrderModel.from_domain(order))
        self.session.commit()

    def find_by_id(self, order_id: OrderId) -> Order | None:
        model = self.session.get(OrderModel, str(order_id))
        return model.to_domain() if model else None
```

### Common Mistakes
1. **Leaking framework into use cases** — Importing Django/Flask/Express types in use case layer
2. **Anemic use cases** — Use cases that just delegate to repositories without business logic
3. **Over-layering simple CRUD** — Adding 4 layers for a simple save/retrieve operation
4. **DTOs everywhere** — Mapping between identical data structures at every boundary
5. **Ignoring the dependency rule** — Having entities reference infrastructure concerns

### Integration with Other Topics
- **SOLID Principles** — Clean Architecture is SOLID applied at the architectural level
- **Hexagonal Architecture** — Clean Architecture is essentially hexagonal with named layers
- **Domain-Driven Design** — The entity layer maps to DDD's domain model
- **CQRS** — Can be applied within the use case layer for read/write separation
- **Microservices** — Each microservice can internally use Clean Architecture

---

## Resources

### Essential Reading
- *Clean Architecture* — Robert C. Martin
- *Get Your Hands Dirty on Clean Architecture* — Tom Hombergs (practical, Spring-focused)
- *Implementing Domain-Driven Design* — Vaughn Vernon (complementary)

### Online Resources
- Uncle Bob's "Clean Architecture" blog post (original 2012)
- Jason Taylor's Clean Architecture template (.NET)
- Various community implementations on GitHub

### Practice Exercises
1. Refactor a framework-coupled app to Clean Architecture layers
2. Implement a use case that works with both REST and CLI delivery
3. Write tests for use cases using in-memory repository adapters
4. Swap a database adapter without changing any use case code
5. Organize a new project with "screaming architecture" folder structure
