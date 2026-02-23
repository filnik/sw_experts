# Saga Pattern

## Profile

### What It Is
The Saga Pattern is a design pattern for managing distributed transactions across multiple services or bounded contexts. Instead of a single ACID transaction, a saga breaks a business process into a sequence of local transactions, each with a compensating action that undoes its effect if a later step fails.

### Origin and Evolution
- Originally described by Hector Garcia-Molina and Kenneth Salem (1987) for long-lived database transactions
- Rediscovered and popularized in the microservices era by Chris Richardson and Caitie McCaffrey
- Two main variants emerged: choreography-based and orchestration-based sagas
- Central to event-driven microservices architectures
- Modern implementations use frameworks like Temporal, Axon, and MassTransit

### Key Authors and References
- **Hector Garcia-Molina & Kenneth Salem** — Original saga paper (1987)
- **Chris Richardson** — *Microservices Patterns* (2018, saga chapters)
- **Caitie McCaffrey** — "Applying the Saga Pattern" talk
- **Bernd Ruecker** — *Practical Process Automation* (orchestration focus)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Local Transaction | A step in the saga executed within a single service |
| Compensating Transaction | An action that semantically undoes a completed step |
| Choreography | Each service listens to events and decides what to do next |
| Orchestration | A central coordinator tells each service what to do |
| Pivot Transaction | The point of no return; after this, only forward |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **No distributed ACID** — Distributed transactions (2PC) don't scale. Sagas embrace eventual consistency with compensation.
2. **Every action has an undo** — Each step must have a compensating transaction that reverses its business effect.
3. **Eventual consistency is acceptable** — The system may be temporarily inconsistent during saga execution but reaches a consistent state eventually.
4. **Failure is expected** — Saga design assumes steps can fail and plans the rollback path explicitly.
5. **Idempotency is mandatory** — Both forward and compensating transactions must be safe to retry.

### When to Use vs. When to Avoid

**Use when:**
- A business process spans multiple services/databases
- Each service owns its data and ACID transactions are local only
- The process can tolerate temporary inconsistency
- Compensation logic is well-defined for each step

**Avoid when:**
- The process can be handled within a single transaction
- Compensation is impossible or too complex (e.g., sending a physical letter)
- Strong consistency is required at every point in time
- The number of steps is very large (saga complexity grows quickly)

---

## Frameworks and Methodologies

### 1. Choreography-Based Saga — step-by-step
1. Define the business process as a sequence of events
2. Each service listens for relevant events and executes its local transaction
3. On success, the service publishes a success event
4. On failure, the service publishes a failure event
5. Other services listen for failure events and execute compensating transactions
6. Design dead-letter handling for unprocessable events

### 2. Orchestration-Based Saga — step-by-step
1. Define the saga orchestrator (coordinator) with explicit step definitions
2. Orchestrator sends commands to each service in sequence
3. Each service executes and reports success/failure to the orchestrator
4. On failure, orchestrator triggers compensating commands in reverse order
5. Orchestrator tracks saga state (pending, completed, compensating, failed)
6. Implement timeout handling for non-responsive services

---

## How to Apply

### Decision Checklist
- [ ] Is compensation clearly defined for every step?
- [ ] Are all steps idempotent (safe to retry)?
- [ ] Is the saga state persisted (survives process crashes)?
- [ ] Are timeouts defined for each step?
- [ ] Is there monitoring/alerting for stuck sagas?

### Implementation Patterns

