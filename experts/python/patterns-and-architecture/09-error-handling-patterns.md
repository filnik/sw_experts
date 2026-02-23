# Error Handling Patterns

## Profile

### What It Is
Python error handling patterns cover exception hierarchy design, custom exceptions, the Result pattern for explicit error handling, and strategies for deciding when to use exceptions vs. return values. Well-designed error handling makes failure modes explicit, prevents silent failures, and keeps error recovery logic clean.

### Origin and Evolution
- Python's exception hierarchy (BaseException → Exception → specific errors)
- EAFP (Easier to Ask Forgiveness than Permission) — Pythonic exception style
- Result/Either pattern from functional programming (Rust, Haskell)
- `returns` library — Result, Maybe, IO monads for Python
- Current: exceptions for exceptional cases, Result pattern gaining adoption for expected failures

### Key Authors and References
- **Luciano Ramalho** — Exception handling in *Fluent Python*
- **Hynek Schlawack** — Exception design articles
- **Dry-python** — `returns` library (Result pattern)
- **Rust** — Result<T, E> pattern influence on Python community

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Custom Exception Hierarchy | Domain-specific exceptions organized by module/feature |
| Exception Groups | PEP 654 (Python 3.11); handle multiple concurrent exceptions |
| Result Pattern | Return `Ok(value)` or `Err(error)` instead of raising |
| EAFP | Try the operation, catch failure; Pythonic approach |
| LBYL | Check preconditions before acting; sometimes clearer |
| Chained Exceptions | `raise NewError() from original_error` for context |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Exceptions for exceptional situations** — Use exceptions for unexpected failures (network down, file missing). Use return values or Result for expected outcomes (user not found, validation failed).
2. **Catch specific exceptions** — Never bare `except:`. Catch the specific exception type. Let unexpected errors propagate.
3. **Custom exceptions per domain** — Create a base exception for your package and derive from it. Callers can catch all your errors with one type.
4. **Chain exceptions for context** — Use `raise NewError("message") from original` to preserve the cause chain.
5. **Fail fast, fail loudly** — Don't silently swallow errors. If you catch an exception and can't handle it, re-raise or log and re-raise.

### When to Use vs. When to Avoid

**Use exceptions when:**
- Failure is unexpected and exceptional (I/O errors, system failures)
- Error must propagate up multiple call levels
- Python stdlib and frameworks expect exceptions

**Use Result pattern when:**
- Failure is an expected outcome (validation, lookup)
- You want type-safe explicit error handling
- Multiple error types need distinct handling without try/except chains

---

## Frameworks and Methodologies

### 1. Exception Hierarchy Design — step-by-step
1. Create a base exception for your package: `class MyAppError(Exception)`
2. Create category exceptions: `class ValidationError(MyAppError)`
3. Create specific exceptions: `class InvalidEmailError(ValidationError)`
4. Add structured data to exceptions (error codes, details)
5. Document which functions raise which exceptions
6. Never inherit from `BaseException` (only for system exits)

### 2. Result Pattern Adoption — step-by-step
1. Define `Result[T, E]` type (or use `returns` library)
2. Replace exceptions with `Ok(value)` / `Err(error)` returns
3. Use pattern matching (3.10+) for result handling
4. Keep exceptions for truly exceptional cases (I/O, system)
5. Use Result for business logic outcomes (validation, lookup)

---

## How to Apply

### Decision Checklist
- [ ] Are custom exceptions organized in a hierarchy?
- [ ] Are only specific exceptions caught (no bare except)?
- [ ] Are exceptions chained with `from` for context?
- [ ] Is the Result pattern used for expected failure cases?
- [ ] Are errors logged with sufficient context for debugging?

### Implementation Patterns

**Custom Exception Hierarchy:**
```python
# Base exception for the package
class OrderServiceError(Exception):
    """Base exception for order service."""

class OrderNotFoundError(OrderServiceError):
    def __init__(self, order_id: str) -> None:
        self.order_id = order_id
        super().__init__(f"Order not found: {order_id}")

class OrderValidationError(OrderServiceError):
    def __init__(self, field: str, message: str) -> None:
        self.field = field
        super().__init__(f"Validation error on {field}: {message}")

class PaymentError(OrderServiceError):
    def __init__(self, order_id: str, reason: str) -> None:
        self.order_id = order_id
        self.reason = reason
        super().__init__(f"Payment failed for {order_id}: {reason}")

# Usage: catch broad or specific
try:
    process_order(order_id)
except OrderNotFoundError:
    return {"error": "Order not found"}, 404
except OrderValidationError as e:
    return {"error": str(e), "field": e.field}, 400
except OrderServiceError:
    return {"error": "Order processing failed"}, 500
```

