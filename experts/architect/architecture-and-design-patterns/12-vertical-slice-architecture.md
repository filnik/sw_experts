# Vertical Slice Architecture

## Profile

### What It Is
Vertical Slice Architecture organizes code by feature (vertical slice) rather than by technical layer (horizontal). Each slice contains everything needed for a specific use case — from the API endpoint through business logic to data access — minimizing coupling between features and maximizing cohesion within each slice.

### Origin and Evolution
- Popularized by Jimmy Bogard (creator of MediatR, AutoMapper)
- Reaction to traditional N-tier/layered architectures where changes span multiple layers
- Influenced by Feature-Driven Development and use-case-driven design
- Often implemented with Mediator pattern (MediatR in .NET, similar in other ecosystems)
- Gaining traction as an alternative to strict Clean Architecture

### Key Authors and References
- **Jimmy Bogard** — Primary advocate, "Vertical Slice Architecture" talks
- **Derek Comartin** — Practical guides and comparisons with Clean Architecture
- **Steven van Deursen & Mark Seemann** — Complementary DI patterns

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Vertical Slice | A complete feature from API to database, self-contained |
| Handler | The central unit: receives request, processes, returns response |
| No shared abstractions | Each slice chooses its own patterns; no forced repository/service layer |
| Mediator | Optional: dispatches requests to handlers, decoupling sender from receiver |
| Feature folder | Directory per feature containing all related code |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Organize by feature, not layer** — Group all code for "Create Order" together, not spread across Controllers/, Services/, Repositories/ folders
2. **Minimize coupling between features** — Changing one feature should not require changes to another. Each slice is independent.
3. **No premature abstractions** — Don't force every feature to use the same repository, service, or pattern. Simple features use simple code.
4. **Right-size complexity per feature** — A complex feature can use DDD patterns internally; a simple CRUD feature can hit the database directly. Each slice decides.
5. **New features are additive** — Adding a feature means adding new files, not modifying existing layers shared by all features.

### When to Use vs. When to Avoid

**Use when:**
- Features have varying complexity (some CRUD, some complex domain logic)
- The team is tired of "layer tax" — changing all layers for simple features
- New features are added frequently
- Code navigation should be feature-oriented
- Different features benefit from different patterns

**Avoid when:**
- Strong domain model is shared heavily across features (DDD aggregate reuse)
- The team prefers consistent architecture patterns everywhere
- The project is small enough that layers aren't burdensome
- Shared cross-cutting concerns would lead to excessive duplication

---

## Frameworks and Methodologies

### 1. Slice Design — step-by-step
1. Identify a feature/use case (e.g., "Place Order", "Get Product Details")
2. Create a feature folder containing request, handler, response, and validation
3. In the handler, implement the complete flow (validation → logic → persistence → response)
4. Choose the right level of abstraction for this specific feature
5. Share only truly cross-cutting concerns (auth, logging, validation pipeline)

### 2. Migration from Layered to Slices — step-by-step
1. Pick one feature in the existing layered architecture
2. Create a feature folder for it
3. Move or copy relevant controller action, service method, and repository call into the slice
4. Inline abstractions that serve only this one feature
5. Delete the old code if no other feature uses it
6. Repeat feature by feature

---

## How to Apply

### Decision Checklist
- [ ] Is each feature self-contained in its own folder?
- [ ] Can I add a new feature without modifying existing slices?
- [ ] Are shared abstractions genuinely shared (used by multiple slices)?
- [ ] Does each slice use appropriate complexity for its needs?
- [ ] Is the mediator/dispatcher adding value, or is it ceremony?

### Implementation Patterns

**Feature Folder Structure:**
```
src/
  features/
    place_order/
      place_order_command.py
      place_order_handler.py
      place_order_validator.py
    get_order/
      get_order_query.py
      get_order_handler.py
    cancel_order/
      cancel_order_command.py
      cancel_order_handler.py
    list_products/
      list_products_query.py
      list_products_handler.py
  shared/
    auth/
    logging/
    database.py
```

