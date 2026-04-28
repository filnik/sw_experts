# Web Frameworks

## Profile

### What It Is
Python web frameworks provide the structure for building web applications and APIs. The landscape spans from full-stack (Django) to micro-frameworks (Flask) to modern async-first API frameworks (FastAPI). Choosing the right framework depends on the project type, team experience, and performance requirements.

### Origin and Evolution
- CGI scripts (1990s) — raw HTTP handling
- Django (2005) — "batteries-included" full-stack framework
- Flask (2010) — lightweight micro-framework, WSGI
- Tornado (2009) — async web framework
- FastAPI (2018) — async, type-annotated, OpenAPI auto-generation
- Litestar (2022) — FastAPI alternative with different design choices
- Current: FastAPI dominates API development; Django for full-stack; Flask declining

### Key Authors and References
- **Adrian Holovaty & Simon Willison** — Django creators
- **Armin Ronacher** — Flask creator
- **Sebastian Ramirez** — FastAPI creator
- **Tom Christie** — Starlette (FastAPI foundation), httpx

### Key Concepts at a Glance
| Framework | Best For | Key Feature |
|-----------|----------|-------------|
| FastAPI | APIs, microservices | Async, Pydantic validation, auto OpenAPI |
| Django | Full-stack web apps | ORM, admin, auth, batteries-included |
| Flask | Simple apps, learning | Minimal, extensible, large ecosystem |
| Litestar | APIs (FastAPI alternative) | Performance, DI, different design |
| Django Ninja | Django + FastAPI-style APIs | Django ecosystem with type-annotated APIs |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Match framework to project** — FastAPI for APIs, Django for full-stack apps with admin/auth. Don't build a REST API in Django templates or a CMS in FastAPI.
2. **Async-first for new APIs** — FastAPI and Litestar are async-native. For new API projects, choose async unless you have a compelling reason not to.
3. **Framework is not architecture** — The framework handles HTTP. Your business logic should be framework-independent. Don't put domain logic in views/routes.
4. **Leverage the ecosystem** — Django's admin, ORM, and auth are powerful. FastAPI's Pydantic integration and auto-docs are valuable. Use what the framework offers.
5. **Performance matters at scale** — For high-throughput APIs, FastAPI + uvicorn + async is significantly faster than Django + gunicorn + sync.

### When to Use vs. When to Avoid

**FastAPI**: REST/GraphQL APIs, microservices, async workloads, ML model serving
**Django**: Full-stack web apps, admin panels, CMS, rapid prototyping with built-in features
**Flask**: Simple web apps, when you want full control, prototyping
**Litestar**: High-performance APIs, projects wanting FastAPI features with different design

---

## Frameworks and Methodologies

### 1. FastAPI Application Design — step-by-step
1. Structure with routers (one per feature/resource)
2. Define Pydantic models for request/response
3. Use Depends() for dependency injection (DB, auth, services)
4. Add middleware for cross-cutting concerns (CORS, logging)
5. Configure lifespan events for startup/shutdown
6. Generate OpenAPI docs automatically

### 2. Django Project Design — step-by-step
1. Create Django project and apps (one per domain area)
2. Define models and migrations
3. Create views (function-based or class-based)
4. Set up URL routing
5. Configure admin for data management
6. Add DRF (Django REST Framework) or Django Ninja for APIs

---

## How to Apply

### Decision Checklist
- [ ] Is the framework choice appropriate for the project type?
- [ ] Is business logic separated from framework code?
- [ ] Are request/response schemas defined (Pydantic, DRF serializers)?
- [ ] Is dependency injection used for services and repositories?
- [ ] Is the API documented (OpenAPI, DRF browsable API)?

### Implementation Patterns

**FastAPI Application:**
```python
from fastapi import FastAPI, Depends, HTTPException, status
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    await database.connect()
    yield
    # Shutdown
    await database.disconnect()

app = FastAPI(title="Order Service", lifespan=lifespan)

# Router for feature separation
from fastapi import APIRouter

router = APIRouter(prefix="/orders", tags=["orders"])

@router.post("/", status_code=status.HTTP_201_CREATED)
async def create_order(
    request: CreateOrderRequest,
    service: OrderService = Depends(get_order_service),
    user: User = Depends(get_current_user),
) -> OrderResponse:
    order = await service.create(request, user_id=user.id)
    return OrderResponse.model_validate(order)

@router.get("/{order_id}")
async def get_order(
    order_id: str,
    service: OrderService = Depends(get_order_service),
) -> OrderResponse:
    order = await service.get(order_id)
    if not order:
        raise HTTPException(status_code=404, detail="Order not found")
    return OrderResponse.model_validate(order)

app.include_router(router)
```

**Django with Django Ninja:**
```python
# Django Ninja — FastAPI-like API in Django
from ninja import NinjaAPI, Schema
from ninja.security import HttpBearer

api = NinjaAPI()

class OrderIn(Schema):
    customer_id: str
    items: list[OrderItemIn]

class OrderOut(Schema):
    id: str
    customer_id: str
    total: float
    status: str
    created_at: datetime

@api.post("/orders/", response=OrderOut)
def create_order(request, payload: OrderIn):
    order = Order.objects.create(
        customer_id=payload.customer_id,
    )
    for item in payload.items:
        OrderItem.objects.create(order=order, **item.dict())
    return order

@api.get("/orders/{order_id}", response=OrderOut)
def get_order(request, order_id: str):
    return get_object_or_404(Order, id=order_id)
```

**Framework Comparison:**
```python
# Same endpoint in different frameworks

# --- FastAPI ---
@app.get("/users/{user_id}")
async def get_user(user_id: str, db: Session = Depends(get_db)) -> UserResponse:
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(404)
    return UserResponse.model_validate(user)

# --- Flask ---
@app.route("/users/<user_id>")
def get_user(user_id: str):
    user = db.session.get(User, user_id)
    if not user:
        abort(404)
    return jsonify(user.to_dict())

# --- Django (function-based view) ---
def get_user(request, user_id):
    user = get_object_or_404(User, id=user_id)
    return JsonResponse(UserSerializer(user).data)
```

### Common Mistakes
1. **Django for pure APIs** — Using Django templates and views for a REST API; use Django Ninja or DRF
2. **Business logic in routes** — Views/routes containing database queries, validation, and business rules; extract to services
3. **Ignoring async** — Building new sync APIs when async frameworks handle concurrency better
4. **Not using Depends** — Creating services and DB connections inside route handlers instead of injecting
5. **Over-engineering with Flask** — Adding ORM, auth, admin manually when Django provides them built-in

### Integration with Other Topics
- **Pydantic** — Request/response validation in FastAPI
- **Dependency Injection** — FastAPI Depends, Django middleware
- **Testing** — TestClient (FastAPI/Starlette), Django test client
- **Async Programming** — FastAPI is async-native
- **API Design** — Framework implements REST/GraphQL design decisions
- **ORM Patterns** — SQLAlchemy (FastAPI/Flask), Django ORM

---

## Resources

### Essential Reading
- FastAPI documentation (fastapi.tiangolo.com)
- Django documentation (docs.djangoproject.com)
- Flask documentation (flask.palletsprojects.com)

### Online Resources
- Litestar documentation (litestar.dev)
- Django Ninja documentation
- Real Python — Framework comparison articles

### Practice Exercises
1. Build the same CRUD API in FastAPI and Django Ninja — compare DX
2. Structure a FastAPI project with routers, Depends, and Pydantic models
3. Add authentication middleware to a FastAPI application
4. Set up Django with admin panel and Django Ninja API
5. Benchmark FastAPI async vs. Django sync for concurrent API calls
