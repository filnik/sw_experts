# CAP Theorem and Distributed Tradeoffs

## Profile

### What It Is
The CAP Theorem states that a distributed data store can simultaneously provide only two of three guarantees: Consistency (every read receives the most recent write), Availability (every request receives a response), and Partition Tolerance (the system continues operating despite network partitions). Since network partitions are inevitable, the real choice is between consistency and availability during a partition.

### Origin and Evolution
- Conjectured by Eric Brewer at PODC 2000, formally proven by Gilbert and Lynch (2002)
- Initially oversimplified as "pick 2 of 3" — later refined to "during a partition, choose C or A"
- Extended by Daniel Abadi's PACELC theorem (2012): even without partitions, there's a latency-consistency tradeoff
- Modern understanding: CAP is a spectrum, not a binary choice; systems make nuanced tradeoffs per operation

### Key Authors and References
- **Eric Brewer** — CAP conjecture (2000), "CAP Twelve Years Later" (2012)
- **Seth Gilbert & Nancy Lynch** — Formal proof (2002)
- **Daniel Abadi** — PACELC theorem
- **Martin Kleppmann** — *Designing Data-Intensive Applications* (2017)

### Key Concepts at a Glance
| Property | Description |
|----------|-------------|
| Consistency | Every read reflects the latest write (linearizability) |
| Availability | Every request gets a non-error response (may be stale) |
| Partition Tolerance | System works despite network splits between nodes |
| PACELC | During Partition: A or C; Else: Latency or Consistency |
| Eventual Consistency | Given no new updates, all replicas converge to the same state |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Partitions are inevitable** — In any distributed system, network failures will happen. The question is not if, but when and how you handle them.
2. **The real choice is C vs. A during partitions** — When the network splits, you must choose: reject requests (consistency) or serve potentially stale data (availability).
3. **Consistency is a spectrum** — From strong/linearizable through causal to eventual consistency. Most systems operate somewhere in between.
4. **Tradeoffs are per-operation, not per-system** — A single system can choose consistency for writes and availability for reads, or strong consistency for payments and eventual for analytics.
5. **PACELC extends the picture** — Even when the network is healthy, there's a tradeoff between latency and consistency (synchronous replication is slow but consistent).

### When to Use vs. When to Avoid

**Choose CP (consistency over availability) when:**
- Financial transactions, inventory with strict limits
- Leader election, distributed locks
- Systems where stale data causes incorrect decisions

**Choose AP (availability over consistency) when:**
- Shopping carts, social media feeds, content delivery
- Systems where temporary staleness is acceptable
- High-availability requirements (99.99%+)

---

## Frameworks and Methodologies

### 1. Consistency Model Selection — step-by-step
1. List all data operations in the system
2. For each operation, determine the impact of reading stale data
3. Classify operations: strong consistency required vs. eventual consistency acceptable
4. Choose appropriate replication strategy per classification
5. Document the consistency guarantees in API contracts

### 2. PACELC Analysis — step-by-step
1. For each data store, determine partition handling (PA or PC)
2. For the normal case (no partition), determine latency vs. consistency (EL or EC)
3. Map existing technology choices: e.g., PostgreSQL is PC/EC, Cassandra is PA/EL, MongoDB is PA/EC (tunable)
4. Validate choices against business requirements
5. Document tradeoffs and their business justification

---

## How to Apply

### Decision Checklist
- [ ] Have I identified which operations need strong consistency?
- [ ] Is eventual consistency explicitly designed for (not accidental)?
- [ ] Are consistency guarantees documented for API consumers?
- [ ] Is there a strategy for partition recovery and data reconciliation?
- [ ] Are tradeoffs understood and accepted by stakeholders?

### Implementation Patterns

