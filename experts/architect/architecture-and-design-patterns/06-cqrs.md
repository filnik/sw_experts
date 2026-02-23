# CQRS (Command Query Responsibility Segregation)

## Profile

### What It Is
CQRS is an architectural pattern that separates the read (query) model from the write (command) model of an application. Instead of using a single model for both reading and writing data, CQRS uses distinct models optimized for their specific purpose.

### Origin and Evolution
- Rooted in Bertrand Meyer's Command-Query Separation (CQS) principle (1988)
- Elevated to an architectural pattern by Greg Young (~2010)
- Often paired with Event Sourcing but is independent of it
- Adopted widely in DDD communities and event-driven systems
- Simplified variants (without separate databases) are common in modern applications

### Key Authors and References
- **Greg Young** — Primary advocate, formalized CQRS pattern
- **Bertrand Meyer** — CQS principle origin
- **Udi Dahan** — CQRS in distributed systems context
- **Martin Fowler** — "CQRS" article on bliki

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Command | An intent to change state; validated and processed by write model |
| Query | A request for data; served by optimized read model |
| Write Model | Domain model optimized for enforcing business rules |
| Read Model | Denormalized projection optimized for queries |
| Eventual Consistency | Read model may lag behind write model |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Separate concerns** — Reads and writes have fundamentally different requirements; optimizing for both in a single model creates compromises
2. **Optimize each side independently** — The write model enforces invariants; the read model serves data. Each can use different data stores, schemas, and scaling strategies
3. **Commands are intentions** — Commands express what the user wants to do, not how the data should change. They are validated and may be rejected
4. **Queries are side-effect free** — Queries never modify state. They return data without triggering business logic
5. **Accept eventual consistency** — When using separate data stores, the read model may be slightly behind the write model. Design the UX accordingly

### When to Use vs. When to Avoid

**Use when:**
- Read and write workloads have very different scaling needs
- Read models require complex denormalization or aggregation
- The domain model is complex and optimizing it for reads compromises its integrity
- Multiple read representations of the same data are needed
- You're already using Event Sourcing

**Avoid when:**
- The domain is simple CRUD
- Read and write models would be nearly identical
- The team cannot handle the added complexity
- Strong consistency is required everywhere (no tolerance for eventual consistency)
- The system is small and unlikely to grow significantly

---

## Frameworks and Methodologies

### 1. Simple CQRS (Same Database) — step-by-step
1. Separate command handlers from query handlers at the code level
2. Commands go through domain model (aggregates, validation)
3. Queries bypass the domain model and read directly from optimized views/queries
4. Use database views or optimized queries for the read side
5. Maintain consistency since both sides share the same database

### 2. Full CQRS (Separate Stores) — step-by-step
1. Design the write model around aggregates and domain events
2. Publish domain events after each state change
3. Build projections (event handlers) that update the read store
4. Design read store schema for query optimization (denormalized)
5. Implement synchronization mechanism (event bus, change data capture)
6. Handle eventual consistency in the UI (optimistic updates, polling)

---

## How to Apply

### Decision Checklist
- [ ] Are read and write patterns significantly different?
- [ ] Would a single model require compromises on either side?
- [ ] Can the business tolerate eventual consistency for reads?
- [ ] Is the added complexity justified by the benefits?
- [ ] Do we have the infrastructure for event-based projection?

### Implementation Patterns

**Command Side:**
```python
@dataclass(frozen=True)
class PlaceOrderCommand:
    customer_id: str
    items: list[OrderItemDTO]

class PlaceOrderHandler:
    def __init__(self, order_repo: OrderRepository, event_bus: EventBus):
        self.order_repo = order_repo
        self.event_bus = event_bus

    def handle(self, command: PlaceOrderCommand) -> OrderId:
        order = Order.create(command.customer_id, command.items)
        self.order_repo.save(order)
        self.event_bus.publish(order.collect_events())
        return order.id
```

**Query Side:**
```python
@dataclass(frozen=True)
class OrderSummaryQuery:
    customer_id: str
    status: str | None = None

@dataclass(frozen=True)
class OrderSummaryView:
    order_id: str
    customer_name: str
    total: Decimal
    item_count: int
    status: str

class OrderSummaryQueryHandler:
    def __init__(self, read_db: ReadDatabase):
        self.read_db = read_db

    def handle(self, query: OrderSummaryQuery) -> list[OrderSummaryView]:
        return self.read_db.query(
            "SELECT * FROM order_summaries WHERE customer_id = :cid",
            {"cid": query.customer_id}
        )
```

**Projection (Read Model Updater):**
```python
class OrderSummaryProjection:
    def __init__(self, read_db: ReadDatabase):
        self.read_db = read_db

    def on_order_placed(self, event: OrderPlaced) -> None:
        self.read_db.upsert("order_summaries", {
            "order_id": event.order_id,
            "customer_name": event.customer_name,
            "total": event.total,
            "item_count": event.item_count,
            "status": "placed",
        })

    def on_order_shipped(self, event: OrderShipped) -> None:
        self.read_db.update("order_summaries",
            where={"order_id": event.order_id},
            set={"status": "shipped"},
        )
```

### Common Mistakes
1. **CQRS everywhere** — Applying CQRS to simple domains where a single model works fine
2. **Ignoring eventual consistency** — Building UI that assumes instant read-after-write consistency
3. **Complex command bus without need** — Adding mediator/command bus infrastructure for simple applications
4. **Not versioning events** — Events change over time; without versioning, projections break
5. **Coupling command and query models** — The read model should be designed for query needs, not mirror the write model

### Integration with Other Topics
- **Event Sourcing** — Natural companion; events from ES feed CQRS projections
- **Domain-Driven Design** — The command side uses DDD aggregates; the query side is purpose-built
- **Event-Driven Architecture** — Events bridge command and query sides
- **Microservices** — CQRS enables independent scaling of read and write services
- **Database Scaling** — Read replicas, materialized views align with CQRS read model

---

## Resources

### Essential Reading
- Greg Young's "CQRS Documents" (online)
- *Implementing Domain-Driven Design* — Vaughn Vernon (CQRS chapters)
- Martin Fowler's "CQRS" bliki article

### Online Resources
- Greg Young's talks on CQRS and Event Sourcing
- Microsoft's CQRS pattern documentation (Azure Architecture Center)
- Udi Dahan's blog on CQRS and messaging

### Practice Exercises
1. Refactor a CRUD service into separate command and query handlers
2. Build a read model projection from domain events
3. Implement optimistic UI updates for an eventually consistent read model
4. Design different read models for the same write model (admin vs. user views)
5. Add a new read model without touching the write side
