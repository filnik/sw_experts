# Data Validation with Pydantic

## Profile

### What It Is
Pydantic is Python's dominant data validation library. It uses type annotations to define data models, validates data at runtime, serializes/deserializes to JSON, and provides settings management. Pydantic v2 (2023) is a ground-up rewrite with a Rust core (pydantic-core) for dramatically better performance.

### Origin and Evolution
- Pydantic v1 (2018) — Data validation using type annotations
- FastAPI (2018) — Built on Pydantic; drove massive adoption
- Pydantic v2 (2023) — Rewritten with Rust core, 5-50x faster
- PEP 593 `Annotated` types — Pydantic v2 embraces `Annotated[int, Field(gt=0)]`
- Current: Pydantic v2 is the standard for data validation, settings, serialization

### Key Authors and References
- **Samuel Colvin** — Pydantic creator
- **Sebastian Ramirez** — FastAPI creator, Pydantic integration
- **Pydantic documentation** — comprehensive reference

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| BaseModel | Declarative model with validation on instantiation |
| Field | Field metadata: default, validation, alias, description |
| Validators | `@field_validator`, `@model_validator` for custom validation |
| BaseSettings | Environment variable and config file loading |
| Serialization | `.model_dump()`, `.model_dump_json()`, `.model_validate()` |
| Annotated | `Annotated[str, Field(min_length=1)]` for reusable constraints |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Validate at the boundary** — Use Pydantic at system boundaries (API input, config, external data). Internal code trusts validated types.
2. **Type annotations as schema** — The model definition IS the validation schema. No separate schema definition language.
3. **Fail fast with clear errors** — Pydantic raises `ValidationError` with detailed field-level errors on invalid data. Users get actionable error messages.
4. **Immutable models for safety** — Use `model_config = ConfigDict(frozen=True)` for value objects that shouldn't change after creation.
5. **Strict vs. lax modes** — Default: lax (coerces "123" to int). Strict mode: no coercion. Choose based on context.

### When to Use vs. When to Avoid

**Use Pydantic when:**
- Validating external input (API requests, form data, config files)
- Defining API response schemas (FastAPI)
- Loading configuration from environment variables
- Parsing data from external services (JSON APIs, databases)

**Use dataclasses when:**
- Internal data structures with no validation needs
- Performance-critical internal models
- Simple data holders without serialization requirements

---

## Frameworks and Methodologies

### 1. API Schema Design — step-by-step
1. Define input models (create/update request bodies)
2. Define output models (response schemas, separate from internal models)
3. Add field validations (min/max, regex, custom validators)
4. Add model-level validators for cross-field validation
5. Use `Annotated` types for reusable field constraints
6. Generate OpenAPI schema automatically (FastAPI)

### 2. Settings Management — step-by-step
1. Create `BaseSettings` subclass for each config group
2. Define fields with types and defaults
3. Configure env file loading (`.env`)
4. Use nested models for grouped settings
5. Validate settings at startup (fail fast)
6. Use `@field_validator` for complex validation (URL format, port ranges)

---

## How to Apply

### Decision Checklist
- [ ] Are all external inputs validated with Pydantic models?
- [ ] Are API input and output models separate?
- [ ] Are settings loaded via BaseSettings?
- [ ] Are reusable constraints defined with Annotated types?
- [ ] Are custom validators used for business rules?

### Implementation Patterns

**Basic Models:**
```python
from pydantic import BaseModel, Field, ConfigDict
from datetime import datetime

class OrderItem(BaseModel):
    product_id: str
    quantity: int = Field(gt=0, description="Must be positive")
    unit_price: float = Field(gt=0, alias="unitPrice")

class CreateOrderRequest(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True)

    customer_id: str = Field(min_length=1)
    items: list[OrderItem] = Field(min_length=1)
    notes: str | None = None

class OrderResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: str
    customer_id: str
    items: list[OrderItem]
    total: float
    status: str
    created_at: datetime

# Validation happens on instantiation
order = CreateOrderRequest(
    customer_id="C123",
    items=[{"product_id": "P1", "quantity": 2, "unitPrice": 29.99}],
)

# Serialization
order.model_dump()              # dict
order.model_dump_json()          # JSON string
order.model_dump(exclude={"notes"})  # Exclude fields
```

