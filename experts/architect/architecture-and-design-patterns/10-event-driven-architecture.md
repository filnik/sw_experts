# Event-Driven Architecture

## Profile

### What It Is
Event-Driven Architecture (EDA) is an architectural paradigm where the flow of the program is determined by events — significant changes in state or occurrences. Components produce, detect, consume, and react to events asynchronously, enabling loose coupling, scalability, and real-time responsiveness.

### Origin and Evolution
- Roots in message-oriented middleware (MOM) of the 1990s
- Evolved from publish-subscribe patterns and enterprise integration
- Accelerated by Apache Kafka (2011) making event streaming mainstream
- Modern EDA encompasses event notification, event-carried state transfer, and event sourcing
- Cloud-native event services (AWS EventBridge, Azure Event Grid) made EDA accessible

### Key Authors and References
- **Martin Fowler** — "What do you mean by Event-Driven?" (2017, taxonomy)
- **Gregor Hohpe** — *Enterprise Integration Patterns* (2003)
- **Ben Stopford** — *Designing Event-Driven Systems* (Confluent, 2018)
- **Jay Kreps** — "The Log" blog post, Kafka co-creator

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Event Notification | Minimal signal that something happened; consumer must query for details |
| Event-Carried State Transfer | Event contains all data consumer needs; no callback required |
| Event Sourcing | Events as the primary data store |
| Event Channel | Medium through which events flow (topic, queue, bus) |
| Event Processor | Consumer that reacts to events and may produce new events |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Loose coupling through events** — Producers don't know about consumers. Components interact through events, not direct calls.
2. **Asynchronous by default** — Components communicate without waiting for responses, improving resilience and throughput.
3. **React, don't poll** — Systems respond to state changes as they happen rather than periodically checking for changes.
4. **Events are facts** — An event records something that happened. It's immutable and represents a historical fact.
5. **Choreography over orchestration** — Prefer components reacting to events independently (choreography) over a central coordinator (orchestration) when possible.

### When to Use vs. When to Avoid

**Use when:**
- Components need loose coupling and independent evolution
- Real-time or near-real-time processing is required
- Multiple consumers need to react to the same event
- System must handle variable load (event buffering smooths spikes)
- Audit trail or event history is valuable

**Avoid when:**
- Request-response semantics are required (user waits for result)
- Strong consistency is needed across components
- The system is simple and synchronous calls suffice
- Debugging and tracing complexity isn't worth the decoupling benefit
- The team lacks experience with async patterns

---

## Frameworks and Methodologies

### 1. Event Storming to EDA — step-by-step
1. Conduct event storming: identify domain events on a timeline
2. Identify producers (commands/aggregates that emit events)
3. Identify consumers (policies/projections that react to events)
4. Design event schemas (include event type, timestamp, payload, metadata)
5. Choose event channels: topics for fan-out, queues for competing consumers
6. Define consumer groups and delivery guarantees

### 2. Event-Driven Integration — step-by-step
1. Identify integration points between services/modules
2. Replace synchronous calls with event publication where appropriate
3. Design idempotent consumers (events may be delivered more than once)
4. Implement dead-letter queues for failed event processing
5. Add correlation IDs for distributed tracing
6. Monitor event flow with lag metrics and alerting

---

## How to Apply

### Decision Checklist
- [ ] Is the producer truly independent of how consumers use the event?
- [ ] Are events designed as immutable facts (past tense naming)?
- [ ] Are consumers idempotent (safe to process the same event twice)?
- [ ] Is there a dead-letter strategy for failed events?
- [ ] Are correlation IDs propagated for tracing?

### Implementation Patterns

**Event Schema Design:**
```json
{
  "event_type": "OrderPlaced",
  "event_id": "evt_abc123",
  "timestamp": "2024-01-15T10:30:00Z",
  "correlation_id": "corr_xyz789",
  "aggregate_id": "order_456",
  "version": 1,
  "payload": {
    "order_id": "order_456",
    "customer_id": "cust_789",
    "total": 99.99,
    "items": [
      {"product_id": "prod_001", "quantity": 2, "price": 49.99}
    ]
  }
}
```

**Producer:**
```python
class OrderService:
    def __init__(self, repo: OrderRepository, event_publisher: EventPublisher):
        self.repo = repo
        self.event_publisher = event_publisher

    def place_order(self, command: PlaceOrderCommand) -> OrderId:
        order = Order.create(command)
        self.repo.save(order)
        self.event_publisher.publish(
            topic="orders",
            event=OrderPlaced(
                order_id=order.id,
                customer_id=order.customer_id,
                total=order.total,
                items=order.items,
            ),
        )
        return order.id
```

**Idempotent Consumer:**
```python
class ShippingEventHandler:
    def __init__(self, repo: ShipmentRepository, processed: ProcessedEvents):
        self.repo = repo
        self.processed = processed

    def on_order_placed(self, event: OrderPlaced, metadata: EventMetadata) -> None:
        # Idempotency check
        if self.processed.already_handled(metadata.event_id):
            return

        shipment = Shipment.create_for_order(event.order_id, event.items)
        self.repo.save(shipment)
        self.processed.mark_handled(metadata.event_id)
```

**Topology Patterns:**
```
Fan-out:     1 producer → topic → N consumers (each gets all events)
Competing:   1 producer → queue → N consumers (load-balanced)
Pipeline:    A → event → B → event → C (processing chain)
Saga:        Orchestrator coordinates via events across services
```

### Common Mistakes
1. **Event as command** — Naming events as instructions ("SendEmail") instead of facts ("OrderPlaced")
2. **Missing idempotency** — Consumers that create duplicates when the same event is redelivered
3. **Tight event coupling** — Consumer schemas tightly bound to producer schemas; breaks when producer evolves
4. **No dead-letter handling** — Failed events silently lost, causing data inconsistency
5. **Event soup** — Too many fine-grained events making the system impossible to reason about
6. **Synchronous mindset** — Designing async flows but expecting synchronous consistency

### Integration with Other Topics
- **Event Sourcing** — EDA distributes events; event sourcing stores them
- **CQRS** — Events bridge the write model to read model projections
- **Microservices** — Events decouple services better than synchronous APIs
- **Saga Pattern** — Manages distributed workflows using events
- **Messaging and Event Streaming** — Kafka, RabbitMQ provide the infrastructure
- **Resilience Patterns** — Async events naturally buffer load and isolate failures

---

## Resources

### Essential Reading
- *Designing Event-Driven Systems* — Ben Stopford (free Confluent book)
- *Enterprise Integration Patterns* — Hohpe & Woolf
- Martin Fowler's "What do you mean by Event-Driven?" article

### Online Resources
- Confluent Developer resources on event-driven architecture
- AWS Event-Driven Architecture guides
- AsyncAPI specification for event-driven APIs

### Practice Exercises
1. Replace a synchronous service call with event-based communication
2. Implement an idempotent event consumer with deduplication
3. Build a fan-out topology where 3 services react to one event
4. Add dead-letter queue handling for a consumer
5. Trace an event flow across 3 services using correlation IDs
