# TypeScript ORM Patterns

## Profile

### What It Is
TypeScript ORM patterns cover type-safe database access using Prisma, Drizzle ORM, and TypeORM. Modern TypeScript ORMs generate types from the database schema, providing compile-time safety for queries, migrations, and data access patterns.

### Origin and Evolution
- TypeORM (2016) — Decorator-based ORM (inspired by Java ORMs)
- Sequelize + types — Added TypeScript support retroactively
- Prisma (2019) — Schema-first ORM with generated types
- Drizzle (2022) — SQL-like query builder with full type safety
- Kysely (2022) — Type-safe SQL query builder
- Current: Prisma for ease of use; Drizzle for SQL-like control; Kysely for raw SQL fans

### Key Authors and References
- **Prisma team** — Prisma ORM documentation
- **Drizzle team** — Drizzle ORM documentation
- **Alex Shelkovskiy** — Drizzle ORM creator

### Key Concepts at a Glance
| ORM | Approach | Strengths |
|-----|----------|-----------|
| Prisma | Schema-first, generated client | DX, migrations, easy to learn |
| Drizzle | TypeScript schema, SQL-like syntax | Control, performance, SQL familiarity |
| TypeORM | Decorator-based, Active Record/Data Mapper | Feature-rich, familiar for Java devs |
| Kysely | Type-safe query builder (no ORM) | Raw SQL control, lightweight |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Types from schema** — Database schema generates TypeScript types. No manual type definitions that can drift from the actual database.
2. **Prisma for DX, Drizzle for control** — Prisma prioritizes developer experience (auto-complete, easy API). Drizzle gives SQL-like control with type safety.
3. **Migrations are mandatory** — Track all schema changes as versioned migrations. Never modify production schemas manually.
4. **Query results are typed** — `SELECT` results, including joins and partial selects, should be fully typed. No `any` in query results.
5. **N+1 prevention** — Include/eager loading (Prisma `include`), joins (Drizzle), or data loaders. Watch for lazy loading traps.

### When to Use vs. When to Avoid

**Prisma**: Rapid development, prototyping, teams new to type-safe ORMs
**Drizzle**: Teams that want SQL control with type safety, performance-sensitive
**TypeORM**: Existing TypeORM projects, decorator-based preference
**Kysely**: Raw SQL preferred, minimal abstraction wanted

---

## Frameworks and Methodologies

### 1. Prisma Setup — step-by-step
1. Install: `npm install prisma @prisma/client`
2. Init: `npx prisma init`
3. Define schema in `prisma/schema.prisma`
4. Generate client: `npx prisma generate`
5. Create migration: `npx prisma migrate dev`
6. Use `PrismaClient` with full type safety

### 2. Drizzle Setup — step-by-step
1. Install: `drizzle-orm` + database driver
2. Define schema in TypeScript (`schema.ts`)
3. Set up Drizzle Kit for migrations
4. Generate migration: `npx drizzle-kit generate`
5. Apply: `npx drizzle-kit migrate`
6. Query with SQL-like syntax, fully typed

---

## How to Apply

### Decision Checklist
- [ ] Are database types generated from schema (not handwritten)?
- [ ] Are migrations tracked and versioned?
- [ ] Are query results fully typed (including joins, partial selects)?
- [ ] Is N+1 loading addressed (include, join, or data loader)?
- [ ] Is the ORM appropriate for the project's SQL complexity?

### Implementation Patterns

**Prisma:**
```prisma
// prisma/schema.prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  orders    Order[]
  createdAt DateTime @default(now())
}

model Order {
  id         String      @id @default(uuid())
  userId     String
  user       User        @relation(fields: [userId], references: [id])
  items      OrderItem[]
  status     OrderStatus @default(DRAFT)
  total      Decimal
  createdAt  DateTime    @default(now())
}

model OrderItem {
  id        Int    @id @default(autoincrement())
  orderId   String
  order     Order  @relation(fields: [orderId], references: [id])
  productId String
  quantity  Int
  unitPrice Decimal
}

enum OrderStatus {
  DRAFT
  CONFIRMED
  SHIPPED
  DELIVERED
}
```

