# ORM Patterns

## Profile

### What It Is
ORM (Object-Relational Mapping) patterns in Python cover SQLAlchemy (Core and ORM), Django ORM, and database migration tools (Alembic, Django migrations). ORMs map Python classes to database tables, enabling database operations through Python objects while managing schema evolution through migrations.

### Origin and Evolution
- SQLObject (2003) — early Python ORM
- SQLAlchemy (2006, Mike Bayer) — dual Core (SQL builder) + ORM layers
- Django ORM — tightly integrated with Django framework
- Alembic (2012) — Migration tool for SQLAlchemy
- SQLAlchemy 2.0 (2023) — Modern API with type hints, async support
- Current: SQLAlchemy 2.0 dominates non-Django; async support mature

### Key Authors and References
- **Mike Bayer** — SQLAlchemy creator
- **Martin Fowler** — Data Mapper vs. Active Record patterns
- **SQLAlchemy documentation** — comprehensive, essential reference
- **Django documentation** — ORM and migration guides

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Data Mapper | ORM maps objects to tables; objects don't know about DB (SQLAlchemy) |
| Active Record | Objects have DB methods (save, delete); Django ORM pattern |
| Unit of Work | Session tracks changes, flushes to DB in one transaction |
| Identity Map | Session returns same object for same row; prevents duplicates |
| Migration | Versioned schema changes (Alembic, Django migrations) |
| Query Builder | Composable, type-safe SQL construction (SQLAlchemy Core) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **SQLAlchemy Core vs. ORM** — Use Core (SQL expression language) for complex queries and bulk operations. Use ORM for CRUD and domain objects. They work together.
2. **Session management is critical** — SQLAlchemy Session is the Unit of Work. Scope it per-request (web) or per-operation (scripts). Don't leak sessions.
3. **Eager loading prevents N+1** — Lazy loading causes N+1 queries. Use `selectinload`, `joinedload`, or `subqueryload` to load relationships explicitly.
4. **Migrations are code** — Track schema changes as versioned migration scripts. Never modify production schemas manually.
5. **Don't hide the SQL** — ORMs abstract SQL, not replace it. Understand the SQL your ORM generates. Use `.statement` or logging to inspect queries.

### When to Use vs. When to Avoid

**Use SQLAlchemy when:**
- Building APIs with FastAPI, Flask, or standalone applications
- Need both ORM and raw SQL capabilities
- Complex queries, multi-database support
- Want fine-grained control over SQL generation

