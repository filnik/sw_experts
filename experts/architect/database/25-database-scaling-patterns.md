# Database Scaling Patterns

## Profile

### What It Is
Database Scaling Patterns are strategies for handling growing data volumes and query loads beyond what a single database server can handle. These include vertical scaling, read replicas, sharding (horizontal partitioning), partitioning, connection pooling, and federated databases.

### Origin and Evolution
- Early databases scaled vertically (bigger hardware)
- Read replicas emerged for read-heavy workloads (MySQL replication, PostgreSQL streaming)
- Sharding pioneered by web-scale companies (Google, Facebook, Twitter)
- Managed services simplified scaling (Aurora, Cloud Spanner, PlanetScale)
- Modern: auto-sharding databases (CockroachDB, YugabyteDB, Vitess)

### Key Authors and References
- **Martin Kleppmann** — *Designing Data-Intensive Applications* (replication, partitioning)
- **Baron Schwartz** — *High Performance MySQL*
- **Sugu Sougoumarane** — Vitess (YouTube's MySQL scaling)
- **Google** — Spanner paper (globally distributed, strongly consistent)

### Key Concepts at a Glance
| Pattern | How It Works | Scales |
|---------|-------------|--------|
| Vertical Scaling | Bigger machine (CPU, RAM, SSD) | Up to hardware limits |
| Read Replicas | Copy data to read-only replicas | Read throughput |
| Sharding | Partition data across multiple servers | Read + write throughput |
| Table Partitioning | Split large tables within one server | Query performance |
| Connection Pooling | Reuse database connections | Connection capacity |
| Caching Layer | Serve frequent reads from cache | Read throughput |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Scale reads before writes** — Most applications are read-heavy. Read replicas and caching solve 80% of scaling problems without the complexity of sharding.
2. **Shard as a last resort** — Sharding introduces massive complexity (cross-shard queries, rebalancing, operational overhead). Exhaust other options first.
3. **Choose the shard key carefully** — A bad shard key creates hot spots, makes cross-shard queries necessary, and is very expensive to change later.
4. **Connection pooling is always needed** — Database connections are expensive. Connection pooling should be the first optimization, not an afterthought.
5. **Measure before scaling** — Profile queries, identify bottlenecks, optimize indexes and queries before adding infrastructure.

### When to Use vs. When to Avoid

**Read Replicas when:** Read/write ratio > 10:1, read latency matters, reporting offloading needed
**Sharding when:** Single-node write capacity is exceeded, dataset exceeds single-node storage, geographical distribution required
**Table Partitioning when:** Large tables (>100M rows), time-series data, archival requirements
**Always:** Connection pooling, query optimization, proper indexing

---

## Frameworks and Methodologies

### 1. Scaling Assessment — step-by-step
1. Measure current performance: QPS, latency p50/p99, CPU, IOPS, connections
2. Identify the bottleneck: CPU? I/O? Connections? Lock contention?
3. Optimize first: indexes, query rewrites, connection pooling, caching
4. If reads are the bottleneck: add read replicas
5. If writes are the bottleneck: consider sharding or database change
6. If data volume is the issue: partitioning or archival strategy

### 2. Sharding Strategy Design — step-by-step
1. Identify the shard key (high cardinality, evenly distributed, aligned with queries)
2. Choose sharding strategy: hash-based, range-based, or directory-based
3. Plan for cross-shard queries (minimize them by co-locating related data)
4. Implement routing layer (application-level or proxy like Vitess)
5. Plan rebalancing strategy for when shards become uneven
6. Test failover and recovery for individual shards

---

## How to Apply

### Decision Checklist
- [ ] Have I optimized queries and indexes before scaling infrastructure?
- [ ] Is connection pooling configured?
- [ ] Is the workload read-heavy (replicas) or write-heavy (sharding)?
- [ ] If sharding: is the shard key high-cardinality and query-aligned?
- [ ] Are cross-shard operations minimized?
- [ ] Is monitoring in place for each shard/replica?

### Implementation Patterns

**Connection Pooling (PgBouncer):**
```ini
# pgbouncer.ini
[databases]
myapp = host=postgres port=5432 dbname=myapp

[pgbouncer]
pool_mode = transaction       # Best for most apps
max_client_conn = 1000        # Accept 1000 client connections
default_pool_size = 20        # But only 20 actual DB connections
min_pool_size = 5
reserve_pool_size = 5
```

**Read Replica Routing:**
```python
class DatabaseRouter:
    def __init__(self, primary: Database, replicas: list[Database]):
        self.primary = primary
        self.replicas = replicas
        self._replica_index = 0

    def get_connection(self, read_only: bool = False) -> Database:
        if read_only and self.replicas:
            # Round-robin across replicas
            replica = self.replicas[self._replica_index % len(self.replicas)]
            self._replica_index += 1
            return replica
        return self.primary

# Usage
class OrderRepository:
    def __init__(self, router: DatabaseRouter):
        self.router = router

    def find_by_id(self, order_id: str) -> Order:
        db = self.router.get_connection(read_only=True)
        return db.query("SELECT * FROM orders WHERE id = %s", order_id)

    def save(self, order: Order) -> None:
        db = self.router.get_connection(read_only=False)  # Uses primary
        db.execute("INSERT INTO orders ...", order.to_dict())
```

**Hash-Based Sharding:**
```python
class ShardRouter:
    def __init__(self, shards: list[Database]):
        self.shards = shards
        self.num_shards = len(shards)

    def get_shard(self, shard_key: str) -> Database:
        shard_index = hash(shard_key) % self.num_shards
        return self.shards[shard_index]

# Usage — shard by customer_id
class OrderRepository:
    def __init__(self, router: ShardRouter):
        self.router = router

    def save(self, order: Order) -> None:
        shard = self.router.get_shard(order.customer_id)
        shard.execute("INSERT INTO orders ...", order.to_dict())

    def find_by_customer(self, customer_id: str) -> list[Order]:
        shard = self.router.get_shard(customer_id)
        return shard.query("SELECT * FROM orders WHERE customer_id = %s", customer_id)
```

**Table Partitioning (PostgreSQL):**
```sql
-- Range partitioning by date
CREATE TABLE events (
    id UUID,
    event_type VARCHAR(50),
    payload JSONB,
    created_at TIMESTAMPTZ NOT NULL
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2024_q1 PARTITION OF events
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
CREATE TABLE events_2024_q2 PARTITION OF events
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- Queries with created_at filter automatically use the right partition
-- Old partitions can be detached and archived
```

### Common Mistakes
1. **Premature sharding** — Adding sharding complexity before optimizing queries, indexes, and caching
2. **Bad shard key** — Low cardinality (e.g., country code) creates hot shards; changing the key later is extremely painful
3. **Cross-shard joins** — Requiring JOINs across shards defeats the purpose; co-locate related data
4. **No connection pooling** — Each application process opening its own connections exhausts the database
5. **Replica lag ignored** — Read replicas lag behind the primary; reading stale data after a write causes bugs
6. **Over-partitioning** — Creating too many small partitions increases planning time and overhead

### Integration with Other Topics
- **Relational Database Design** — Schema design affects scalability
- **Caching Strategies** — Caching is the first line of scaling defense
- **CAP Theorem** — Replication involves consistency tradeoffs
- **Microservices** — Database-per-service naturally distributes load
- **System Design Fundamentals** — Database is typically the first bottleneck in system design
- **Data Modeling** — Shard key selection is a data modeling decision

---

## Resources

### Essential Reading
- *Designing Data-Intensive Applications* — Martin Kleppmann (chapters 5-6)
- *High Performance MySQL* — Baron Schwartz et al.
- Vitess documentation (vitess.io)

### Online Resources
- PlanetScale engineering blog on sharding
- AWS Aurora scaling documentation
- Citus (distributed PostgreSQL) documentation

### Practice Exercises
1. Set up PostgreSQL streaming replication with a read replica
2. Implement application-level read/write routing
3. Design a sharding strategy for a multi-tenant SaaS
4. Partition a time-series table and benchmark query performance
5. Configure PgBouncer and measure connection capacity improvement
