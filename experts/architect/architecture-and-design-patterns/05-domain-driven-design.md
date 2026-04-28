# Domain-Driven Design

## Profile

### What It Is
Domain-Driven Design (DDD) is an approach to software development that centers the design and implementation on the core business domain. It provides strategic and tactical patterns for modeling complex business logic, establishing a shared language between developers and domain experts, and structuring systems around business boundaries.

### Origin and Evolution
- Introduced by Eric Evans in *Domain-Driven Design: Tackling Complexity in the Heart of Software* (2003)
- Built on object-oriented analysis and modeling traditions
- Revitalized by Vaughn Vernon's *Implementing Domain-Driven Design* (2013)
- Heavily adopted with microservices (bounded contexts map naturally to services)
- Modern evolution includes event-storming, collaborative modeling, and functional DDD

### Key Authors and References
- **Eric Evans** — *Domain-Driven Design* (2003, "The Blue Book")
- **Vaughn Vernon** — *Implementing Domain-Driven Design* (2013, "The Red Book")
- **Alberto Brandolini** — Event Storming methodology
- **Scott Wlaschin** — *Domain Modeling Made Functional* (2018)

### Key Concepts at a Glance
| Concept | Level | Description |
|---------|-------|-------------|
| Ubiquitous Language | Strategic | Shared language between devs and domain experts |
| Bounded Context | Strategic | Explicit boundary within which a model applies |
| Context Map | Strategic | Relationships between bounded contexts |
| Aggregate | Tactical | Cluster of entities with a consistency boundary |
| Entity | Tactical | Object defined by identity, not attributes |
| Value Object | Tactical | Immutable object defined by attributes |
| Domain Event | Tactical | Record of something significant that happened |
| Repository | Tactical | Abstraction for aggregate persistence |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Focus on the core domain** — The most valuable part of the software is the domain model. Invest the best talent and effort there, not in infrastructure.
2. **Model-driven design** — The code IS the model. There is no separate "design document" — the domain model lives in the code and evolves with understanding.
3. **Ubiquitous language** — Developers and domain experts share one language. If the code says `Order`, the business says `Order`. No translation layers.
4. **Bounded contexts for autonomy** — Each context has its own model, its own language, and its own boundaries. The same word can mean different things in different contexts.
5. **Strategic design before tactical patterns** — Getting context boundaries right matters more than perfecting entity/value object distinctions.

### When to Use vs. When to Avoid

**Use when:**
- The domain is complex and central to the business value
- Domain experts are available and willing to collaborate
- The system will be maintained and evolved over years
- Multiple teams work on different parts of the domain
- Business rules are intricate and change frequently

**Avoid when:**
- The domain is simple CRUD with no complex rules
- There are no accessible domain experts
- The project is a short-lived prototype
- The team lacks experience and mentorship in DDD
- The system is purely technical (infrastructure tooling)

---

## Frameworks and Methodologies

### 1. Event Storming — step-by-step
1. Gather domain experts and developers in a room with a wide wall
2. Write domain events (past tense, orange stickies) on the timeline
3. Identify commands (blue) that trigger events
4. Identify aggregates (yellow) that handle commands
5. Mark bounded context boundaries where language/models diverge
6. Identify policies (lilac) — reactions to events
7. Refine into a context map showing relationships

### 2. Aggregate Design — step-by-step
1. Identify the invariant that must be protected (consistency boundary)
2. Choose the aggregate root (the entity that enforces the invariant)
3. Include only entities/value objects needed to enforce the invariant
4. Keep aggregates small — prefer references by ID over object references
5. Design inter-aggregate communication through domain events
6. One transaction = one aggregate modification

---

## How to Apply

### Decision Checklist
- [ ] Have I identified the bounded contexts and their boundaries?
- [ ] Is the ubiquitous language documented and used in code?
- [ ] Are aggregates designed around invariants, not data structure?
- [ ] Are bounded context relationships mapped (ACL, shared kernel, etc.)?
- [ ] Is the core domain getting the most investment?

### Implementation Patterns

**Aggregate with Domain Events:**
```python
class Order:
    def __init__(self, order_id: OrderId, customer_id: CustomerId):
        self.id = order_id
        self.customer_id = customer_id
        self.items: list[OrderItem] = []
        self.status = OrderStatus.DRAFT
        self._events: list[DomainEvent] = []

    def add_item(self, product_id: ProductId, quantity: int, price: Money) -> None:
        if self.status != OrderStatus.DRAFT:
            raise OrderNotModifiable(self.id)
        item = OrderItem(product_id, quantity, price)
        self.items.append(item)

    def place(self) -> None:
        if not self.items:
            raise EmptyOrderCannotBePlaced(self.id)
        self.status = OrderStatus.PLACED
        self._events.append(OrderPlaced(order_id=self.id, total=self.total))

    @property
    def total(self) -> Money:
        return sum((item.subtotal for item in self.items), Money.zero())

    def collect_events(self) -> list[DomainEvent]:
        events = self._events.copy()
        self._events.clear()
        return events
```

**Value Object:**
```python
@dataclass(frozen=True)
class Money:
    amount: Decimal
    currency: str

    def __add__(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise CurrencyMismatch(self.currency, other.currency)
        return Money(self.amount + other.amount, self.currency)

    @classmethod
    def zero(cls, currency: str = "EUR") -> "Money":
        return cls(Decimal("0"), currency)
```

**Context Map Relationships:**
```
[Order Context] --Conformist--> [Shipping Context]
[Order Context] --ACL--> [Payment Context]
[Catalog Context] --Published Language--> [Order Context]
[Order Context] --Shared Kernel--> [Inventory Context]
```

### Common Mistakes
1. **DDD everywhere** — Applying tactical DDD patterns to simple CRUD; reserve DDD for complex domains
2. **Anemic domain model** — Entities that are just data bags with logic in services (procedural disguised as OO)
3. **Large aggregates** — Aggregates that include too many entities, causing contention and performance issues
4. **Ignoring bounded context boundaries** — Sharing one model across the entire system, leading to a big ball of mud
5. **Ubiquitous language only in code** — The language must be shared with domain experts, not just among developers

### Integration with Other Topics
- **Hexagonal Architecture** — Perfect fit; the domain model sits at the center of the hexagon
- **Clean Architecture** — DDD's domain layer maps to Clean Architecture's entities layer
- **Event Sourcing** — Storing domain events as the source of truth
- **CQRS** — Separating read and write models, often used with DDD
- **Microservices** — Bounded contexts inform service boundaries
- **Event-Driven Architecture** — Domain events drive inter-context communication

---

## Resources

### Essential Reading
- *Domain-Driven Design* — Eric Evans ("The Blue Book")
- *Implementing Domain-Driven Design* — Vaughn Vernon ("The Red Book")
- *Domain Modeling Made Functional* — Scott Wlaschin
- *Learning Domain-Driven Design* — Vlad Khononov

### Online Resources
- DDD Community website (dddcommunity.org)
- EventStorming.com — Alberto Brandolini
- Martin Fowler's DDD articles

### Practice Exercises
1. Run an Event Storming session for a familiar domain
2. Model an aggregate with invariant protection and domain events
3. Identify bounded contexts in an existing monolithic application
4. Implement a value object with proper equality semantics
5. Map context relationships for a multi-team system