**Tunable Consistency (Quorum):**
```python
# Quorum-based replication: N replicas, W writes, R reads
# Strong consistency: W + R > N
# Example: N=3, W=2, R=2 → guaranteed to read latest write

class QuorumConfig:
    def __init__(self, replicas: int, write_quorum: int, read_quorum: int):
        self.n = replicas
        self.w = write_quorum
        self.r = read_quorum

    @property
    def is_strongly_consistent(self) -> bool:
        return self.w + self.r > self.n

    @property
    def is_highly_available(self) -> bool:
        return self.w == 1  # Can write to any single node

# Configurations
strong = QuorumConfig(replicas=3, write_quorum=2, read_quorum=2)  # CP
available = QuorumConfig(replicas=3, write_quorum=1, read_quorum=1)  # AP
```

**Conflict Resolution for AP Systems:**
```python
# Last-write-wins (simple but lossy)
def resolve_lww(versions: list[VersionedValue]) -> VersionedValue:
    return max(versions, key=lambda v: v.timestamp)

# Application-level merge (preserves intent)
class ShoppingCart:
    def merge(self, other: "ShoppingCart") -> "ShoppingCart":
        # Union of items, max quantity for conflicts
        merged_items = {}
        for item in self.items + other.items:
            if item.product_id in merged_items:
                merged_items[item.product_id] = max(
                    merged_items[item.product_id], item.quantity
                )
            else:
                merged_items[item.product_id] = item.quantity
        return ShoppingCart(items=merged_items)
```

**Read-Your-Writes Consistency:**
```python
class SessionConsistency:
    """Client reads its own writes even in an AP system."""

    def __init__(self, primary: Database, replicas: list[Database]):
        self.primary = primary
        self.replicas = replicas

    def write(self, key: str, value: Any) -> WriteToken:
        result = self.primary.write(key, value)
        return WriteToken(version=result.version)

    def read(self, key: str, after: WriteToken | None = None) -> Any:
        if after and not self._replica_caught_up(after):
            return self.primary.read(key)  # Fall back to primary
        return self._random_replica().read(key)
```

### Common Mistakes
1. **"Pick 2 of 3" misunderstanding** — CAP doesn't mean you pick 2 properties globally; partition tolerance is mandatory in distributed systems
2. **Ignoring the normal case** — CAP only applies during partitions; PACELC addresses the common case (latency vs. consistency)
3. **Confusing consistency models** — CAP's "C" is linearizability, not "all nodes have same data eventually"
4. **No conflict resolution strategy** — Choosing AP without planning how to merge divergent state after partition heals
5. **Assuming single-node systems avoid CAP** — Single-node avoids distributed tradeoffs but sacrifices availability (single point of failure)

### Integration with Other Topics
- **Database Scaling Patterns** — Replication strategies directly implement CAP tradeoffs
- **NoSQL and Polyglot Persistence** — Different databases sit at different points on the CAP spectrum
- **Microservices** — Each service makes independent consistency choices
- **Event Sourcing** — Event log enables eventual consistency with auditability
- **Caching Strategies** — Caches introduce staleness, a form of AP tradeoff
- **Resilience Patterns** — Circuit breakers and fallbacks operate during partition scenarios

---

## Resources

### Essential Reading
- *Designing Data-Intensive Applications* — Martin Kleppmann (chapters 5, 7, 9)
- Eric Brewer — "CAP Twelve Years Later: How the Rules Have Changed" (2012)
- Daniel Abadi — "Consistency Tradeoffs in Modern Distributed Database System Design"

### Online Resources
- Martin Kleppmann's "Please stop calling databases CP or AP" article
- Jepsen.io — Kyle Kingsbury's distributed systems correctness testing
- AWS Architecture Blog on consistency models

### Practice Exercises
1. Classify 5 common databases on the CAP/PACELC spectrum
2. Design a system where payments use CP and product catalog uses AP
3. Implement a conflict resolution strategy for a shopping cart
4. Set up a quorum-based write with N=5, W=3, R=3 and verify consistency
5. Simulate a network partition and observe behavior in a CP vs. AP system