**Orchestration-Based Saga:**
```python
class CreateOrderSaga:
    steps = [
        SagaStep(
            action=lambda ctx: ctx.order_service.create_order(ctx.order),
            compensation=lambda ctx: ctx.order_service.cancel_order(ctx.order_id),
        ),
        SagaStep(
            action=lambda ctx: ctx.payment_service.charge(ctx.payment),
            compensation=lambda ctx: ctx.payment_service.refund(ctx.payment_id),
        ),
        SagaStep(
            action=lambda ctx: ctx.inventory_service.reserve(ctx.items),
            compensation=lambda ctx: ctx.inventory_service.release(ctx.reservation_id),
        ),
        SagaStep(  # Pivot transaction — no compensation needed after this
            action=lambda ctx: ctx.order_service.confirm_order(ctx.order_id),
            compensation=None,
        ),
    ]

class SagaOrchestrator:
    def execute(self, saga: Saga, context: SagaContext) -> SagaResult:
        completed_steps = []
        for step in saga.steps:
            try:
                step.action(context)
                completed_steps.append(step)
            except Exception as e:
                self._compensate(completed_steps, context)
                return SagaResult.failed(e)
        return SagaResult.completed()

    def _compensate(self, completed: list[SagaStep], ctx: SagaContext) -> None:
        for step in reversed(completed):
            if step.compensation:
                step.compensation(ctx)
```

**Choreography-Based Saga:**
```python
# Order Service
class OrderService:
    def create_order(self, command: CreateOrder) -> None:
        order = Order.create(command)
        self.repo.save(order)
        self.events.publish(OrderCreated(order_id=order.id, items=order.items))

    def on_payment_failed(self, event: PaymentFailed) -> None:
        order = self.repo.find(event.order_id)
        order.cancel("Payment failed")
        self.repo.save(order)

# Payment Service
class PaymentService:
    def on_order_created(self, event: OrderCreated) -> None:
        try:
            payment = self.gateway.charge(event.total)
            self.events.publish(PaymentCompleted(order_id=event.order_id))
        except PaymentError:
            self.events.publish(PaymentFailed(order_id=event.order_id))

# Inventory Service
class InventoryService:
    def on_payment_completed(self, event: PaymentCompleted) -> None:
        try:
            self.reserve_items(event.order_id)
            self.events.publish(InventoryReserved(order_id=event.order_id))
        except InsufficientStock:
            self.events.publish(InventoryReservationFailed(order_id=event.order_id))
```

**Choreography vs. Orchestration Decision:**
```
Choreography:
  + Simple for 2-4 steps
  + No single point of failure
  - Hard to track overall saga state
  - Difficult to debug with many steps
  - Cyclic event dependencies possible

Orchestration:
  + Clear, centralized workflow logic
  + Easy to monitor and debug
  + Handles complex branching
  - Orchestrator is a potential bottleneck
  - Risk of becoming a "god service"
```

### Common Mistakes
1. **Missing compensations** — Not defining undo logic for every step; saga can get stuck in inconsistent state
2. **Non-idempotent steps** — Retrying a step creates duplicates (e.g., charging twice)
3. **No saga state persistence** — Process crash loses saga progress; restart doesn't know where to resume
4. **Ignoring timeouts** — A non-responding service blocks the saga indefinitely
5. **Choreography spaghetti** — Too many services with event-based choreography becomes impossible to follow

### Integration with Other Topics
- **Microservices Architecture** — Sagas manage cross-service transactions
- **Event-Driven Architecture** — Events trigger saga steps and compensations
- **Event Sourcing** — Saga state can be event-sourced
- **CQRS** — Read models may be temporarily inconsistent during saga execution
- **Resilience Patterns** — Retries, circuit breakers complement saga reliability
- **Messaging and Event Streaming** — Message brokers transport saga commands/events

---

## Resources

### Essential Reading
- *Microservices Patterns* — Chris Richardson (chapters 4-5)
- *Practical Process Automation* — Bernd Ruecker
- Garcia-Molina & Salem — "Sagas" (1987 paper)

### Online Resources
- microservices.io — Saga pattern documentation
- Caitie McCaffrey's "Applying the Saga Pattern" talk
- Temporal.io documentation (modern saga/workflow engine)

### Practice Exercises
1. Design a saga for an e-commerce order flow (create order, charge, reserve inventory, ship)
2. Implement compensating transactions for each step
3. Convert a choreography-based saga to orchestration-based
4. Handle a timeout scenario where a service doesn't respond
5. Add saga state persistence that survives process restarts
