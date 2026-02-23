# Microservices Architecture

## Profile

### What It Is
Microservices Architecture is an architectural style that structures an application as a collection of small, autonomous services, each running in its own process, communicating via lightweight protocols, and organized around business capabilities. Each service is independently deployable, scalable, and maintainable.

### Origin and Evolution
- Emerged from Service-Oriented Architecture (SOA) with focus on independence and autonomy
- Popularized by Netflix, Amazon, and Spotify in the early 2010s
- Named and formalized by James Lewis and Martin Fowler (2014)
- Evolved alongside containerization (Docker), orchestration (Kubernetes), and DevOps practices
- Current trend: right-sizing services, avoiding "nanoservices"

### Key Authors and References
- **Sam Newman** — *Building Microservices* (2015, 2nd ed. 2021)
- **Martin Fowler & James Lewis** — "Microservices" article (2014)
- **Chris Richardson** — *Microservices Patterns* (2018)
- **Neal Ford, Rebecca Parsons, Patrick Kua** — *Building Evolutionary Architectures*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Service boundary | Autonomous unit organized around a business capability |
| Independent deployment | Each service deploys without coordinating with others |
| Decentralized data | Each service owns its data store |
| Smart endpoints, dumb pipes | Business logic in services, simple communication |
| Design for failure | Assume everything can fail; build resilience in |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Single responsibility per service** — Each service does one thing well, aligned with a bounded context or business capability
2. **Decentralized everything** — Decentralized governance, data management, and technology choices. Each team owns their service end-to-end
3. **Design for failure** — Services will fail. Design for graceful degradation with circuit breakers, retries, timeouts, and fallbacks
4. **Independent deployability** — The primary goal. If you can't deploy services independently, you don't have microservices
5. **Evolutionary design** — Services can be rewritten, split, or merged as understanding evolves. Avoid big upfront design

### When to Use vs. When to Avoid

**Use when:**
- The organization has multiple autonomous teams
- Different parts of the system have different scaling needs
- Different services benefit from different technology stacks
- Independent deployment cadence is important
- The domain is well-understood and bounded contexts are clear

**Avoid when:**
- The team is small (< 8-10 developers)
- The domain is not well-understood (start monolith-first)
- The organization lacks DevOps maturity (CI/CD, monitoring, containerization)
- Network latency and distributed transactions are unacceptable
- You're optimizing for initial development speed over long-term flexibility

---

## Frameworks and Methodologies

### 1. Decomposition by Business Capability — step-by-step
1. Identify core business capabilities (what the business does, not how)
2. Map capabilities to bounded contexts (DDD context mapping)
3. Define service boundaries: one service per bounded context
4. Identify shared data and design integration patterns (events, APIs)
5. Define service contracts (API schemas, event schemas)
6. Assign team ownership per service

### 2. Strangler Fig Migration — step-by-step
1. Identify a module in the monolith to extract
2. Build the new microservice alongside the monolith
3. Route traffic for that capability to the new service (facade/proxy)
4. Migrate data from shared database to service-owned store
5. Remove the old module from the monolith
6. Repeat for the next module

---

## How to Apply

### Decision Checklist
- [ ] Can each service be deployed independently?
- [ ] Does each service own its data?
- [ ] Are service boundaries aligned with business capabilities?
- [ ] Is there a robust CI/CD pipeline for each service?
- [ ] Is observability (logging, tracing, metrics) in place?
- [ ] Are failure modes handled (circuit breakers, retries)?

### Implementation Patterns

**Service Communication:**
```
Synchronous:
  - REST/HTTP for simple request-response
  - gRPC for high-performance internal communication

Asynchronous:
  - Events via message broker (Kafka, RabbitMQ)
  - Preferred for decoupling and resilience
```

**API Gateway Pattern:**
```yaml
# API Gateway routes requests to appropriate services
routes:
  /api/orders/*:    order-service
  /api/products/*:  catalog-service
  /api/users/*:     user-service
  /api/payments/*:  payment-service
```

**Database per Service:**
```
order-service     → PostgreSQL (relational, ACID transactions)
catalog-service   → MongoDB (flexible schema, read-heavy)
search-service    → Elasticsearch (full-text search)
analytics-service → ClickHouse (columnar, analytics)
```

**Inter-Service Event Communication:**
```python
# Order service publishes event
class OrderService:
    def place_order(self, command: PlaceOrderCommand) -> OrderId:
        order = Order.create(command)
        self.repo.save(order)
        self.event_bus.publish(OrderPlaced(
            order_id=order.id,
            customer_id=order.customer_id,
            total=order.total,
        ))
        return order.id

# Inventory service reacts to event
class InventoryEventHandler:
    def on_order_placed(self, event: OrderPlaced) -> None:
        for item in event.items:
            self.inventory.reserve(item.product_id, item.quantity)
```

### Common Mistakes
1. **Distributed monolith** — Services that must be deployed together and share a database; all the pain of microservices with none of the benefits
2. **Too small services** — "Nanoservices" that create massive operational overhead for minimal benefit
3. **Shared database** — Multiple services reading/writing the same database tables
4. **Synchronous chains** — Service A calls B calls C calls D synchronously; fragile and slow
5. **Missing observability** — Without distributed tracing and centralized logging, debugging is impossible
6. **Starting with microservices** — Building microservices before understanding the domain leads to wrong boundaries

### Integration with Other Topics
- **Domain-Driven Design** — Bounded contexts define service boundaries
- **Event-Driven Architecture** — Asynchronous communication between services
- **CQRS** — Applied within individual services
- **Saga Pattern** — Manages distributed transactions across services
- **API Gateway/BFF** — Routes and aggregates requests at the edge
- **Containerization** — Docker + Kubernetes enable microservices deployment
- **CI/CD Pipelines** — Independent deployment requires mature CI/CD
- **Observability** — Essential for operating distributed services

---

## Resources

### Essential Reading
- *Building Microservices* (2nd ed.) — Sam Newman
- *Microservices Patterns* — Chris Richardson
- *Monolith to Microservices* — Sam Newman

### Online Resources
- microservices.io — Chris Richardson's pattern catalog
- Martin Fowler's "Microservices" article
- Netflix Tech Blog on microservices architecture

### Practice Exercises
1. Decompose a monolith into service boundaries using bounded contexts
2. Implement inter-service communication with events
3. Add circuit breaker and retry logic to a service call
4. Set up independent CI/CD pipelines for two services
5. Implement the Strangler Fig pattern to extract a service from a monolith
