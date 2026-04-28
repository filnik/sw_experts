# Error Handling in TypeScript

## Profile

### What It Is
TypeScript error handling patterns go beyond try/catch — using discriminated unions to create type-safe Result/Either types, typed error hierarchies, and explicit error handling that the compiler can verify. This approach makes error states visible in the type system rather than hidden in thrown exceptions.

### Origin and Evolution
- JavaScript try/catch — untyped, any value can be thrown
- TypeScript: `catch (e)` is `unknown` (since TS 4.4 with `useUnknownInCatchVariables`)
- Influence from Rust (`Result<T, E>`), Haskell (`Either`), Go (error return values)
- `neverthrow` library — Result type for TypeScript
- `effect` library — Effect system with typed errors
- Current: discriminated unions + neverthrow for explicit error handling

### Key Authors and References
- **Matt Pocock** — TypeScript error handling patterns
- **neverthrow** — Popular Result type library
- **Effect** — Typed effect system (inspired by ZIO)
- **Dan Vanderkam** — Error handling in *Effective TypeScript*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Result<T, E> | Union of `Ok<T>` or `Err<E>` — explicit success/failure |
| Discriminated Union | `{success: true, data: T} \| {success: false, error: E}` |
| neverthrow | Library: `ok(value)` / `err(error)` with chainable API |
| Typed Errors | Error classes with discriminant for type narrowing |
| Exhaustive Error Handling | Compiler ensures all error types are handled |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Make errors visible in types** — Functions that can fail should show it in their return type (`Result<T, E>`), not just throw exceptions.
2. **Discriminated unions for error types** — Model errors with a `type` discriminant: `{type: "not_found"}`, `{type: "validation", fields: ...}`. Enables exhaustive handling.
3. **Exceptions for truly exceptional cases** — Use try/catch for unexpected failures (network errors, runtime errors). Use Result for expected business failures (validation, not found).
4. **Never use `any` in catch blocks** — `catch (e: unknown)` is the safe default. Narrow with `instanceof` or type guards.
5. **Chain results, don't nest try/catch** — Use `result.map()`, `result.andThen()` to chain operations. Avoids deeply nested try/catch.

### When to Use vs. When to Avoid

**Use Result pattern when:**
- Business logic failures (validation, not found, conflicts)
- Multiple error types need different handling
- Functional composition of fallible operations

**Use exceptions when:**
- Unexpected runtime errors (network, file system)
- Framework boundaries (Express/Fastify error middleware)
- Simple scripts where Result adds overhead

---

## Frameworks and Methodologies

### 1. Error Strategy Design — step-by-step
1. Classify errors: expected (business) vs. unexpected (infrastructure)
2. Use Result for expected errors
3. Use exceptions for unexpected errors
4. Define error types as discriminated unions
5. Handle all error cases exhaustively
6. Map errors at boundaries (domain → HTTP status codes)

### 2. neverthrow Adoption — step-by-step
1. Install `neverthrow`
2. Return `ok(value)` or `err(error)` from functions
3. Chain operations with `.map()`, `.andThen()`
4. Handle results with `.match()` or pattern matching
5. Combine multiple results with `Result.combine()`

---

## How to Apply

### Decision Checklist
- [ ] Are business failures returned as Results (not thrown)?
- [ ] Are error types defined with discriminated unions?
- [ ] Is error handling exhaustive (all error types handled)?
- [ ] Are catch blocks typed with `unknown`?
- [ ] Are errors mapped at layer boundaries?

### Implementation Patterns

**Discriminated Union Errors:**
```typescript
// Define error types
type OrderError =
  | { type: "not_found"; orderId: string }
  | { type: "validation"; errors: { field: string; message: string }[] }
  | { type: "insufficient_stock"; productId: string; available: number }
  | { type: "payment_failed"; reason: string };

// Simple Result type
type Result<T, E> = { success: true; data: T } | { success: false; error: E };

function createOrder(input: CreateOrderInput): Result<Order, OrderError> {
  const errors = validateOrder(input);
  if (errors.length > 0) {
    return { success: false, error: { type: "validation", errors } };
  }

  for (const item of input.items) {
    const stock = getStock(item.productId);
    if (stock < item.quantity) {
      return {
        success: false,
        error: { type: "insufficient_stock", productId: item.productId, available: stock },
      };
    }
  }

  const order = buildOrder(input);
  return { success: true, data: order };
}

// Exhaustive handling
function handleOrderResult(result: Result<Order, OrderError>): Response {
  if (result.success) {
    return { status: 201, body: result.data };
  }

  switch (result.error.type) {
    case "not_found":
      return { status: 404, body: `Order ${result.error.orderId} not found` };
    case "validation":
      return { status: 400, body: result.error.errors };
    case "insufficient_stock":
      return { status: 409, body: `Only ${result.error.available} available` };
    case "payment_failed":
      return { status: 402, body: result.error.reason };
  }
}
```