**Simple Slice (CRUD):**
```python
# features/list_products/list_products_handler.py
@dataclass(frozen=True)
class ListProductsQuery:
    category: str | None = None
    page: int = 1
    size: int = 20

@dataclass(frozen=True)
class ProductListItem:
    id: str
    name: str
    price: Decimal

class ListProductsHandler:
    def __init__(self, db: Database):
        self.db = db

    def handle(self, query: ListProductsQuery) -> list[ProductListItem]:
        # Simple feature = simple implementation. No repository abstraction needed.
        rows = self.db.query(
            "SELECT id, name, price FROM products WHERE (:cat IS NULL OR category = :cat) LIMIT :size OFFSET :offset",
            {"cat": query.category, "size": query.size, "offset": (query.page - 1) * query.size}
        )
        return [ProductListItem(**row) for row in rows]
```

**Complex Slice (Domain Logic):**
```python
# features/place_order/place_order_handler.py
@dataclass(frozen=True)
class PlaceOrderCommand:
    customer_id: str
    items: list[OrderItemDTO]

class PlaceOrderHandler:
    def __init__(self, order_repo: OrderRepository, event_bus: EventBus,
                 pricing: PricingService):
        self.order_repo = order_repo
        self.event_bus = event_bus
        self.pricing = pricing

    def handle(self, command: PlaceOrderCommand) -> OrderId:
        # Complex feature = rich domain model, patterns as needed
        priced_items = self.pricing.calculate(command.items)
        order = Order.create(command.customer_id, priced_items)
        order.apply_discounts()
        order.validate_inventory()
        self.order_repo.save(order)
        self.event_bus.publish(order.collect_events())
        return order.id
```

**Mediator Integration:**
```python
# Using a mediator to dispatch
class Mediator:
    def __init__(self, handlers: dict[type, Handler]):
        self.handlers = handlers

    def send(self, request: Any) -> Any:
        handler = self.handlers[type(request)]
        return handler.handle(request)

# In API layer
@app.post("/orders")
def create_order(body: CreateOrderBody, mediator: Mediator):
    command = PlaceOrderCommand(customer_id=body.customer_id, items=body.items)
    order_id = mediator.send(command)
    return {"order_id": order_id}
```

### Common Mistakes
1. **Reintroducing layers inside slices** — Each slice having its own controller, service, repository layers defeats the purpose
2. **Over-sharing** — Extracting "shared" code too aggressively, creating hidden coupling between slices
3. **Inconsistent slice size** — Some slices are 5 files, others are 50; keep slices focused on one use case
4. **Mediator as religion** — Using mediator for every interaction, even simple function calls within a slice
5. **Ignoring cross-cutting concerns** — Authentication, logging, validation should still be shared infrastructure, not duplicated per slice

### Integration with Other Topics
- **Clean Architecture** — Can be combined: use clean architecture within complex slices
- **CQRS** — Natural fit: command slices and query slices have different patterns
- **Modular Monolith** — Slices can be organized within modules
- **Domain-Driven Design** — Complex slices use DDD tactical patterns internally
- **Microservices** — Each microservice can use vertical slices internally

---

## Resources

### Essential Reading
- Jimmy Bogard's "Vertical Slice Architecture" talk and blog posts
- Derek Comartin's "Restructuring to a Vertical Slice Architecture" video series

### Online Resources
- Jimmy Bogard's blog (lostechies.com / jimmybogard.com)
- CodeOpinion (Derek Comartin) YouTube channel
- Comparison articles: Vertical Slices vs. Clean Architecture

### Practice Exercises
1. Refactor a layered controller/service/repository into vertical slices
2. Implement two slices: one simple (CRUD) and one complex (domain logic)
3. Add a new feature as a new slice without touching existing code
4. Design cross-cutting concerns (auth, validation) shared across slices
5. Compare the same feature implemented in layered vs. vertical slice style
