# Event Sourcing

## Profile

### What It Is
Event Sourcing is a data persistence pattern where the state of an entity is determined by a sequence of events rather than by storing current state. Instead of updating a row in a database, every state change is captured as an immutable event. The current state is derived by replaying events from the beginning.

### Origin and Evolution
- Conceptual roots in accounting (ledger-based record keeping)
- Formalized in software by Greg Young and the DDD community (~2006-2010)
- Influenced by Martin Fowler's "Event Sourcing" pattern description
- Adopted in financial systems, audit-critical domains, and event-driven architectures
- Modern implementations leverage event stores like EventStoreDB, Axon, Marten

### Key Authors and References
- **Greg Young** — Primary advocate, EventStoreDB creator
- **Martin Fowler** — Pattern description and documentation
- **Vaughn Vernon** — Practical implementation in *Implementing DDD*
- **Adam Dymitruk** — Event Modeling methodology

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Event | Immutable record of something that happened in the past |
| Event Store | Append-only log of events, partitioned by aggregate/stream |
| Stream | Sequence of events for a single aggregate instance |
| Projection | Derived read model built by processing events |
| Snapshot | Cached state at a point in time to avoid full replay |
| Rehydration | Rebuilding current state by replaying events |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Events are the source of truth** — The event log is the authoritative record; any derived state (projections, caches) can be rebuilt from events
2. **Events are immutable** — Once stored, events never change. Corrections are modeled as compensating events, not mutations
3. **Append-only storage** — The event store only supports appending new events, never updating or deleting existing ones
4. **Temporal queries are natural** — Since all history is preserved, you can reconstruct state at any point in time
5. **Projections are disposable** — Read models can be deleted and rebuilt from the event log at any time

### When to Use vs. When to Avoid

**Use when:**
- Complete audit trail is required (finance, healthcare, legal)
- Temporal queries ("what was the state at time T?") are needed
- The domain naturally expresses as events (order lifecycle, workflow)
- CQRS is already in use and events feed projections
- Debugging requires understanding the full sequence of state changes

**Avoid when:**
- Simple CRUD with no audit requirements
- The domain doesn't naturally map to events
- The team is unfamiliar and there's no time to learn
- Strong consistency requirements conflict with eventual consistency of projections
- Storage costs of keeping all history are prohibitive

---

## Frameworks and Methodologies

### 1. Event Modeling — step-by-step
1. Lay out a timeline of events (what happened in the system)
2. Group events into streams (one per aggregate/entity)
3. Identify commands that produce events
4. Design projections (read models) needed by the UI
5. Map policies (automated reactions: event → command)
6. Validate with domain experts using the visual timeline

### 2. Aggregate + Event Sourcing — step-by-step
1. Define the aggregate and its commands
2. For each command, define the events it produces
3. Implement the `apply` method: given current state + event → new state
4. Implement the `handle` method: given current state + command → events
5. Load aggregate by replaying events from the store
6. Save by appending new events to the store
7. Add snapshots when event count per stream becomes large

---

## How to Apply

### Decision Checklist
- [ ] Does the domain naturally express as a sequence of events?
- [ ] Is a complete audit trail required?
- [ ] Are temporal queries a real requirement?
- [ ] Can the team handle eventual consistency?
- [ ] Is the infrastructure (event store, projections) manageable?

### Implementation Patterns

**Event-Sourced Aggregate:**
```python
class BankAccount:
    def __init__(self):
        self.balance = Decimal("0")
        self.is_open = False
        self._uncommitted_events: list[Event] = []

    # Command handler
    def deposit(self, amount: Decimal) -> None:
        if not self.is_open:
            raise AccountClosed()
        if amount <= 0:
            raise InvalidAmount(amount)
        self._apply(MoneyDeposited(amount=amount))

    def withdraw(self, amount: Decimal) -> None:
        if not self.is_open:
            raise AccountClosed()
        if amount > self.balance:
            raise InsufficientFunds(self.balance, amount)
        self._apply(MoneyWithdrawn(amount=amount))

    # Event applicators
    def _apply(self, event: Event) -> None:
        self._on(event)
        self._uncommitted_events.append(event)

    def _on(self, event: Event) -> None:
        match event:
            case AccountOpened():
                self.is_open = True
            case MoneyDeposited(amount=amount):
                self.balance += amount
            case MoneyWithdrawn(amount=amount):
                self.balance -= amount
            case AccountClosed():
                self.is_open = False

    @classmethod
    def from_events(cls, events: list[Event]) -> "BankAccount":
        account = cls()
        for event in events:
            account._on(event)
        return account
```

**Event Store Interface:**
```python
class EventStore(Protocol):
    def append(self, stream_id: str, events: list[Event],
               expected_version: int) -> None: ...

    def load(self, stream_id: str) -> list[Event]: ...

    def load_from(self, stream_id: str, version: int) -> list[Event]: ...
```

**Projection:**
```python
class AccountBalanceProjection:
    def __init__(self, read_db: ReadDB):
        self.read_db = read_db

    def handle(self, event: Event, metadata: EventMetadata) -> None:
        match event:
            case AccountOpened():
                self.read_db.insert("balances", {
                    "account_id": metadata.stream_id,
                    "balance": 0, "status": "open"
                })
            case MoneyDeposited(amount=amount):
                self.read_db.increment("balances",
                    key=metadata.stream_id, field="balance", by=amount)
            case MoneyWithdrawn(amount=amount):
                self.read_db.increment("balances",
                    key=metadata.stream_id, field="balance", by=-amount)
```

### Common Mistakes
1. **Using events as a messaging system** — Events in event sourcing are for state reconstruction, not inter-service communication (those are integration events)
2. **Mutable events** — Changing stored events breaks the integrity of the system
3. **No event versioning** — Events will evolve; plan for upcasting from the start
4. **Huge aggregates** — Thousands of events per stream without snapshots cause slow load times
5. **CRUD mindset with events** — Events like `UserUpdated(fields={...})` are just CRUD in disguise; model meaningful domain events

### Integration with Other Topics
- **CQRS** — Natural companion; events feed projections for the read model
- **Domain-Driven Design** — Events map to domain events; aggregates are the consistency boundary
- **Event-Driven Architecture** — Event sourcing provides the event log; EDA provides the distribution
- **Saga Pattern** — Sagas coordinate multi-aggregate workflows using events
- **Messaging and Event Streaming** — Kafka/EventStoreDB serve as event stores and distribution

---

## Resources

### Essential Reading
- Greg Young's "Event Sourcing" paper
- *Implementing Domain-Driven Design* — Vaughn Vernon (Event Sourcing chapters)
- *Versioning in an Event Sourced System* — Greg Young

### Online Resources
- EventStoreDB documentation
- Event Modeling (eventmodeling.org) — Adam Dymitruk
- Martin Fowler's "Event Sourcing" article

### Practice Exercises
1. Implement an event-sourced shopping cart aggregate
2. Build a projection that maintains account balances from deposit/withdrawal events
3. Implement snapshotting for an aggregate with many events
4. Handle event schema evolution (add a field to an existing event type)
5. Rebuild a projection from scratch by replaying all events