**neverthrow Library:**
```typescript
import { ok, err, Result, ResultAsync } from "neverthrow";

type CreateOrderError =
  | { type: "validation"; errors: string[] }
  | { type: "stock"; productId: string };

function validateInput(input: unknown): Result<CreateOrderInput, CreateOrderError> {
  const parsed = schema.safeParse(input);
  if (!parsed.success) {
    return err({ type: "validation", errors: parsed.error.issues.map((i) => i.message) });
  }
  return ok(parsed.data);
}

function checkStock(input: CreateOrderInput): Result<CreateOrderInput, CreateOrderError> {
  for (const item of input.items) {
    if (!hasStock(item.productId, item.quantity)) {
      return err({ type: "stock", productId: item.productId });
    }
  }
  return ok(input);
}

function processOrder(input: CreateOrderInput): Result<Order, CreateOrderError> {
  return ok(buildOrder(input));
}

// Chain with andThen — stops at first error
function createOrder(rawInput: unknown): Result<Order, CreateOrderError> {
  return validateInput(rawInput)
    .andThen(checkStock)
    .andThen(processOrder);
}

// Handle with match
const response = createOrder(body).match(
  (order) => ({ status: 201, body: order }),
  (error) => {
    switch (error.type) {
      case "validation": return { status: 400, body: error.errors };
      case "stock": return { status: 409, body: `Out of stock: ${error.productId}` };
    }
  },
);

// Async with ResultAsync
function fetchUser(id: string): ResultAsync<User, ApiError> {
  return ResultAsync.fromPromise(
    api.get(`/users/${id}`),
    (error) => ({ type: "network" as const, message: String(error) }),
  );
}
```

**Typed Exception Hierarchy:**
```typescript
// When exceptions are the right choice
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

class NotFoundError extends AppError {
  constructor(entity: string, id: string) {
    super(`${entity} not found: ${id}`, "NOT_FOUND", 404);
  }
}

class ValidationError extends AppError {
  constructor(public readonly errors: { field: string; message: string }[]) {
    super("Validation failed", "VALIDATION_ERROR", 400);
  }
}

// Error boundary middleware (Express/Fastify)
function errorHandler(err: unknown, req: Request, res: Response, next: NextFunction) {
  if (err instanceof AppError) {
    res.status(err.statusCode).json({ code: err.code, message: err.message });
  } else {
    console.error("Unexpected error:", err);
    res.status(500).json({ code: "INTERNAL_ERROR", message: "Internal server error" });
  }
}
```

**Safe Catch Pattern:**
```typescript
// TypeScript catch is 'unknown' — narrow safely
async function fetchData(url: string): Promise<Result<Data, FetchError>> {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      return err({ type: "http", status: response.status });
    }
    const data = await response.json();
    return ok(data as Data);
  } catch (error) {
    if (error instanceof TypeError) {
      return err({ type: "network", message: error.message });
    }
    return err({ type: "unknown", message: String(error) });
  }
}
```

### Common Mistakes
1. **`catch (e: any)`** — Loses type safety; use `unknown` and narrow
2. **Throwing strings** — `throw "error"` instead of `throw new Error("error")`; no stack trace
3. **Mixing Result and exceptions** — Inconsistent error handling; choose one strategy per layer
4. **Not handling all error types** — Missing a case in the switch; use exhaustive checks with `never`
5. **Swallowing errors** — `catch (e) {}` or `catch (e) { console.log(e) }` without recovery

### Integration with Other Topics
- **TypeScript Type System** — Discriminated unions for error types
- **Zod** — Validation errors as typed Results
- **React** — Error boundaries for component-level error handling
- **API Patterns** — Map domain errors to HTTP status codes
- **Testing** — Test error paths explicitly with Result assertions

---

## Resources

### Essential Reading
- *Effective TypeScript* — Dan Vanderkam (error handling items)
- neverthrow documentation (github.com/supermacro/neverthrow)
- Matt Pocock — Error handling patterns (totaltypescript.com)

### Online Resources
- Effect documentation (effect.website) — typed effect system
- TypeScript handbook — Narrowing and control flow analysis
- Total TypeScript — Error handling workshop

### Practice Exercises
1. Implement a Result type with ok/err/map/andThen/match
2. Refactor a try/catch chain to use neverthrow Result chaining
3. Create a typed error hierarchy for an API service
4. Map domain errors to HTTP responses in a middleware
5. Write exhaustive error handling with discriminated union switch
