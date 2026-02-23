# Database Transactions and Isolation

## Profile

### What It Is
Database transactions are logical units of work that group multiple operations into an atomic, consistent, isolated, and durable (ACID) sequence. Isolation levels define how transactions interact with each other — balancing correctness against performance by controlling which anomalies (dirty reads, phantom reads, write skew) are permitted.

### Origin and Evolution
- ACID properties formalized by Jim Gray and Andreas Reuter (1983)
- SQL standard defined four isolation levels (SQL-92)
- MVCC (Multi-Version Concurrency Control) became dominant in modern databases (PostgreSQL, Oracle, MySQL InnoDB)
- Distributed transactions: Two-Phase Commit (2PC), Saga pattern
- Modern: serializable snapshot isolation (PostgreSQL), deterministic databases

### Key Authors and References
- **Jim Gray** — Transaction processing pioneer, *Transaction Processing* (1993)
- **Martin Kleppmann** — *Designing Data-Intensive Applications* (chapter 7)
- **Peter Bailis** — Research on isolation levels and their anomalies
- **Dan Abadi** — Deterministic database systems

### Key Concepts at a Glance
| Anomaly | Description | Prevented By |
|---------|-------------|-------------|
| Dirty Read | Reading uncommitted data from another transaction | Read Committed+ |
| Non-Repeatable Read | Same query returns different results within one transaction | Repeatable Read+ |
| Phantom Read | New rows appear in a repeated range query | Serializable |
| Write Skew | Two transactions read same data, make different writes | Serializable |
| Lost Update | Two transactions overwrite each other's changes | Repeatable Read+ (with locks) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Understand your isolation level** — Most databases default to Read Committed. Know what anomalies are possible at your level and whether they matter for your use case.
2. **ACID is not free** — Stronger isolation means more locking, less concurrency, and lower throughput. Choose the weakest isolation that ensures correctness.
3. **Keep transactions short** — Long-running transactions hold locks, block other transactions, and increase the chance of deadlocks and conflicts.
4. **Optimistic concurrency for reads** — Use MVCC-based isolation (snapshot) rather than locking for read-heavy workloads.
5. **Explicit locking when needed** — When isolation levels aren't sufficient (write skew), use explicit locks (SELECT FOR UPDATE) rather than blindly upgrading to Serializable.

### When to Use vs. When to Avoid

**Strong isolation (Serializable) when:**
- Financial transactions, inventory management
- Write skew could cause incorrect results
- Correctness is more important than throughput

**Weaker isolation (Read Committed) when:**
- Read-heavy workloads with rare conflicts
- Performance is critical and anomalies are tolerable
- Conflicts can be detected and retried at the application level

---

## Frameworks and Methodologies

### 1. Isolation Level Selection — step-by-step
1. Identify the critical data operations in the application
2. For each operation, determine which anomalies would cause bugs
3. Map required protections to the minimum isolation level
4. Set database-wide default to Read Committed (most common)
5. Use stronger isolation for specific transactions that need it
6. Implement application-level retry logic for serialization failures

### 2. Optimistic Concurrency Control — step-by-step
1. Add a version column (integer or timestamp) to each table
2. Read the current version with the data
3. On update, include `WHERE version = :expected_version`
4. If zero rows updated, the data was modified by another transaction
5. Retry the entire read-compute-write cycle
6. Alert on excessive retries (indicates contention)

---

## How to Apply

### Decision Checklist
- [ ] What isolation level is the database using by default?
- [ ] Which operations require stronger isolation?
- [ ] Are transactions kept as short as possible?
- [ ] Is there retry logic for serialization/conflict failures?
- [ ] Is optimistic concurrency preferred over pessimistic locking?

### Implementation Patterns

**Isolation Levels in Practice:**
```sql
-- Read Committed (default in PostgreSQL)
-- Sees only committed data; each statement sees latest committed state
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Sees committed data
-- Another transaction could commit a change here
SELECT balance FROM accounts WHERE id = 1;  -- May see different value!
COMMIT;

-- Repeatable Read (snapshot isolation in PostgreSQL)
-- Transaction sees a consistent snapshot from its start
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- snapshot at tx start
-- Other commits don't affect this transaction's reads
SELECT balance FROM accounts WHERE id = 1;  -- Same value guaranteed
COMMIT;

-- Serializable
-- Prevents all anomalies; transactions behave as if run sequentially
BEGIN ISOLATION LEVEL SERIALIZABLE;
-- If conflict detected, one transaction aborts with serialization_failure
-- Application MUST retry
COMMIT;
```

