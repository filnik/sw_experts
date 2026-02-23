# Advanced Types

## Profile

### What It Is
TypeScript's advanced type system includes generics, utility types, template literal types, conditional types, mapped types, and infer. These features enable type-level programming — creating types that adapt to usage, enforce constraints, and derive new types from existing ones.

### Origin and Evolution
- Generics from TypeScript 1.0 — type parameters for functions and classes
- Mapped types (2.1) — `{[K in keyof T]: ...}`
- Conditional types (2.8) — `T extends U ? X : Y`
- Template literal types (4.1) — `\`${string}Changed\``
- `satisfies` operator (4.9) — Type validation without widening
- `const` type parameters (5.0) — Infer literal types in generics
- Current: TypeScript's type system is Turing-complete (use responsibly)

### Key Authors and References
- **Anders Hejlsberg** — TypeScript creator, type system designer
- **Matt Pocock** — "Total TypeScript" (advanced type patterns)
- **Stefan Baumgartner** — *TypeScript Cookbook* (2023)
- **type-challenges** — Community exercises for type-level programming

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Generics | `<T>` — Type parameters for reusable typed functions/classes |
| Utility Types | `Partial<T>`, `Pick<T>`, `Omit<T>`, `Record<K,V>`, `Required<T>` |
| Mapped Types | Transform each property: `{[K in keyof T]: NewType}` |
| Conditional Types | `T extends U ? X : Y` — type-level if/else |
| Template Literals | `\`on${Capitalize<string>}\`` — string pattern types |
| `infer` | Extract types within conditional types |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Generics for reusable abstractions** — Use generics when a function or class works with multiple types while preserving type relationships.
2. **Utility types before custom types** — `Partial`, `Pick`, `Omit`, `Record` cover most type transformations. Learn them before writing custom mapped types.
3. **Constrained generics** — Use `extends` to constrain generic parameters: `<T extends { id: string }>`. Don't accept wider types than needed.
4. **Keep types readable** — Complex conditional types are hard to maintain. Extract into named types. If a type needs a comment to explain, simplify it.
5. **Type-level programming is a last resort** — Custom conditional and mapped types are powerful but hard to debug. Use them for library code, not application code.

### When to Use vs. When to Avoid

**Use advanced types when:**
- Building reusable libraries with type-safe APIs
- Deriving types from existing ones (API response → form state)
- Enforcing constraints at the type level (branded types, exact shapes)

**Keep it simple when:**
- Application code where explicit types are clearer
- Small team unfamiliar with advanced patterns
- Types become unreadable or unmaintainable

---

## Frameworks and Methodologies

### 1. Generic Design — step-by-step
1. Start without generics; use specific types
2. When you duplicate for different types, introduce `<T>`
3. Add constraints with `extends` to limit acceptable types
4. Use multiple type parameters when types relate to each other
5. Provide defaults: `<T = unknown>` for optional generics
6. Test inference: the caller shouldn't need to specify `<T>` manually

### 2. Type Derivation — step-by-step
1. Define the source type (API response, database model)
2. Use utility types to derive variants: `Partial<T>`, `Omit<T, "id">`
3. Use `Pick<T>` for subsets
4. Use mapped types for transformations
5. Export derived types alongside source types

---

## How to Apply

