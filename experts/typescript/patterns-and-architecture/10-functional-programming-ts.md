# Functional Programming in TypeScript

## Profile

### What It Is
Functional programming (FP) in TypeScript leverages immutability, pure functions, composition, and algebraic data types (Option, Either, Result) to write predictable, testable code. TypeScript's type system supports FP patterns well — generics, discriminated unions, and type inference make FP practical without sacrificing readability.

### Origin and Evolution
- JavaScript's functional roots (first-class functions, closures)
- Lodash/Ramda (2013+) — Utility libraries with FP patterns
- fp-ts (2017, Giulio Canti) — Full FP library (monads, functors)
- Effect (2022+) — Modern TypeScript effect system (inspired by ZIO/Scala)
- Current: pragmatic FP (immutability, composition, Result) preferred over heavy FP (monads, HKT)

### Key Authors and References
- **Giulio Canti** — fp-ts creator
- **Michael Arnaldi** — Effect creator
- **Kyle Simpson** — *Functional-Light JavaScript*
- **Luis Atencio** — *Functional Programming in JavaScript*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Pure Functions | Same input → same output, no side effects |
| Immutability | Never mutate; create new values |
| Function Composition | `pipe(f, g, h)` — combine small functions into pipelines |
| Option/Maybe | `Some(value)` or `None` — type-safe nullable handling |
| Either/Result | `Right(value)` or `Left(error)` — type-safe error handling |
| Pipe | `pipe(value, fn1, fn2, fn3)` — left-to-right composition |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Immutability by default** — Use `readonly`, `as const`, spread operators. Never mutate arrays or objects. Create new values instead.
2. **Pure functions for logic** — Business logic should be pure: no side effects, no state mutation, no I/O. Side effects at the edges.
3. **Pipe for composition** — `pipe(data, validate, transform, save)` is more readable than nested function calls. Left-to-right, step by step.
4. **Pragmatic FP** — Use Option/Result for nullable and error handling. Don't force monads on the whole codebase. Mix FP and OO where each is natural.
5. **Types make FP safe** — TypeScript's generics and discriminated unions make FP patterns type-safe. Option<T>, Result<T, E> — the compiler verifies correctness.

### When to Use vs. When to Avoid

**Use FP patterns when:**
- Data transformation pipelines (ETL, API response mapping)
- Business logic that benefits from pure functions
- Error handling with Result/Either
- Complex data processing with composition

**Keep it simple when:**
- Team is unfamiliar with FP concepts
- Framework patterns are OO (NestJS, Angular)
- FP adds abstraction without clarity
- Simple imperative code is more readable

---

## Frameworks and Methodologies

### 1. Pragmatic FP Adoption — step-by-step
1. Enforce immutability (`readonly`, `as const`, no mutation)
2. Extract pure functions from classes/components
3. Use `pipe()` for data transformation chains
4. Adopt Option for nullable handling
5. Adopt Result/Either for error handling
6. Add Effect or fp-ts only for complex async workflows

### 2. Pipeline Design — step-by-step
1. Define input type
2. Break processing into small pure functions
3. Compose with pipe
4. Handle errors with Result at each step
5. Execute side effects at the end of the pipeline

---

## How to Apply

### Decision Checklist
- [ ] Are data structures immutable (readonly, no mutation)?
- [ ] Are business logic functions pure?
- [ ] Is function composition used (pipe) for data transformations?
- [ ] Is Option used instead of null checks?
- [ ] Is Result used for expected error handling?

### Implementation Patterns

**Immutability:**
```typescript
// Readonly types
type User = Readonly<{
  id: string;
  name: string;
  roles: readonly string[];
}>;

// as const for literal types
const config = {
  maxRetries: 3,
  timeout: 5000,
  features: ["auth", "logging"],
} as const;
// type: { readonly maxRetries: 3; readonly timeout: 5000; readonly features: readonly ["auth", "logging"] }

// Immutable updates
function addRole(user: User, role: string): User {
  return { ...user, roles: [...user.roles, role] };
}

// Immutable array operations
const items = [1, 2, 3];
const added = [...items, 4];         // [1, 2, 3, 4]
const removed = items.filter(x => x !== 2);  // [1, 3]
const updated = items.map(x => x === 2 ? 20 : x);  // [1, 20, 3]
```

**Pipe and Composition:**
```typescript
// Simple pipe function
function pipe<A, B>(a: A, ab: (a: A) => B): B;
function pipe<A, B, C>(a: A, ab: (a: A) => B, bc: (b: B) => C): C;
function pipe<A, B, C, D>(a: A, ab: (a: A) => B, bc: (b: B) => C, cd: (c: C) => D): D;
function pipe(value: unknown, ...fns: Function[]): unknown {
  return fns.reduce((acc, fn) => fn(acc), value);
}

// Usage
const processOrder = (rawData: unknown) =>
  pipe(
    rawData,
    validateOrderInput,     // unknown → OrderInput
    calculateTotal,         // OrderInput → OrderWithTotal
    applyDiscount,          // OrderWithTotal → OrderWithDiscount
    formatForDatabase,      // OrderWithDiscount → OrderRecord
  );

// Reusable transformation functions
const normalize = (s: string) => s.trim().toLowerCase();
const capitalize = (s: string) => s.charAt(0).toUpperCase() + s.slice(1);
const slugify = (s: string) => s.replace(/\s+/g, "-").replace(/[^\w-]/g, "");

const formatName = (name: string) => pipe(name, normalize, capitalize);
const createSlug = (title: string) => pipe(title, normalize, slugify);
```

