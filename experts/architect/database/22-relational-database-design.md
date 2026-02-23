# Relational Database Design

## Profile

### What It Is
Relational Database Design is the discipline of structuring data in relational databases (RDBMS) using tables, columns, relationships, and constraints to ensure data integrity, minimize redundancy, and support efficient querying. It encompasses normalization theory, schema design, indexing strategy, and constraint management.

### Origin and Evolution
- Founded by E.F. Codd's relational model (1970)
- Normalization theory developed through 1970s-80s (1NF through 5NF, BCNF)
- SQL standardized (SQL-86, SQL-92, SQL:2023)
- Modern RDBMS: PostgreSQL, MySQL, SQL Server, Oracle
- Current: relational databases remain the default for structured, transactional data

### Key Authors and References
- **E.F. Codd** — Relational model (1970)
- **C.J. Date** — *An Introduction to Database Systems* (comprehensive reference)
- **Joe Celko** — *SQL for Smarties* series
- **Markus Winand** — *SQL Performance Explained*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Normalization | Reducing redundancy by decomposing tables (1NF-5NF) |
| Primary Key | Unique identifier for each row |
| Foreign Key | Reference to another table's primary key |
| Index | Data structure for fast lookups |
| Constraint | Rule enforcing data integrity (NOT NULL, UNIQUE, CHECK) |
| ACID | Atomicity, Consistency, Isolation, Durability |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Normalize first, denormalize with purpose** — Start with a normalized schema to eliminate redundancy. Denormalize only when query performance demands it, and document why.
2. **Enforce integrity at the database level** — Use constraints (FK, UNIQUE, CHECK, NOT NULL). Don't rely on application code alone to maintain data consistency.
3. **Design for queries, not just storage** — Understand access patterns before finalizing the schema. The best normalized schema is useless if queries are unacceptably slow.
4. **Indexes are not free** — Every index speeds reads but slows writes. Index for actual query patterns, not every possible column.
5. **Schema is a contract** — The schema documents the data model. Treat migrations as code: versioned, reviewed, and tested.

### When to Use vs. When to Avoid

**Use relational databases when:**
- Data is structured with clear relationships
- ACID transactions are required
- Complex queries (joins, aggregations) are frequent
- Data integrity is critical (finance, healthcare)

**Consider alternatives when:**
- Schema is highly dynamic or document-oriented
- Scale exceeds single-node capacity and partitioning is complex
- Access patterns are key-value or graph-based
- Write throughput requirements are extreme

---

## Frameworks and Methodologies

### 1. Schema Design Process — step-by-step
1. Identify entities from domain model (nouns become tables)
2. Define attributes for each entity (columns with types)
3. Identify relationships (one-to-one, one-to-many, many-to-many)
4. Apply normalization (at least 3NF for transactional data)
5. Define primary keys (prefer surrogate keys or UUIDs)
6. Add foreign keys and constraints
7. Design indexes based on known query patterns
8. Review with denormalization opportunities for read-heavy paths

### 2. Index Strategy — step-by-step
1. Identify the most frequent and critical queries
2. For each query, determine which columns are in WHERE, JOIN, ORDER BY
3. Create indexes for high-frequency query columns
4. Use composite indexes for multi-column filters (leftmost prefix rule)
5. Consider covering indexes for read-heavy queries
6. Monitor slow query log and add indexes iteratively
7. Remove unused indexes (they cost write performance)

---

## How to Apply

### Decision Checklist
- [ ] Is the schema normalized to at least 3NF?
- [ ] Are all relationships enforced with foreign keys?
- [ ] Are NOT NULL constraints applied where appropriate?
- [ ] Are indexes designed for actual query patterns?
- [ ] Is there a migration strategy (versioned, reversible)?

### Implementation Patterns

**Normalized Schema:**
```sql
-- Entities
CREATE TABLE customers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID NOT NULL REFERENCES customers(id),
    status VARCHAR(20) NOT NULL DEFAULT 'draft'
        CHECK (status IN ('draft', 'placed', 'shipped', 'delivered', 'cancelled')),
    total_amount DECIMAL(10, 2) NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID NOT NULL REFERENCES products(id),
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,
    UNIQUE (order_id, product_id)
);

-- Indexes for common queries
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_status ON orders(status) WHERE status != 'cancelled';
CREATE INDEX idx_order_items_order_id ON order_items(order_id);
```

**Denormalization for Performance:**
```sql
-- Materialized view for dashboard (denormalized)
CREATE MATERIALIZED VIEW customer_order_stats AS
SELECT
    c.id AS customer_id,
    c.name,
    COUNT(o.id) AS total_orders,
    SUM(o.total_amount) AS lifetime_value,
    MAX(o.created_at) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;

CREATE UNIQUE INDEX idx_cos_customer ON customer_order_stats(customer_id);

-- Refresh periodically
REFRESH MATERIALIZED VIEW CONCURRENTLY customer_order_stats;
```

### Common Mistakes
1. **No foreign keys** — "We'll enforce it in the app" leads to orphaned records
2. **Over-indexing** — Indexes on every column slow writes and waste storage
3. **VARCHAR(255) everywhere** — Use appropriate types; don't default to oversized strings
4. **Missing NOT NULL** — Columns that should never be null aren't constrained
5. **Natural keys as primary keys** — Email, SSN change; prefer surrogate keys (UUID, serial)
6. **No migration tooling** — Schema changes applied manually lead to drift between environments

### Integration with Other Topics
- **Data Modeling** — ER modeling informs relational schema design
- **Database Transactions** — ACID properties govern relational database behavior
- **NoSQL** — Understanding when relational isn't the right fit
- **ORM Patterns** — ORMs map relational schemas to application objects
- **Database Scaling** — Partitioning, replication strategies for relational databases
- **Clean Architecture** — Database schema is an implementation detail, not a domain model

---

## Resources

### Essential Reading
- *SQL Performance Explained* — Markus Winand
- *An Introduction to Database Systems* — C.J. Date
- *Database Design for Mere Mortals* — Michael Hernandez

### Online Resources
- use-the-index-luke.com — Markus Winand's indexing guide
- PostgreSQL documentation (particularly on indexing and constraints)
- SQLZoo and LeetCode SQL exercises

### Practice Exercises
1. Design a normalized schema for an e-commerce system
2. Analyze a slow query and add appropriate indexes
3. Implement a migration that adds a column with a NOT NULL constraint to a large table
4. Convert a denormalized table into 3NF and verify with queries
5. Design a composite index strategy for a multi-filter search query