**Custom Validators:**
```python
from pydantic import BaseModel, field_validator, model_validator

class UserRegistration(BaseModel):
    email: str
    password: str = Field(min_length=8)
    password_confirm: str
    age: int = Field(ge=18)

    @field_validator("email")
    @classmethod
    def validate_email(cls, v: str) -> str:
        if "@" not in v or "." not in v.split("@")[-1]:
            raise ValueError("Invalid email format")
        return v.lower().strip()

    @field_validator("password")
    @classmethod
    def validate_password_strength(cls, v: str) -> str:
        if not any(c.isupper() for c in v):
            raise ValueError("Password must contain uppercase letter")
        if not any(c.isdigit() for c in v):
            raise ValueError("Password must contain a digit")
        return v

    @model_validator(mode="after")
    def passwords_match(self) -> "UserRegistration":
        if self.password != self.password_confirm:
            raise ValueError("Passwords do not match")
        return self
```

**Annotated Types for Reusable Constraints:**
```python
from typing import Annotated
from pydantic import BaseModel, Field, StringConstraints

# Define reusable types
NonEmptyStr = Annotated[str, StringConstraints(min_length=1, strip_whitespace=True)]
PositiveInt = Annotated[int, Field(gt=0)]
Email = Annotated[str, StringConstraints(pattern=r"^[\w.-]+@[\w.-]+\.\w+$", to_lower=True)]
Slug = Annotated[str, StringConstraints(pattern=r"^[a-z0-9-]+$", max_length=50)]

# Use in any model
class Product(BaseModel):
    name: NonEmptyStr
    slug: Slug
    price: Annotated[float, Field(gt=0, le=999999)]
    stock: PositiveInt

class Customer(BaseModel):
    name: NonEmptyStr
    email: Email
```

**Settings Management:**
```python
from pydantic import Field
from pydantic_settings import BaseSettings

class DatabaseSettings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="DB_")

    host: str = "localhost"
    port: int = 5432
    name: str = "myapp"
    user: str = "postgres"
    password: str  # Required, no default
    pool_size: int = Field(default=10, ge=1, le=100)

    @property
    def url(self) -> str:
        return f"postgresql://{self.user}:{self.password}@{self.host}:{self.port}/{self.name}"

class AppSettings(BaseSettings):
    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
    )

    debug: bool = False
    secret_key: str
    allowed_origins: list[str] = ["http://localhost:3000"]
    db: DatabaseSettings = DatabaseSettings()

# Reads from environment variables:
# SECRET_KEY=xxx DB_HOST=db.example.com DB_PASSWORD=xxx
settings = AppSettings()
```

**Discriminated Unions:**
```python
from typing import Annotated, Literal, Union
from pydantic import BaseModel, Field

class EmailNotification(BaseModel):
    type: Literal["email"] = "email"
    to: str
    subject: str
    body: str

class SlackNotification(BaseModel):
    type: Literal["slack"] = "slack"
    channel: str
    message: str

class WebhookNotification(BaseModel):
    type: Literal["webhook"] = "webhook"
    url: str
    payload: dict

Notification = Annotated[
    Union[EmailNotification, SlackNotification, WebhookNotification],
    Field(discriminator="type"),
]

class NotificationRequest(BaseModel):
    notifications: list[Notification]

# Automatic parsing based on "type" field
data = {"notifications": [
    {"type": "email", "to": "a@b.com", "subject": "Hi", "body": "Hello"},
    {"type": "slack", "channel": "#general", "message": "Hello"},
]}
req = NotificationRequest.model_validate(data)
```

### Common Mistakes
1. **Using Pydantic for internal models** — Heavy validation for internal data that's already trusted; use dataclasses
2. **Same model for input and output** — Exposing internal fields to API; separate request/response models
3. **Not using Annotated** — Repeating `Field(gt=0)` everywhere instead of defining `PositiveInt` once
4. **Mutable models for value objects** — Not using `frozen=True` for models that shouldn't change
5. **Ignoring validation errors** — Catching `ValidationError` and returning generic error; expose detailed field errors

### Integration with Other Topics
- **FastAPI** — Built on Pydantic; automatic request validation and OpenAPI schema
- **Type Hints** — Pydantic uses type annotations as the validation schema
- **Error Handling** — `ValidationError` provides structured error details
- **Clean Architecture** — Pydantic at boundaries; domain models as dataclasses/plain classes
- **Testing** — Factory patterns with Pydantic models for test data

---

## Resources

### Essential Reading
- Pydantic v2 documentation (docs.pydantic.dev)
- FastAPI documentation — request validation sections
- Samuel Colvin — Pydantic v2 migration guide

### Online Resources
- pydantic-settings documentation
- Real Python — Pydantic tutorials
- Pydantic v1 to v2 migration guide

### Practice Exercises
1. Define API request/response models with field validators
2. Create reusable Annotated types for common constraints
3. Implement BaseSettings for a multi-environment application
4. Build discriminated unions for a notification system
5. Benchmark Pydantic v2 vs. manual validation for a complex model