**Exception Chaining:**
```python
def fetch_user_profile(user_id: str) -> UserProfile:
    try:
        response = httpx.get(f"{API_URL}/users/{user_id}")
        response.raise_for_status()
    except httpx.HTTPStatusError as e:
        raise UserNotFoundError(user_id) from e
    except httpx.ConnectError as e:
        raise ServiceUnavailableError("user-service") from e

    try:
        return UserProfile.model_validate(response.json())
    except ValidationError as e:
        raise DataCorruptionError(f"Invalid user data for {user_id}") from e
```

**Result Pattern (simple implementation):**
```python
from dataclasses import dataclass
from typing import Generic, TypeVar, Union

T = TypeVar("T")
E = TypeVar("E")

@dataclass(frozen=True)
class Ok(Generic[T]):
    value: T

@dataclass(frozen=True)
class Err(Generic[E]):
    error: E

Result = Union[Ok[T], Err[E]]

# Domain errors as data, not exceptions
@dataclass(frozen=True)
class ValidationFailure:
    field: str
    message: str

@dataclass(frozen=True)
class NotFound:
    entity: str
    id: str

def validate_order(data: dict) -> Result[Order, ValidationFailure]:
    if not data.get("items"):
        return Err(ValidationFailure("items", "Order must have at least one item"))
    if data.get("total", 0) <= 0:
        return Err(ValidationFailure("total", "Total must be positive"))
    return Ok(Order(**data))

def find_order(order_id: str) -> Result[Order, NotFound]:
    order = db.query(Order).get(order_id)
    if order is None:
        return Err(NotFound("Order", order_id))
    return Ok(order)

# Pattern matching (Python 3.10+)
match find_order("123"):
    case Ok(order):
        print(f"Found: {order.id}")
    case Err(NotFound(entity, id)):
        print(f"{entity} {id} not found")
```

**Structured Error Responses:**
```python
from dataclasses import dataclass, field
from enum import Enum

class ErrorCode(Enum):
    VALIDATION = "VALIDATION_ERROR"
    NOT_FOUND = "NOT_FOUND"
    CONFLICT = "CONFLICT"
    INTERNAL = "INTERNAL_ERROR"

@dataclass
class ErrorDetail:
    code: ErrorCode
    message: str
    field: str | None = None
    details: dict | None = None

class AppError(Exception):
    def __init__(self, error: ErrorDetail) -> None:
        self.error = error
        super().__init__(error.message)

# FastAPI error handler
@app.exception_handler(AppError)
async def handle_app_error(request: Request, exc: AppError):
    status_map = {
        ErrorCode.VALIDATION: 400,
        ErrorCode.NOT_FOUND: 404,
        ErrorCode.CONFLICT: 409,
        ErrorCode.INTERNAL: 500,
    }
    return JSONResponse(
        status_code=status_map[exc.error.code],
        content={"error": asdict(exc.error)},
    )
```

### Common Mistakes
1. **Bare `except:`** — Catches `KeyboardInterrupt`, `SystemExit`; always specify exception type
2. **Swallowing exceptions** — `except Exception: pass` hides bugs; at minimum log the error
3. **Using exceptions for control flow** — `StopIteration` is the exception; don't use exceptions for normal branching
4. **No exception chaining** — `raise NewError()` without `from original` loses the root cause
5. **Stringly-typed errors** — Raising `ValueError("invalid email")` instead of `InvalidEmailError`; can't catch specifically

### Integration with Other Topics
- **Clean Architecture** — Domain exceptions at the core; translated at boundaries
- **Testing** — `pytest.raises` for expected exceptions; Result pattern simplifies assertions
- **Web Frameworks** — Exception handlers map domain errors to HTTP responses
- **Type Hints** — Result pattern enables type-safe error handling
- **Pydantic** — `ValidationError` for input validation at boundaries

---

## Resources

### Essential Reading
- *Fluent Python* (2nd ed.) — Exception handling chapters
- *Robust Python* — Patrick Viafore (error handling patterns)
- PEP 654 — Exception Groups and except* (Python 3.11)

### Online Resources
- returns library documentation (returns.readthedocs.io)
- Real Python — Exception handling tutorials
- Python docs — Built-in exceptions hierarchy

### Practice Exercises
1. Design a custom exception hierarchy for a payment module
2. Implement the Result pattern and refactor a function from exceptions
3. Add exception chaining to an existing HTTP client wrapper
4. Create a FastAPI error handler that maps domain errors to HTTP status codes
5. Refactor bare `except` clauses in existing code to specific exception types
