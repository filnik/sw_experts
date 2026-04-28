# Modular Monolith

## Profile

### What It Is
A Modular Monolith is a single deployable unit that is internally structured into well-defined, loosely coupled modules. Each module encapsulates a specific business capability with clear boundaries, its own data, and explicit interfaces — providing many benefits of microservices without the distributed systems complexity.

### Origin and Evolution
- Reaction to premature microservices adoption and "distributed monolith" failures
- Championed by practitioners like Kamil Grzybowski, Simon Brown, and Sam Newman
- Aligns with the "monolith-first" approach recommended by Fowler and others
- Increasingly seen as the pragmatic default for new projects
- Often serves as a stepping stone toward eventual microservices extraction

### Key Authors and References
- **Simon Brown** — "Modular Monoliths" talks and C4 model
- **Kamil Grzybowski** — Modular Monolith with DDD (practical implementation)
- **Sam Newman** — "Start with a monolith" guidance in *Building Microservices*
- **Vlad Khononov** — *Learning Domain-Driven Design* (modular decomposition)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Module | Self-contained unit with its own domain, data, and public API |
| Module boundary | Explicit interface; no reaching into module internals |
| Internal data ownership | Each module owns its database schema/tables |
| In-process communication | Modules communicate via function calls or events, not HTTP |
| Single deployment unit | One artifact deployed as one process |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Strong module boundaries** — Each module has a public API. Internal types, data, and implementation are not accessible from outside.
2. **Data ownership** — Each module owns its data (separate schemas or table prefixes). No cross-module joins.
3. **Explicit dependencies** — Module dependencies are declared and minimized. Circular dependencies are forbidden.
4. **In-process communication** — Modules communicate through method calls, in-process events, or a mediator — not HTTP or queues.
5. **Deployable as one unit** — Despite internal modularity, the system deploys as a single process, avoiding distributed systems complexity.

### When to Use vs. When to Avoid

**Use when:**
- Starting a new project (monolith-first strategy)
- Team size is small to medium (2-15 developers)
- Domain boundaries are emerging but not fully stable
- Operational simplicity is valued
- The organization isn't ready for microservices infrastructure
- You want microservices-like boundaries without the overhead

**Avoid when:**
- Different modules genuinely need different scaling characteristics
- Different modules need different technology stacks
- Teams are distributed and need fully independent deployment
- The system is already microservices and working well

---

## Frameworks and Methodologies

### 1. Module Design — step-by-step
1. Identify bounded contexts through event storming or domain analysis
2. Create a module for each bounded context
3. Define the public API for each module (interfaces, DTOs)
4. Separate database schemas per module (or table prefixes)
5. Implement inter-module communication (direct calls for queries, events for reactions)
6. Enforce boundaries with architectural tests

### 2. Modular Monolith to Microservices — step-by-step
1. Verify modules are truly independent (no shared data, clean APIs)
2. Identify the module with the highest scaling/deployment need
3. Extract it into a separate service with its own database
4. Replace in-process calls with HTTP/gRPC or async events
5. Add infrastructure (service discovery, monitoring) for the new service
6. Repeat for the next module that justifies extraction

---

## How to Apply

### Decision Checklist
- [ ] Is each module's public API clearly defined and enforced?
- [ ] Does each module own its data exclusively?
- [ ] Are there no circular dependencies between modules?
- [ ] Can a module be extracted to a microservice if needed?
- [ ] Are boundaries tested with architecture fitness functions?

### Implementation Patterns

**Module Structure:**
```
src/
  modules/
    orders/
      api/              # Public interface (what other modules can call)
        order_module.py
        types.py        # DTOs shared with other modules
      domain/           # Internal domain model
        order.py
        order_item.py
      infrastructure/   # Internal adapters
        postgres_repo.py
      events/           # Events published by this module
        order_events.py
    inventory/
      api/
        inventory_module.py
        types.py
      domain/
      infrastructure/
      events/
    shared/             # Truly shared kernel (minimal)
      event_bus.py
      base_types.py
```

**Module Public API:**
```python
# orders/api/order_module.py — the ONLY entry point
class OrderModule:
    def __init__(self, repo: OrderRepository, event_bus: EventBus):
        self.repo = repo
        self.event_bus = event_bus

    def place_order(self, command: PlaceOrderCommand) -> OrderId:
        """Public API: place a new order."""
        order = Order.create(command.customer_id, command.items)
        self.repo.save(order)
        self.event_bus.publish(OrderPlaced(order_id=order.id))
        return order.id

    def get_order_summary(self, order_id: OrderId) -> OrderSummaryDTO:
        """Public API: get order summary for display."""
        order = self.repo.find_by_id(order_id)
        return OrderSummaryDTO.from_domain(order)
```

**Inter-Module Communication:**
```python
# Inventory module reacts to order events (in-process)
class InventoryModule:
    def __init__(self, repo: InventoryRepository):
        self.repo = repo

    def on_order_placed(self, event: OrderPlaced) -> None:
        """React to order placed by reserving inventory."""
        for item in event.items:
            stock = self.repo.find_by_product_id(item.product_id)
            stock.reserve(item.quantity)
            self.repo.save(stock)
```

**Boundary Enforcement (Architecture Test):**
```python
# Using ArchUnit-style tests
def test_modules_respect_boundaries():
    """No module should import another module's internal types."""
    for module in ["orders", "inventory", "payments"]:
        internal_imports = find_imports(
            from_module=f"src.modules",
            to_pattern=f"src.modules.{module}.domain",
            exclude=f"src.modules.{module}",
        )
        assert internal_imports == [], f"Boundary violation: {internal_imports}"
```

### Common Mistakes
1. **Shared database tables** — Modules querying each other's tables; defeats data ownership
2. **No boundary enforcement** — Module boundaries exist in theory but nothing prevents violations
3. **God module** — One module that everything depends on, becoming a bottleneck
4. **Over-modularizing** — Creating 30 modules for a simple application; start with 3-5
5. **Treating modules as microservices** — Adding HTTP between modules within the same process; unnecessary overhead

### Integration with Other Topics
- **Domain-Driven Design** — Modules map to bounded contexts
- **Microservices** — Modular monolith is the natural precursor; extract when needed
- **Clean Architecture** — Each module can use clean architecture internally
- **Hexagonal Architecture** — Each module has its own ports and adapters
- **Vertical Slice Architecture** — Can complement modular monolith (slices within modules)
- **Team Topologies** — Module ownership aligns with stream-aligned teams

---

## Resources

### Essential Reading
- *Building Microservices* (2nd ed.) — Sam Newman (monolith-first chapter)
- *Learning Domain-Driven Design* — Vlad Khononov (modular decomposition)
- Kamil Grzybowski's "Modular Monolith with DDD" series

### Online Resources
- Simon Brown's "Modular Monoliths" conference talks
- Shopify Engineering blog on modular monolith
- Mauro Servienti's "All Our Aggregates Are Wrong" talk

### Practice Exercises
1. Structure a new application as a modular monolith with 3 modules
2. Enforce module boundaries with architecture tests
3. Implement in-process event communication between modules
4. Extract a module into a separate service
5. Separate database schemas per module in an existing monolith