### Decision Checklist
- [ ] Are generics used for type-safe reusable functions?
- [ ] Are utility types used before custom type manipulations?
- [ ] Are generic constraints (`extends`) applied appropriately?
- [ ] Are complex types extracted into named type aliases?
- [ ] Is type inference verified (callers don't need `<T>` manually)?

### Implementation Patterns

**Generics:**
```typescript
// Basic generic function
function first<T>(items: T[]): T | undefined {
  return items[0];
}
const n = first([1, 2, 3]);    // type: number | undefined
const s = first(["a", "b"]);   // type: string | undefined

// Constrained generics
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30 };
const name = getProperty(user, "name");  // type: string
// getProperty(user, "email");           // Error: "email" not in keyof user

// Generic class
class TypedMap<K extends string, V> {
  private map = new Map<K, V>();

  set(key: K, value: V): void {
    this.map.set(key, value);
  }

  get(key: K): V | undefined {
    return this.map.get(key);
  }
}

// Multiple generics with constraint relationships
function merge<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}
```

**Utility Types:**
```typescript
interface User {
  id: string;
  name: string;
  email: string;
  role: "admin" | "user";
  createdAt: Date;
}

// Derive types from User
type CreateUserInput = Omit<User, "id" | "createdAt">;
type UpdateUserInput = Partial<Omit<User, "id" | "createdAt">>;
type UserSummary = Pick<User, "id" | "name" | "role">;
type UserRecord = Record<string, User>;

// Required: make optional properties required
type StrictConfig = Required<Partial<Config>>;

// Readonly: immutable version
type FrozenUser = Readonly<User>;

// ReturnType: extract function return type
type ApiResponse = ReturnType<typeof fetchUsers>;

// Parameters: extract function parameter types
type FetchParams = Parameters<typeof fetchUsers>;
```

**Mapped Types:**
```typescript
// Make all properties optional and nullable
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

// Make all properties readonly deeply
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

// Transform property types
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// { getName: () => string; getEmail: () => string; ... }
```

**Conditional Types:**
```typescript
// Basic conditional
type IsString<T> = T extends string ? true : false;
type A = IsString<"hello">;  // true
type B = IsString<42>;       // false

// Extract with infer
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;
type Result = UnwrapPromise<Promise<User>>;  // User
type Plain = UnwrapPromise<string>;          // string

// Array element type
type ElementOf<T> = T extends (infer E)[] ? E : never;
type Item = ElementOf<string[]>;  // string

// Function return type extraction
type ReturnOf<T> = T extends (...args: any[]) => infer R ? R : never;
```

**Template Literal Types:**
```typescript
// Event handler types
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

// Route parameters
type ExtractParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractParams<Rest>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type Params = ExtractParams<"/users/:userId/posts/:postId">;
// "userId" | "postId"
```

**Branded Types:**
```typescript
// Branded types for type-safe identifiers
type Brand<T, B extends string> = T & { readonly __brand: B };

type UserId = Brand<string, "UserId">;
type OrderId = Brand<string, "OrderId">;

function createUserId(id: string): UserId {
  return id as UserId;
}

function getUser(id: UserId): User { ... }

const userId = createUserId("u123");
const orderId = "o456" as OrderId;

getUser(userId);   // OK
// getUser(orderId); // Error: OrderId is not assignable to UserId
// getUser("raw");   // Error: string is not assignable to UserId
```

### Common Mistakes
1. **Over-engineered types** — 20-line conditional types for simple problems; readability matters
2. **No generic constraints** — `<T>` without `extends`; accepts types that will fail at runtime
3. **Forgetting `as const`** — Array literals widened to `string[]` instead of literal tuple types
4. **Not using built-in utility types** — Reimplementing `Partial`, `Pick`, `Omit` manually
5. **Type-level recursion depth** — Deeply recursive types hit TypeScript's recursion limit

### Integration with Other Topics
- **TypeScript Type System** — Advanced types build on the foundation
- **React + TypeScript** — Generic components, typed hooks
- **Zod** — Schema types that generate TypeScript types
- **tRPC** — Type-safe API layer using advanced generics
- **Design Patterns** — Typed Factory, Strategy, Builder patterns

---

## Resources

### Essential Reading
- *Effective TypeScript* — Dan Vanderkam
- *TypeScript Cookbook* — Stefan Baumgartner
- Matt Pocock — Total TypeScript (totaltypescript.com)

### Online Resources
- type-challenges (github.com/type-challenges)
- TypeScript Handbook — Generics, Conditional Types sections
- TypeScript Playground for experimentation

### Practice Exercises
1. Create a generic `Result<T, E>` type with `ok()` and `err()` constructors
2. Build a type-safe event emitter with mapped types
3. Implement `DeepPartial<T>` with recursive mapped types
4. Extract route parameters from a URL pattern using template literal types
5. Create branded ID types for different entities (UserId, OrderId)
