# NoSQL and Polyglot Persistence

## Profile

### What It Is
NoSQL databases are non-relational data stores designed for specific access patterns, high scalability, and flexible schemas. Polyglot Persistence is the practice of using different database technologies for different data storage needs within the same system — choosing the best tool for each use case rather than forcing all data into one technology.

### Origin and Evolution
- NoSQL movement emerged ~2009 driven by web-scale companies (Google BigTable, Amazon Dynamo)
- Four main categories: document, key-value, column-family, graph
- Martin Fowler and Pramod Sadalage coined "Polyglot Persistence" (2011)
- Modern convergence: multi-model databases (Cosmos DB, ArangoDB), NewSQL (CockroachDB, TiDB)
- Current trend: purpose-built databases for specific workloads

### Key Authors and References
- **Martin Fowler & Pramod Sadalage** — *NoSQL Distilled* (2012)
- **Amazon** — Dynamo paper (2007)
- **Google** — BigTable paper (2006), Spanner paper (2012)
- **Martin Kleppmann** — *Designing Data-Intensive Applications* (2017)

### Key Concepts at a Glance
| Type | Examples | Best For |
|------|----------|----------|
| Document | MongoDB, CouchDB | Flexible schemas, nested data |
| Key-Value | Redis, DynamoDB | Simple lookups, caching, sessions |
| Column-Family | Cassandra, HBase | Time-series, write-heavy, wide rows |
| Graph | Neo4j, Neptune | Relationships, traversals, recommendations |
| Search | Elasticsearch, Meilisearch | Full-text search, faceted queries |
| Time-Series | TimescaleDB, InfluxDB | Metrics, IoT, event data |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Match the database to the access pattern** — Don't force graph queries into a relational database or document data into rigid tables. Each database type excels at specific patterns.
2. **Schema flexibility has a cost** — Schema-less doesn't mean schema-free. The schema moves to the application, which must handle all historical formats.
3. **CAP tradeoffs are inherent** — NoSQL databases make explicit consistency/availability tradeoffs. Understand where your chosen database sits.
4. **Denormalization is intentional** — NoSQL databases optimize for reads by duplicating data. Embrace denormalization but plan for update propagation.
5. **Operational complexity scales with variety** — Each database technology requires expertise to operate. Polyglot persistence multiplies operational burden.

### When to Use vs. When to Avoid

**Use NoSQL when:**
- Data doesn't fit relational model naturally (documents, graphs, time-series)
- Scale requirements exceed single-node relational capacity
- Schema evolves frequently
- Specific access patterns match a NoSQL strength

**Use Polyglot Persistence when:**
- Different services have genuinely different data needs
- Team has operational capacity for multiple databases
- Performance requirements can't be met with a single technology

**Avoid when:**
- Relational database handles all needs adequately
- Team lacks expertise to operate multiple databases
- Data relationships are complex and cross database boundaries

---

## Frameworks and Methodologies

### 1. Database Selection — step-by-step
1. Catalog all data entities and their access patterns
2. Classify each: CRUD, search, graph traversal, time-series, key-value lookup
3. Identify consistency and durability requirements per entity
4. Map each classification to candidate database types
5. Evaluate operational cost and team expertise
6. Choose the minimum number of databases that cover all needs
7. Document the decision and tradeoffs (ADR)

### 2. Data Modeling for NoSQL — step-by-step
1. Start with the queries, not the entities (query-driven design)
2. Design tables/collections to serve each query directly
3. Denormalize to avoid joins (embed related data)
4. Accept data duplication; plan synchronization strategy
5. Choose partition key for even data distribution
6. Test with realistic data volumes

---

## How to Apply

### Decision Checklist
- [ ] Is the database choice driven by access patterns, not familiarity?
- [ ] Are consistency requirements understood for each data store?
- [ ] Is the team prepared to operate all chosen databases?
- [ ] Is data synchronization between stores planned?
- [ ] Have you considered a single database first (simpler)?

### Implementation Patterns

**Document Database (MongoDB):**
```python
# Embedded document — denormalized for read performance
order_document = {
    "_id": "order_123",
    "customer": {
        "id": "cust_456",
        "name": "John Doe",
        "email": "john@example.com"
    },
    "items": [
        {"product_id": "prod_1", "name": "Widget", "quantity": 2, "price": 29.99},
        {"product_id": "prod_2", "name": "Gadget", "quantity": 1, "price": 49.99},
    ],
    "total": 109.97,
    "status": "placed",
    "created_at": "2024-01-15T10:30:00Z"
}
# One query retrieves everything needed for order display
```

**Key-Value (Redis):**
```python
# Session storage
redis.setex(f"session:{session_id}", 3600, json.dumps(session_data))
session = json.loads(redis.get(f"session:{session_id}"))

# Leaderboard
redis.zadd("leaderboard", {"player_1": 1500, "player_2": 1200})
top_10 = redis.zrevrange("leaderboard", 0, 9, withscores=True)

# Rate limiting
key = f"rate:{user_id}:{minute}"
current = redis.incr(key)
if current == 1:
    redis.expire(key, 60)
if current > 100:
    raise RateLimitExceeded()
```

**Polyglot Persistence Architecture:**
```
                          ┌─────────────────┐
                          │   API Gateway    │
                          └────────┬────────┘
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                    ▼
    ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
    │  Order Service   │ │ Product Service │ │  Search Service  │
    │  (PostgreSQL)    │ │  (MongoDB)      │ │ (Elasticsearch)  │
    │  ACID, relations │ │  Flexible schema│ │  Full-text, facets│
    └─────────────────┘ └─────────────────┘ └─────────────────┘
              │                   │                    │
              └──── Events ───────┴──── Events ────────┘
                          (Kafka for sync)
```

### Common Mistakes
1. **Using MongoDB as a relational database** — Adding references everywhere and doing application-level joins
2. **Ignoring operational cost** — Each database requires monitoring, backups, upgrades, and expertise
3. **No data synchronization plan** — Polyglot persistence with data in multiple stores but no sync strategy
4. **Schema-less chaos** — Storing arbitrary shapes without any validation, making queries unreliable
5. **Choosing NoSQL for hype** — Using MongoDB when PostgreSQL would work perfectly with better tooling

### Integration with Other Topics
- **CAP Theorem** — NoSQL databases make explicit CAP tradeoffs
- **Microservices** — Each service can choose its own database (database-per-service)
- **Data Modeling** — NoSQL requires query-driven modeling, not entity-relationship modeling
- **Database Scaling** — NoSQL databases are often designed for horizontal scaling
- **Event-Driven Architecture** — Events synchronize data across polyglot stores
- **Search Engine Architecture** — Elasticsearch as part of a polyglot strategy

---

## Resources

### Essential Reading
- *NoSQL Distilled* — Fowler & Sadalage
- *Designing Data-Intensive Applications* — Martin Kleppmann
- *MongoDB: The Definitive Guide* — Shannon Bradshaw et al.

### Online Resources
- Martin Fowler's "Polyglot Persistence" article
- MongoDB University free courses
- Redis University free courses

### Practice Exercises
1. Model the same domain in relational, document, and graph databases
2. Design a polyglot architecture for an e-commerce platform
3. Implement data synchronization between PostgreSQL and Elasticsearch
4. Benchmark a read-heavy workload on MongoDB vs. PostgreSQL
5. Design a DynamoDB table with efficient partition key for a multi-tenant SaaS
