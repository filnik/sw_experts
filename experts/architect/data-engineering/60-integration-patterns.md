# Integration Patterns

## Profile

### What It Is
Integration Patterns are proven solutions for connecting disparate systems, services, and data sources. They address the challenges of heterogeneous systems communicating through different protocols, data formats, and interaction styles — from simple point-to-point to complex enterprise integration scenarios.

### Origin and Evolution
- Enterprise Application Integration (EAI) emerged in the 1990s
- Gregor Hohpe & Bobby Woolf's *Enterprise Integration Patterns* (2003) — definitive catalog
- SOA and ESB (Enterprise Service Bus) era (2000s)
- Modern: API-first integration, event-driven integration, iPaaS (Zapier, MuleSoft)
- Current: webhook-based integration, GraphQL federation, AsyncAPI

### Key Authors and References
- **Gregor Hohpe & Bobby Woolf** — *Enterprise Integration Patterns* (2003)
- **Martin Fowler** — Integration patterns articles
- **Sam Newman** — Service integration in *Building Microservices*

### Key Concepts at a Glance
| Pattern | Description |
|---------|-------------|
| Message Channel | Pipe connecting sender to receiver |
| Message Router | Routes messages based on conditions |
| Message Translator | Converts between data formats |
| Message Filter | Drops unwanted messages |
| Content-Based Router | Routes based on message content |
| Aggregator | Combines multiple messages into one |
| Scatter-Gather | Sends to multiple recipients, collects responses |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Loose coupling between systems** — Integrated systems should be able to change independently. Don't create tight dependencies.
2. **Prefer asynchronous over synchronous** — Async integration (events, messages) is more resilient than synchronous API calls.
3. **Anti-corruption layer** — Don't let external system concepts pollute your domain model. Translate at the boundary.
4. **Idempotent receivers** — Messages may be delivered more than once. Receivers must handle duplicates safely.
5. **Design for failure** — External systems will be unavailable. Design with retries, dead letter queues, and fallbacks.

### When to Use vs. When to Avoid

**Complex integration patterns when:**
- Multiple systems with different data models
- Reliability and ordering guarantees needed
- Complex routing or transformation logic
- High-volume message processing

**Simple approaches when:**
- Two systems with a direct API call
- Low volume, synchronous request-response suffices
- Integration is temporary or one-off

---

## Frameworks and Methodologies

### 1. Integration Design — step-by-step
1. Map all systems and their integration points
2. Classify each integration: sync vs. async, frequency, volume
3. Define data contracts between systems
4. Choose patterns: API call, event, file transfer, shared database
5. Implement anti-corruption layers at boundaries
6. Add monitoring, alerting, and dead letter handling

### 2. Anti-Corruption Layer — step-by-step
1. Define your domain model (internal language)
2. Map external system's model to your model (translation)
3. Create a translator/adapter at the boundary
4. Never expose external system types to your domain
5. Handle external system failures gracefully
6. Test with mocked external system responses

---

## How to Apply

### Decision Checklist
- [ ] Is each integration point documented with data contracts?
- [ ] Are anti-corruption layers in place for external systems?
- [ ] Are async integrations idempotent?
- [ ] Is there error handling and dead letter processing?
- [ ] Can each integration be monitored independently?

### Implementation Patterns

**Anti-Corruption Layer:**
```python
# External payment system returns its own types
class StripePaymentAdapter:
    """ACL: translates Stripe concepts to our domain."""

    def __init__(self, stripe_client: StripeClient):
        self.client = stripe_client

    def charge(self, amount: Money, method: PaymentMethod) -> PaymentResult:
        # Translate our domain to Stripe's API
        stripe_response = self.client.create_charge(
            amount=int(amount.cents),
            currency=amount.currency.lower(),
            source=method.stripe_token,
        )
        # Translate Stripe's response to our domain
        return PaymentResult(
            id=PaymentId(stripe_response.id),
            status=self._map_status(stripe_response.status),
            amount=amount,
        )

    def _map_status(self, stripe_status: str) -> PaymentStatus:
        return {"succeeded": PaymentStatus.COMPLETED,
                "pending": PaymentStatus.PENDING,
                "failed": PaymentStatus.FAILED}[stripe_status]
```

**Webhook Integration with Idempotency:**
```python
@app.post("/webhooks/stripe")
async def handle_stripe_webhook(request: Request):
    event = verify_webhook_signature(request)

    # Idempotency: skip if already processed
    if await is_event_processed(event.id):
        return {"status": "already_processed"}

    match event.type:
        case "payment_intent.succeeded":
            await handle_payment_success(event.data)
        case "payment_intent.payment_failed":
            await handle_payment_failure(event.data)

    await mark_event_processed(event.id)
    return {"status": "processed"}
```

### Common Mistakes
1. **Shared database integration** — Two services sharing a database table; tight coupling, no boundaries
2. **No anti-corruption layer** — External system's data model leaks into domain code
3. **Synchronous chain** — A → B → C → D synchronously; fragile, slow, cascading failures
4. **No idempotency** — Webhook retries create duplicate records
5. **Missing error handling** — External system failure crashes the integration with no recovery

### Integration with Other Topics
- **Domain-Driven Design** — ACL is a DDD strategic pattern for bounded contexts
- **Microservices** — Integration patterns connect microservices
- **Event-Driven Architecture** — Events are the primary async integration mechanism
- **API Design** — APIs are the synchronous integration interface
- **Resilience Patterns** — Circuit breakers and retries for integration reliability
- **Messaging** — Message brokers provide the plumbing for async integration

---

## Resources

### Essential Reading
- *Enterprise Integration Patterns* — Hohpe & Woolf
- *Building Microservices* — Sam Newman (integration chapters)
- *Domain-Driven Design* — Eric Evans (anti-corruption layer)

### Online Resources
- enterpriseintegrationpatterns.com — Pattern catalog
- Apache Camel documentation (implements EIP)
- MuleSoft integration patterns guide

### Practice Exercises
1. Implement an anti-corruption layer for a third-party API
2. Build an idempotent webhook handler
3. Design an event-driven integration between 3 services
4. Implement a content-based router for message routing
5. Add dead letter queue handling for a message consumer