**Optimistic Concurrency Control:**
```python
class AccountRepository:
    def transfer(self, from_id: str, to_id: str, amount: Decimal) -> None:
        for attempt in range(3):
            try:
                with self.db.transaction():
                    from_acc = self.db.query_one(
                        "SELECT id, balance, version FROM accounts WHERE id = %s",
                        from_id
                    )
                    if from_acc.balance < amount:
                        raise InsufficientFunds()

                    rows = self.db.execute(
                        "UPDATE accounts SET balance = balance - %s, version = version + 1 "
                        "WHERE id = %s AND version = %s",
                        amount, from_id, from_acc.version
                    )
                    if rows == 0:
                        raise OptimisticLockError()

                    self.db.execute(
                        "UPDATE accounts SET balance = balance + %s, version = version + 1 "
                        "WHERE id = %s",
                        amount, to_id
                    )
                    return
            except OptimisticLockError:
                if attempt == 2:
                    raise
                continue
```

**Pessimistic Locking (SELECT FOR UPDATE):**
```python
def reserve_inventory(self, product_id: str, quantity: int) -> None:
    with self.db.transaction():
        # Lock the row — other transactions wait
        stock = self.db.query_one(
            "SELECT * FROM inventory WHERE product_id = %s FOR UPDATE",
            product_id
        )
        if stock.available < quantity:
            raise InsufficientStock()
        self.db.execute(
            "UPDATE inventory SET available = available - %s WHERE product_id = %s",
            quantity, product_id
        )
```

**Write Skew Prevention:**
```sql
-- Write skew example: two doctors try to go off-call simultaneously
-- Both read "2 doctors on call", both go off-call, leaving 0

-- Prevention with Serializable:
BEGIN ISOLATION LEVEL SERIALIZABLE;
SELECT count(*) FROM doctors WHERE on_call = true AND shift_id = 123;
-- If count >= 2, allow going off-call
UPDATE doctors SET on_call = false WHERE id = 456 AND shift_id = 123;
COMMIT;
-- If write skew detected, one transaction gets serialization_failure
```

### Common Mistakes
1. **Assuming Serializable by default** — Most databases default to Read Committed; assuming stronger isolation leads to bugs
2. **Long transactions** — Holding a transaction open during user input or external API calls blocks others
3. **No retry on serialization failure** — Serializable isolation requires retry logic; without it, operations fail permanently
4. **Pessimistic locking everywhere** — SELECT FOR UPDATE on every read destroys concurrency; use only where needed
5. **Forgetting about deadlocks** — Acquiring locks in inconsistent order across transactions causes deadlocks
6. **Application-level "transactions"** — Doing read-modify-write without database transaction guarantees

### Integration with Other Topics
- **Relational Database Design** — Constraints and transactions enforce data integrity
- **Database Scaling** — Replication introduces consistency challenges
- **CAP Theorem** — Distributed transactions face partition/consistency tradeoffs
- **Event Sourcing** — Events replace mutable state; optimistic concurrency on event streams
- **Saga Pattern** — Replaces distributed transactions with compensating actions
- **Concurrency and Parallelism** — Transaction isolation is the database's concurrency control

---

## Resources

### Essential Reading
- *Designing Data-Intensive Applications* — Martin Kleppmann (chapter 7)
- *Transaction Processing: Concepts and Techniques* — Jim Gray & Andreas Reuter
- PostgreSQL documentation on Transaction Isolation

### Online Resources
- Jepsen.io — Consistency and isolation testing for databases
- Peter Bailis — "Silence is Golden: Coordination-Avoiding Systems Design"
- PostgreSQL wiki on SSI (Serializable Snapshot Isolation)

### Practice Exercises
1. Demonstrate a dirty read, non-repeatable read, and phantom read
2. Implement optimistic concurrency control with version columns
3. Show how write skew occurs at Read Committed and is prevented at Serializable
4. Benchmark throughput at different isolation levels
5. Implement retry logic for serialization failures