**Use Django ORM when:**
- Building with Django (it's tightly integrated)
- Need admin panel integration (Django admin reads ORM models)
- Simpler query patterns suffice

**Use raw SQL when:**
- Performance-critical bulk operations
- Complex analytics queries
- Database-specific features not supported by ORM

---

## Frameworks and Methodologies

### 1. SQLAlchemy Project Setup — step-by-step
1. Define models using `DeclarativeBase` (SQLAlchemy 2.0)
2. Configure engine and session factory
3. Set up Alembic for migrations: `alembic init`
4. Create initial migration: `alembic revision --autogenerate`
5. Apply: `alembic upgrade head`
6. Use async engine + async session for FastAPI

### 2. Migration Workflow — step-by-step
1. Modify model (add column, change type, add table)
2. Generate migration: `alembic revision --autogenerate -m "description"`
3. Review generated migration (autogenerate is not perfect)
4. Test migration: apply and rollback in dev
5. Commit migration file alongside model changes
6. CI runs migrations against test database

---

## How to Apply

### Decision Checklist
- [ ] Is session lifecycle managed properly (per-request)?
- [ ] Are relationships loaded eagerly where needed (N+1 prevention)?
- [ ] Are migrations auto-generated and reviewed?
- [ ] Is SQLAlchemy Core used for complex/bulk queries?
- [ ] Is generated SQL inspected for performance?

### Implementation Patterns

**SQLAlchemy 2.0 Models:**
```python
from sqlalchemy import String, ForeignKey, Numeric
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
from datetime import datetime

class Base(DeclarativeBase):
    pass

class Order(Base):
    __tablename__ = "orders"

    id: Mapped[str] = mapped_column(String(36), primary_key=True)
    customer_id: Mapped[str] = mapped_column(String(36), index=True)
    status: Mapped[str] = mapped_column(String(20), default="pending")
    total: Mapped[float] = mapped_column(Numeric(10, 2))
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)

    items: Mapped[list["OrderItem"]] = relationship(
        back_populates="order", cascade="all, delete-orphan"
    )

class OrderItem(Base):
    __tablename__ = "order_items"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    order_id: Mapped[str] = mapped_column(ForeignKey("orders.id"))
    product_id: Mapped[str] = mapped_column(String(36))
    quantity: Mapped[int]
    unit_price: Mapped[float] = mapped_column(Numeric(10, 2))

    order: Mapped["Order"] = relationship(back_populates="items")
```

**Session Management (async with FastAPI):**
```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

engine = create_async_engine("postgresql+asyncpg://user:pass@localhost/db")
async_session = async_sessionmaker(engine, expire_on_commit=False)

# FastAPI dependency
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

@app.get("/orders/{order_id}")
async def get_order(order_id: str, db: AsyncSession = Depends(get_db)):
    result = await db.execute(
        select(Order)
        .where(Order.id == order_id)
        .options(selectinload(Order.items))  # Eager load items
    )
    order = result.scalar_one_or_none()
    if not order:
        raise HTTPException(404)
    return OrderResponse.model_validate(order)
```

**N+1 Prevention:**
```python
from sqlalchemy.orm import selectinload, joinedload

# BAD: N+1 — lazy loads items for each order
orders = (await db.execute(select(Order))).scalars().all()
for order in orders:
    print(order.items)  # Each access triggers a separate query!

# GOOD: eager loading
orders = (await db.execute(
    select(Order).options(selectinload(Order.items))
)).scalars().all()
for order in orders:
    print(order.items)  # Already loaded, no extra queries

# joinedload for single-value relationships
user = (await db.execute(
    select(Order)
    .options(joinedload(Order.customer))  # LEFT JOIN
    .where(Order.id == order_id)
)).scalar_one()
```

**Repository Pattern with SQLAlchemy:**
```python
from typing import Generic, TypeVar
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

T = TypeVar("T", bound=Base)

class SQLAlchemyRepository(Generic[T]):
    def __init__(self, session: AsyncSession, model: type[T]) -> None:
        self._session = session
        self._model = model

    async def get(self, id: str) -> T | None:
        return await self._session.get(self._model, id)

    async def list(self, limit: int = 100, offset: int = 0) -> list[T]:
        result = await self._session.execute(
            select(self._model).limit(limit).offset(offset)
        )
        return list(result.scalars().all())

    async def save(self, entity: T) -> T:
        self._session.add(entity)
        await self._session.flush()
        return entity

    async def delete(self, entity: T) -> None:
        await self._session.delete(entity)

# Usage
class OrderRepository(SQLAlchemyRepository[Order]):
    async def find_by_customer(self, customer_id: str) -> list[Order]:
        result = await self._session.execute(
            select(Order)
            .where(Order.customer_id == customer_id)
            .options(selectinload(Order.items))
            .order_by(Order.created_at.desc())
        )
        return list(result.scalars().all())
```

### Common Mistakes
1. **N+1 queries** — Lazy loading in loops; always use selectinload/joinedload for needed relationships
2. **Session scope leaks** — Session kept open too long or shared across requests; scope per-request
3. **Not reviewing auto-generated migrations** — Alembic autogenerate misses some changes; always review
4. **Raw SQL strings everywhere** — Using `text()` for everything; use SQLAlchemy Core for type-safe queries
5. **Fat models** — Business logic in ORM models; keep models as data mappers, logic in services

### Integration with Other Topics
- **Web Frameworks** — SQLAlchemy with FastAPI/Flask; Django ORM with Django
- **Clean Architecture** — Repository pattern wraps ORM; domain models separate from DB models
- **Testing** — In-memory SQLite for fast unit tests; test database for integration
- **Database Design** — ORM models reflect relational schema
- **Async Programming** — AsyncSession for non-blocking database operations

---

## Resources

### Essential Reading
- SQLAlchemy 2.0 documentation (docs.sqlalchemy.org)
- Alembic documentation (alembic.sqlalchemy.org)
- Django ORM documentation

### Online Resources
- SQLAlchemy tutorial (docs.sqlalchemy.org/en/20/tutorial)
- Real Python — SQLAlchemy tutorials
- Mike Bayer's talks on SQLAlchemy design

### Practice Exercises
1. Define SQLAlchemy 2.0 models with relationships and configure async session
2. Set up Alembic and create 3 migrations (create, add column, add index)
3. Implement N+1 prevention with selectinload and joinedload
4. Build a generic Repository class with SQLAlchemy
5. Compare SQLAlchemy query performance with and without eager loading
