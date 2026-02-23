# Hexagonal Architecture (Ports & Adapters)

## Profile

### What It Is
Hexagonal Architecture (also known as Ports & Adapters) is an architectural pattern that isolates the application core from external concerns by defining explicit boundaries through ports (interfaces) and adapters (implementations). The core application drives and is driven by the outside world exclusively through these ports.

### Origin and Evolution
- Conceived by Alistair Cockburn in 2005
- Evolved from frustration with traditional layered architectures where business logic leaked into UI and database layers
- Influenced Clean Architecture (Martin, 2012) and Onion Architecture (Palermo, 2008)
- Widely adopted in DDD communities and microservices architectures

### Key Authors and References
- **Alistair Cockburn** — Original article "Hexagonal Architecture" (2005)
- **Robert C. Martin** — Extended concepts in Clean Architecture
- **Vaughn Vernon** — Practical application in *Implementing Domain-Driven Design*
- **Herberto Graca** — Detailed blog series on Ports & Adapters

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Port | An interface defining how the application interacts with the outside |
| Adapter | A concrete implementation of a port for a specific technology |
| Driving Port | Port through which external actors drive the application (inbound) |
| Driven Port | Port through which the application drives external systems (outbound) |
| Application Core | Business logic with zero external dependencies |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Symmetry of driving and driven sides** — The application is at the center; external actors (users, tests, other systems) interact symmetrically through ports on any side
2. **Technology independence** — The application core contains no reference to any specific technology, framework, or infrastructure
3. **Ports define the application boundary** — Every interaction crosses the boundary through an explicit port; no backdoor access
4. **Adapters are replaceable** — Any adapter can be swapped without affecting the core, enabling easy testing and technology migration
5. **Test-first design** — The architecture naturally supports testing by substituting real adapters with test doubles

### When to Use vs. When to Avoid

**Use when:**
- The application has significant business logic worth protecting
- You need to support multiple entry points (REST, CLI, events, tests)
- Technology decisions should be deferrable
- The system will evolve and adapters will change over time
- Integration testing with real infrastructure is expensive

**Avoid when:**
- The application is a thin wrapper around a database (CRUD)
- There's only one possible adapter per port (no real benefit from abstraction)
- The team is small and the system is simple
- Speed of initial delivery outweighs long-term flexibility

---

## Frameworks and Methodologies

### 1. Port Discovery — step-by-step
1. List all external actors that interact with the application
2. Classify each as "driving" (initiates action) or "driven" (receives action)
3. For each actor, define a port interface expressing the interaction
4. Group driving ports into application services (use cases)
5. Group driven ports by capability (persistence, messaging, notifications)

### 2. Adapter Implementation — step-by-step
1. Choose a technology for each driven port (e.g., PostgreSQL for persistence)
2. Implement the adapter satisfying the port interface exactly
3. Create a test adapter (in-memory, fake) for each driven port
4. Wire adapters to ports at the composition root
5. Verify the application works with both real and test adapters

---

## How to Apply

### Decision Checklist
- [ ] Are all external interactions going through explicit ports?
- [ ] Does the application core import zero infrastructure code?
- [ ] Can I run the full application with in-memory adapters?
- [ ] Is each adapter independently replaceable?
- [ ] Are driving and driven ports clearly separated?

### Implementation Patterns

**Port Definitions:**
```python
# Driving port (inbound) — what the outside can ask the app to do
class OrderService(Protocol):
    def place_order(self, command: PlaceOrderCommand) -> OrderId: ...
    def cancel_order(self, order_id: OrderId) -> None: ...

# Driven port (outbound) — what the app needs from the outside
class OrderRepository(Protocol):
    def save(self, order: Order) -> None: ...
    def find_by_id(self, order_id: OrderId) -> Order | None: ...

class PaymentGateway(Protocol):
    def charge(self, amount: Money, method: PaymentMethod) -> PaymentResult: ...
```

**Adapters:**
```python
# Driving adapter — REST controller
class OrderController:
    def __init__(self, order_service: OrderService):
        self.order_service = order_service

    def post_order(self, request: Request) -> Response:
        command = PlaceOrderCommand.from_request(request)
        order_id = self.order_service.place_order(command)
        return Response(status=201, body={"order_id": str(order_id)})

# Driven adapter — PostgreSQL repository
class PostgresOrderRepository:
    def __init__(self, session: Session):
        self.session = session

    def save(self, order: Order) -> None:
        self.session.add(OrderModel.from_domain(order))

    def find_by_id(self, order_id: OrderId) -> Order | None:
        row = self.session.get(OrderModel, str(order_id))
        return row.to_domain() if row else None

# Test adapter — in-memory repository
class InMemoryOrderRepository:
    def __init__(self):
        self.orders: dict[OrderId, Order] = {}

    def save(self, order: Order) -> None:
        self.orders[order.id] = order

    def find_by_id(self, order_id: OrderId) -> Order | None:
        return self.orders.get(order_id)
```

### Common Mistakes
1. **Port pollution** — Defining ports that mirror database schemas instead of domain operations
2. **Adapter leakage** — Letting adapter-specific exceptions or types cross into the core
3. **Missing driven ports** — Calling external services directly from the core without a port
4. **Monolithic composition root** — One giant file wiring hundreds of dependencies instead of modular configuration
5. **Confusing layers with hexagonal** — Treating it as a traditional layered architecture with renamed layers

### Integration with Other Topics
- **Clean Architecture** — Clean Architecture is hexagonal with explicitly named concentric layers
- **Domain-Driven Design** — The application core maps to DDD's domain and application layers
- **SOLID Principles** — Ports are ISP; adapters implement DIP; the pattern embodies OCP
- **Microservices** — Each microservice's internal structure can be hexagonal
- **Test-Driven Development** — In-memory adapters enable fast, isolated unit testing

---

## Resources

### Essential Reading
- *Hexagonal Architecture* — Alistair Cockburn (original article)
- *Get Your Hands Dirty on Clean Architecture* — Tom Hombergs
- *Implementing Domain-Driven Design* — Vaughn Vernon (chapters on architecture)

### Online Resources
- Alistair Cockburn's wiki on Hexagonal Architecture
- Herberto Graca's "DDD, Hexagonal, Onion, Clean, CQRS" blog series
- Netflix engineering blog on hexagonal microservices

### Practice Exercises
1. Design ports for an e-commerce checkout flow
2. Implement the same application with REST and CLI driving adapters
3. Swap a PostgreSQL adapter for an in-memory adapter in tests
4. Draw the hexagonal diagram for an existing application
5. Refactor a framework-coupled service into hexagonal structure