**Option Type:**
```typescript
type Option<T> = { _tag: "Some"; value: T } | { _tag: "None" };

const some = <T>(value: T): Option<T> => ({ _tag: "Some", value });
const none: Option<never> = { _tag: "None" };

function map<A, B>(option: Option<A>, f: (a: A) => B): Option<B> {
  return option._tag === "Some" ? some(f(option.value)) : none;
}

function flatMap<A, B>(option: Option<A>, f: (a: A) => Option<B>): Option<B> {
  return option._tag === "Some" ? f(option.value) : none;
}

function getOrElse<T>(option: Option<T>, defaultValue: T): T {
  return option._tag === "Some" ? option.value : defaultValue;
}

// Usage
function findUser(id: string): Option<User> {
  const user = db.get(id);
  return user ? some(user) : none;
}

const userName = pipe(
  findUser("123"),
  (opt) => map(opt, (user) => user.name),
  (opt) => getOrElse(opt, "Unknown"),
);
```

**Result/Either for Error Handling:**
```typescript
type Result<T, E> =
  | { _tag: "Ok"; value: T }
  | { _tag: "Err"; error: E };

const ok = <T>(value: T): Result<T, never> => ({ _tag: "Ok", value });
const err = <E>(error: E): Result<never, E> => ({ _tag: "Err", error });

function mapResult<T, U, E>(result: Result<T, E>, f: (t: T) => U): Result<U, E> {
  return result._tag === "Ok" ? ok(f(result.value)) : result;
}

function flatMapResult<T, U, E>(result: Result<T, E>, f: (t: T) => Result<U, E>): Result<U, E> {
  return result._tag === "Ok" ? f(result.value) : result;
}

// Pipeline with error handling
function processRegistration(input: unknown): Result<User, RegistrationError> {
  return pipe(
    validateInput(input),                           // Result<RegInput, ValidationErr>
    (r) => flatMapResult(r, checkEmailAvailable),   // Result<RegInput, ValidationErr | ConflictErr>
    (r) => flatMapResult(r, createUser),            // Result<User, ...>
    (r) => flatMapResult(r, sendWelcomeEmail),      // Result<User, ...>
  );
}
```

**Practical FP with Array Methods:**
```typescript
// Functional array processing (built into JS)
const orders: Order[] = fetchOrders();

const report = orders
  .filter((o) => o.status === "completed")
  .map((o) => ({
    id: o.id,
    total: o.items.reduce((sum, i) => sum + i.price * i.quantity, 0),
  }))
  .sort((a, b) => b.total - a.total)
  .slice(0, 10);

// Reduce for aggregation
const totalByCategory = orders.reduce<Record<string, number>>(
  (acc, order) => {
    const category = order.category;
    acc[category] = (acc[category] ?? 0) + order.total;
    return acc;
  },
  {},
);

// groupBy (using Object.groupBy or custom)
const byStatus = Object.groupBy(orders, (o) => o.status);
```

### Common Mistakes
1. **FP purism** — Forcing monads everywhere when imperative code is clearer
2. **Mutating "immutable" data** — Using `readonly` but mutating through references; use deep readonly
3. **Too many abstractions** — HKT (Higher-Kinded Types) in application code; keep it pragmatic
4. **Ignoring team context** — FP patterns unfamiliar to the team; train first, adopt gradually
5. **No pipe/compose** — Deeply nested function calls instead of pipelines

### Integration with Other Topics
- **Error Handling** — Result/Either for type-safe error management
- **TypeScript Type System** — Generics enable typed FP abstractions
- **React** — Hooks are functional; immutable state updates
- **Testing** — Pure functions are trivially testable
- **Design Patterns** — Strategy as function, Observer as callbacks

---

## Resources

### Essential Reading
- *Functional-Light JavaScript* — Kyle Simpson (free online)
- fp-ts documentation (gcanti.github.io/fp-ts)
- Effect documentation (effect.website)

### Online Resources
- Total TypeScript — FP patterns
- Giulio Canti — fp-ts getting started guide
- Fun Fun Function — FP in JavaScript (YouTube)

### Practice Exercises
1. Refactor a mutable data pipeline to immutable with pipe
2. Implement Option and Result types from scratch
3. Convert imperative error handling to Result-based pipeline
4. Use readonly types and `as const` to enforce immutability
5. Build a data processing pipeline with filter/map/reduce composition
