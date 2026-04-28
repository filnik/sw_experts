# TypeScript Type System

## Profile

### What It Is
TypeScript's type system adds static typing to JavaScript through type annotations, type inference, unions, intersections, type narrowing, and type guards. It catches errors at compile time, provides rich IDE support, and serves as documentation — all while compiling to plain JavaScript.

### Origin and Evolution
- TypeScript 1.0 (2014, Microsoft) — Basic types, interfaces, classes
- TypeScript 2.0 (2016) — Strict null checks, discriminated unions, control flow analysis
- TypeScript 3.0 (2018) — Project references, tuple improvements
- TypeScript 4.0+ (2020+) — Template literal types, recursive types, variadic tuples
- TypeScript 5.0+ (2023+) — Decorators, const type parameters, module resolution
- Current: TypeScript 5.x with increasingly sophisticated type inference

### Key Authors and References
- **Anders Hejlsberg** — TypeScript creator (also C#, Delphi)
- **Matt Pocock** — TypeScript educator, "Total TypeScript"
- **Stefan Baumgartner** — *TypeScript Cookbook*
- **TypeScript Handbook** — Official reference

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Type Inference | TypeScript infers types from usage; explicit annotations often optional |
| Union Types | `string \| number` — value can be either type |
| Type Narrowing | Conditional checks narrow union types to specific types |
| Type Guards | Functions that narrow types: `x is string` |
| Literal Types | `"success" \| "error"` — specific values as types |
| never | Type for impossible states (exhaustive switch cases) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Let inference work** — Don't annotate what TypeScript can infer. Annotate function signatures, not every variable. `const x = 5` is already `number`.
2. **Strict mode always** — Enable `strict: true` in tsconfig. Strict null checks, no implicit any, no implicit returns. This is non-negotiable for quality.
3. **Union types over overloads** — Use `string | number` instead of overloaded function signatures. Simpler, more composable, better inference.
4. **Discriminated unions for state** — Model state machines with discriminated unions: `{status: "loading"} | {status: "success", data: T} | {status: "error", error: Error}`.
5. **Types should guide, not fight** — If you're fighting the type system, your types don't match your runtime behavior. Fix the types or the code.

### When to Use vs. When to Avoid

**Strict typing when:**
- Library public APIs
- Complex data transformations
- State machines and business logic
- Team projects

**Looser typing when:**
- Rapid prototyping (tighten later)
- Interacting with untyped third-party libraries (use `any` temporarily)
- Simple scripts where overhead isn't worth it

---

## Frameworks and Methodologies

### 1. Type-First Development — step-by-step
1. Define data types/interfaces before implementation
2. Use discriminated unions for state representation
3. Let inference handle implementation details
4. Add explicit return types to public functions
5. Use `satisfies` operator for validation without widening
6. Enable strict mode and fix all errors

### 2. Type Narrowing Mastery — step-by-step
1. Learn built-in narrowing: `typeof`, `instanceof`, `in`, `===`
2. Use discriminated unions with a `type` or `kind` field
3. Create type guard functions for complex checks
4. Use assertion functions for validation
5. Handle `never` for exhaustive checking

---

## How to Apply

### Decision Checklist
- [ ] Is `strict: true` enabled in tsconfig?
- [ ] Are function signatures explicitly typed (params + return)?
- [ ] Are discriminated unions used for state representation?
- [ ] Is type narrowing used instead of type assertions?
- [ ] Is `any` avoided (or documented when necessary)?

### Implementation Patterns

**Basic Types and Inference:**
```typescript
// Let inference work — don't over-annotate
const name = "Alice";                    // type: "Alice" (literal)
let count = 0;                           // type: number
const items = [1, 2, 3];                 // type: number[]
const config = { port: 3000, host: "localhost" }; // inferred shape

// Annotate function signatures
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// Union types
type ID = string | number;

function getUser(id: ID): User {
  // TypeScript narrows based on typeof
  if (typeof id === "string") {
    return findBySlug(id);  // id is string here
  }
  return findById(id);       // id is number here
}
```

**Discriminated Unions:**
```typescript
// Model state with discriminated unions
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: Error };

function renderOrders(state: AsyncState<Order[]>) {
  switch (state.status) {
    case "idle":
      return <EmptyState />;
    case "loading":
      return <Spinner />;
    case "success":
      return <OrderList orders={state.data} />;  // data is Order[]
    case "error":
      return <ErrorMessage error={state.error} />;  // error is Error
    // No default needed — TypeScript knows all cases are covered
  }
}

// Domain modeling with discriminated unions
type PaymentMethod =
  | { type: "credit_card"; cardNumber: string; expiry: string }
  | { type: "bank_transfer"; iban: string }
  | { type: "paypal"; email: string };

function processPayment(method: PaymentMethod): void {
  switch (method.type) {
    case "credit_card":
      chargeCard(method.cardNumber, method.expiry);
      break;
    case "bank_transfer":
      initiateTransfer(method.iban);
      break;
    case "paypal":
      chargePaypal(method.email);
      break;
  }
}
```

**Type Narrowing and Guards:**
```typescript
// Built-in narrowing
function format(value: string | number | Date): string {
  if (typeof value === "string") return value;
  if (typeof value === "number") return value.toFixed(2);
  return value.toISOString();  // narrowed to Date
}

// Custom type guard
function isUser(value: unknown): value is User {
  return (
    typeof value === "object" &&
    value !== null &&
    "id" in value &&
    "email" in value
  );
}

// Assertion function
function assertNonNull<T>(value: T | null | undefined, msg?: string): asserts value is T {
  if (value == null) {
    throw new Error(msg ?? "Expected non-null value");
  }
}

// Exhaustive check with never
function exhaustiveCheck(value: never): never {
  throw new Error(`Unhandled case: ${value}`);
}

type Shape = { kind: "circle"; radius: number } | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle": return Math.PI * shape.radius ** 2;
    case "square": return shape.side ** 2;
    default: return exhaustiveCheck(shape); // Compile error if a case is missed
  }
}
```

**`satisfies` Operator:**
```typescript
// satisfies validates type without widening
type Color = "red" | "green" | "blue";
type Theme = Record<string, Color>;

// With `as`: type is Theme (widened, loses specific keys)
const theme1 = { primary: "red", secondary: "blue" } as Theme;

// With `satisfies`: type keeps specific keys AND validates
const theme2 = {
  primary: "red",
  secondary: "blue",
} satisfies Theme;

theme2.primary;   // type: "red" (literal, not just Color)
// theme2.unknown; // Error: property doesn't exist
```

### Common Mistakes
1. **Using `any` to silence errors** — Hides bugs; use `unknown` and narrow, or fix the types
2. **Not using strict mode** — Non-strict allows implicit any and null; bugs slip through
3. **Type assertions (`as`)** — `value as User` skips validation; use type guards instead
4. **Overly complex types** — 10-line conditional types for simple problems; keep types readable
5. **Not using discriminated unions** — Modeling state with boolean flags (`isLoading`, `isError`) instead of union states

### Integration with Other Topics
- **Advanced Types** — Generics, utility types, conditional types extend the type system
- **TypeScript Configuration** — tsconfig settings control type checking behavior
- **React + TypeScript** — Typed components, hooks, and context
- **Zod / Runtime Validation** — Bridge between compile-time types and runtime validation
- **Error Handling** — Discriminated unions for Result/Either pattern

---

## Resources

### Essential Reading
- TypeScript Handbook (typescriptlang.org/docs/handbook)
- *Effective TypeScript* — Dan Vanderkam
- Matt Pocock — Total TypeScript (totaltypescript.com)

### Online Resources
- TypeScript Playground (typescriptlang.org/play)
- type-challenges (github.com/type-challenges) — Practice exercises
- TypeScript release notes for each version

### Practice Exercises
1. Enable strict mode on an existing project and fix all type errors
2. Model an order lifecycle with discriminated unions (draft → confirmed → shipped → delivered)
3. Write type guards for API response validation
4. Replace all `as` assertions with proper type narrowing
5. Use `satisfies` for configuration objects
