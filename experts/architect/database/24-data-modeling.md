# Data Modeling

## Profile

### What It Is
Data Modeling is the process of creating a visual representation of data structures, relationships, and constraints that define how data is stored, organized, and accessed. It bridges the gap between business requirements and database implementation through conceptual, logical, and physical models.

### Origin and Evolution
- Entity-Relationship (ER) model introduced by Peter Chen (1976)
- Evolved through conceptual → logical → physical modeling stages
- UML class diagrams adopted for object-oriented data modeling
- Modern: event-driven data modeling, NoSQL-specific modeling, data mesh
- Current trend: schema-as-code, versioned data models, collaborative modeling tools

### Key Authors and References
- **Peter Chen** — ER model (1976)
- **C.J. Date** — Relational theory and data modeling
- **Len Silverston** — *The Data Model Resource Book* (universal patterns)
- **Martin Kleppmann** — Data models in *Designing Data-Intensive Applications*

### Key Concepts at a Glance
| Level | Focus | Output |
|-------|-------|--------|
| Conceptual | Business entities and relationships | ER diagram, domain model |
| Logical | Attributes, types, keys, normalization | Detailed schema with no tech specifics |
| Physical | Tables, indexes, partitions, engine-specific | DDL, migration scripts |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Start with the domain, not the database** — Understand business entities and their relationships before thinking about tables and columns.
2. **Model at three levels** — Conceptual (what), logical (how, abstractly), physical (how, concretely). Each level serves different stakeholders.
3. **Access patterns drive physical design** — Conceptual and logical models reflect the domain; physical models optimize for how data will be queried.
4. **Naming is critical** — Consistent, clear naming conventions make schemas self-documenting. A well-named column eliminates the need for comments.
5. **Models evolve** — Data models are living artifacts. Plan for evolution through migrations, versioning, and backward compatibility.

### When to Use vs. When to Avoid

**Invest in formal data modeling when:**
- The domain is complex with many entities and relationships
- Multiple teams or systems share the data
- Data integrity and consistency are critical
- The schema will be long-lived (years)

**Lightweight modeling suffices when:**
- Small team, simple domain
- Rapid prototyping where the model will change drastically
- Event-sourced systems (events are the model)
- Document databases with embedded data

---

## Frameworks and Methodologies

### 1. ER Modeling — step-by-step
1. Identify entities (nouns from requirements: Customer, Order, Product)
2. Define relationships between entities (Customer places Order)
3. Determine cardinality (one-to-many, many-to-many)
4. Add attributes to each entity
5. Identify primary keys and candidate keys
6. Normalize to 3NF
7. Review with domain experts using the conceptual diagram

### 2. Query-Driven Modeling (NoSQL) — step-by-step
1. List all queries the application will execute
2. For each query, define the ideal response shape
3. Design collections/tables that serve each query directly
4. Accept denormalization to avoid cross-collection lookups
5. Plan for data duplication and update propagation
6. Design partition/sort keys for efficient access

---

## How to Apply

### Decision Checklist
- [ ] Is there a conceptual model that stakeholders understand?
- [ ] Does the logical model capture all entities, relationships, and constraints?
- [ ] Is the physical model optimized for known access patterns?
- [ ] Are naming conventions consistent across the schema?
- [ ] Is there a migration strategy for schema evolution?

### Implementation Patterns

**Conceptual Model (ER Diagram):**
```
[Customer] 1──────* [Order] 1──────* [OrderItem]
    │                  │                   │
    │                  │                   │
    │                  *                   *
    │            [Payment]           [Product]
    │
    *
[Address]
```

**Logical to Physical Translation:**
```sql
-- Logical: Customer has many Addresses (1:N)
-- Physical: addresses table with customer_id FK
CREATE TABLE addresses (
    id UUID PRIMARY KEY,
    customer_id UUID NOT NULL REFERENCES customers(id),
    type VARCHAR(20) NOT NULL CHECK (type IN ('billing', 'shipping')),
    street VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    country CHAR(2) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,
    is_default BOOLEAN NOT NULL DEFAULT false,
    UNIQUE (customer_id, type, is_default) -- Only one default per type
);

-- Logical: Order has many Products (M:N via OrderItem)
-- Physical: junction table with extra attributes
CREATE TABLE order_items (
    order_id UUID REFERENCES orders(id),
    product_id UUID REFERENCES products(id),
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10,2) NOT NULL, -- Price at time of order (snapshot)
    PRIMARY KEY (order_id, product_id)
);
```

**Common Data Model Patterns:**
```
-- Polymorphic association (Type/Subtype)
-- Option 1: Single Table Inheritance
CREATE TABLE notifications (
    id UUID PRIMARY KEY,
    type VARCHAR(20) NOT NULL, -- 'email', 'sms', 'push'
    recipient VARCHAR(255) NOT NULL,
    subject VARCHAR(255),      -- email only
    phone_number VARCHAR(20),  -- sms only
    device_token VARCHAR(255), -- push only
    body TEXT NOT NULL
);

-- Option 2: Class Table Inheritance
CREATE TABLE notifications (
    id UUID PRIMARY KEY,
    type VARCHAR(20) NOT NULL,
    body TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL
);
CREATE TABLE email_notifications (
    notification_id UUID PRIMARY KEY REFERENCES notifications(id),
    recipient_email VARCHAR(255) NOT NULL,
    subject VARCHAR(255) NOT NULL
);
CREATE TABLE sms_notifications (
    notification_id UUID PRIMARY KEY REFERENCES notifications(id),
    phone_number VARCHAR(20) NOT NULL
);
```

### Common Mistakes
1. **Skipping the conceptual model** — Going straight to tables without understanding the domain leads to fragile schemas
2. **Modeling for every possible query** — Designing for hypothetical access patterns instead of known ones
3. **Inconsistent naming** — Mixing `user_id`, `userId`, `UserID`, `fk_user` in the same schema
4. **Snapshot vs. reference confusion** — Referencing product price by FK when you need the price at order time (should be copied)
5. **Ignoring temporal aspects** — Not modeling "as of" dates for slowly changing dimensions
6. **Over-normalizing** — 6NF for all tables when 3NF with strategic denormalization is more practical

### Integration with Other Topics
- **Domain-Driven Design** — Aggregates and entities map to data model structures
- **Relational Database Design** — Physical implementation of data models
- **NoSQL** — Query-driven modeling for non-relational stores
- **Database Scaling** — Partition key design is a data modeling concern
- **Event Sourcing** — Events replace traditional CRUD data models
- **API Design** — API resources often mirror (but don't equal) data model entities

---

## Resources

### Essential Reading
- *The Data Model Resource Book* (Vol. 1-3) — Len Silverston
- *Designing Data-Intensive Applications* — Martin Kleppmann
- *Database Design for Mere Mortals* — Michael Hernandez

### Online Resources
- dbdiagram.io — Free online data modeling tool
- Erwin Data Modeler documentation
- Martin Fowler's analysis patterns

### Practice Exercises
1. Create a conceptual ER model for a hotel booking system
2. Translate a conceptual model to a normalized logical model (3NF)
3. Design the same domain for both PostgreSQL and MongoDB
4. Model a slowly changing dimension (customer address history)
5. Design a data model for a multi-tenant SaaS application