```typescript
// Prisma usage — fully typed
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

// Create with relations
const order = await prisma.order.create({
  data: {
    userId: "user-123",
    total: 99.99,
    items: {
      create: [
        { productId: "p1", quantity: 2, unitPrice: 29.99 },
        { productId: "p2", quantity: 1, unitPrice: 40.01 },
      ],
    },
  },
  include: { items: true, user: true },
});
// order.items is OrderItem[], order.user is User — typed

// Query with filters
const orders = await prisma.order.findMany({
  where: {
    userId: "user-123",
    status: "CONFIRMED",
    createdAt: { gte: new Date("2024-01-01") },
  },
  include: { items: true },
  orderBy: { createdAt: "desc" },
  take: 20,
});

// Partial select (only requested fields)
const userEmails = await prisma.user.findMany({
  select: { email: true, name: true },
});
// Type: { email: string; name: string }[] — no extra fields
```

**Drizzle ORM:**
```typescript
// schema.ts — define schema in TypeScript
import { pgTable, text, integer, decimal, timestamp, pgEnum } from "drizzle-orm/pg-core";

export const orderStatusEnum = pgEnum("order_status", ["draft", "confirmed", "shipped", "delivered"]);

export const users = pgTable("users", {
  id: text("id").primaryKey().defaultRandom(),
  email: text("email").notNull().unique(),
  name: text("name").notNull(),
  createdAt: timestamp("created_at").defaultNow(),
});

export const orders = pgTable("orders", {
  id: text("id").primaryKey().defaultRandom(),
  userId: text("user_id").notNull().references(() => users.id),
  status: orderStatusEnum("status").default("draft"),
  total: decimal("total", { precision: 10, scale: 2 }).notNull(),
  createdAt: timestamp("created_at").defaultNow(),
});

export const orderItems = pgTable("order_items", {
  id: integer("id").primaryKey().generatedAlwaysAsIdentity(),
  orderId: text("order_id").notNull().references(() => orders.id),
  productId: text("product_id").notNull(),
  quantity: integer("quantity").notNull(),
  unitPrice: decimal("unit_price", { precision: 10, scale: 2 }).notNull(),
});

// Queries — SQL-like, fully typed
import { eq, desc, gte, and } from "drizzle-orm";

// Select with join
const result = await db
  .select({
    orderId: orders.id,
    total: orders.total,
    userName: users.name,
  })
  .from(orders)
  .innerJoin(users, eq(orders.userId, users.id))
  .where(and(
    eq(orders.userId, "user-123"),
    eq(orders.status, "confirmed"),
    gte(orders.createdAt, new Date("2024-01-01")),
  ))
  .orderBy(desc(orders.createdAt))
  .limit(20);
// Type: { orderId: string; total: string; userName: string }[]

// Insert
await db.insert(orders).values({
  userId: "user-123",
  total: "99.99",
  status: "draft",
});
```

**Repository Pattern:**
```typescript
// With Prisma
class PrismaOrderRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string) {
    return this.prisma.order.findUnique({
      where: { id },
      include: { items: true },
    });
  }

  async findByUser(userId: string, filters?: OrderFilters) {
    return this.prisma.order.findMany({
      where: {
        userId,
        status: filters?.status,
        createdAt: filters?.since ? { gte: filters.since } : undefined,
      },
      include: { items: true },
      orderBy: { createdAt: "desc" },
    });
  }

  async create(data: CreateOrderInput) {
    return this.prisma.order.create({
      data: {
        userId: data.userId,
        total: data.total,
        items: { create: data.items },
      },
      include: { items: true },
    });
  }
}
```

### Common Mistakes
1. **N+1 queries** — Accessing relations without `include` (Prisma) or `join` (Drizzle)
2. **No migrations** — Schema changes applied directly to database
3. **Leaking ORM types** — Exposing Prisma types to API layer; use DTOs at boundaries
4. **Raw SQL for everything** — Using `$queryRaw` when the ORM handles it type-safely
5. **Not using transactions** — Multiple writes without `$transaction` (Prisma) or `db.transaction` (Drizzle)

### Integration with Other Topics
- **Node.js + TypeScript** — ORM used in Express/Fastify/tRPC backends
- **Clean Architecture** — Repository pattern wraps ORM; domain models separate
- **Testing** — Test with in-memory database or test containers
- **Database Design** — Schema reflects relational design principles
- **TypeScript Type System** — ORM generates types from schema

---

## Resources

### Essential Reading
- Prisma documentation (prisma.io/docs)
- Drizzle ORM documentation (orm.drizzle.team)
- Kysely documentation (kysely.dev)

### Online Resources
- Prisma schema reference
- Drizzle Kit migration guide
- TypeORM documentation (typeorm.io)

### Practice Exercises
1. Define a Prisma schema with 3 models and relations, generate client
2. Set up Drizzle with TypeScript schema and migrations
3. Implement a Repository pattern with Prisma
4. Compare Prisma vs Drizzle for a complex join query
5. Set up transactions for multi-table write operations
