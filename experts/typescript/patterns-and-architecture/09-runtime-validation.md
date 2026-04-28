# Runtime Validation

## Profile

### What It Is
Runtime validation bridges TypeScript's compile-time type system with runtime reality. TypeScript types are erased at runtime — data from APIs, forms, and external sources arrives untyped. Libraries like Zod, Valibot, and io-ts define schemas that validate data at runtime AND generate TypeScript types from those schemas.

### Origin and Evolution
- Manual validation (pre-2018) — if/else checks, no type integration
- io-ts (2017) — Runtime validation with fp-ts integration
- Yup (2018) — Schema validation (popular with Formik)
- Zod (2020, Colin McDonnell) — TypeScript-first schema validation
- Valibot (2023) — Bundle-size optimized alternative to Zod
- Current: Zod dominates; Valibot for bundle-sensitive apps

### Key Authors and References
- **Colin McDonnell** — Zod creator
- **Fabian Hiller** — Valibot creator
- **Giulio Canti** — io-ts creator

### Key Concepts at a Glance
| Library | Key Feature |
|---------|-------------|
| Zod | TypeScript-first, chainable API, widely adopted |
| Valibot | Tiny bundle size, modular API |
| io-ts | Functional, fp-ts integration |
| ArkType | Performance-focused, type-level inference |
| Schema-first | Define schema → derive TypeScript type |
| Code-first | Define TypeScript type → create validator |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Schema is the source of truth** — Define the Zod schema; derive the TypeScript type from it. Not the other way around. `z.infer<typeof schema>` gives you the type.
2. **Validate at boundaries** — API inputs, form data, environment variables, JSON from storage. Internal code trusts validated data.
3. **Parse, don't validate** — Zod's `.parse()` returns the typed result. You get a validated AND typed value in one step.
4. **Compose schemas** — Build complex schemas from simple ones. `z.object()`, `.extend()`, `.merge()`, `.pick()`, `.omit()` for schema composition.
5. **Errors are data** — Validation errors are structured with field paths and messages. Display them in forms, return them in APIs.

### When to Use vs. When to Avoid

**Use runtime validation when:**
- Parsing API request/response bodies
- Validating form inputs
- Loading configuration from environment
- Processing any external data

**Skip validation when:**
- Internal function calls with trusted data
- Data already validated at an outer boundary
- Performance-critical hot paths (validate once, pass typed data)

---

## Frameworks and Methodologies

### 1. Schema Design — step-by-step
1. Define schemas for each input type (API request, form, config)
2. Derive TypeScript types: `type User = z.infer<typeof UserSchema>`
3. Add transforms for data normalization (trim, lowercase)
4. Add custom validations with `.refine()` or `.superRefine()`
5. Compose schemas with `.pick()`, `.omit()`, `.extend()`
6. Generate OpenAPI from schemas (zod-to-openapi)

### 2. Form Validation — step-by-step
1. Define form schema with Zod
2. Integrate with form library (react-hook-form + @hookform/resolvers/zod)
3. Display field-level errors from Zod validation
4. Share schema between client and server (monorepo)

---

## How to Apply

### Decision Checklist
- [ ] Are all external inputs validated with schemas?
- [ ] Are TypeScript types derived from schemas (not duplicated)?
- [ ] Are validation errors structured and displayable?
- [ ] Are schemas composed from smaller reusable schemas?
- [ ] Is validation happening at system boundaries only?

### Implementation Patterns

**Zod Basics:**
```typescript
import { z } from "zod";

// Define schema
const UserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(18).optional(),
  role: z.enum(["admin", "user", "viewer"]),
});

// Derive type from schema
type User = z.infer<typeof UserSchema>;
// { name: string; email: string; age?: number; role: "admin" | "user" | "viewer" }

// Parse (throws on invalid)
const user = UserSchema.parse(rawData);

// Safe parse (returns result)
const result = UserSchema.safeParse(rawData);
if (result.success) {
  const user = result.data; // typed as User
} else {
  const errors = result.error.flatten();
  // { fieldErrors: { name: ["too short"], email: ["invalid"] } }
}
```

**Schema Composition:**
```typescript
// Base schema
const BaseEntity = z.object({
  id: z.string().uuid(),
  createdAt: z.coerce.date(),
  updatedAt: z.coerce.date(),
});

// Extend for specific entities
const OrderSchema = BaseEntity.extend({
  customerId: z.string().uuid(),
  items: z.array(OrderItemSchema).min(1),
  status: z.enum(["draft", "confirmed", "shipped", "delivered"]),
  total: z.number().positive(),
});

// Input schemas (no auto-generated fields)
const CreateOrderInput = OrderSchema.omit({
  id: true,
  createdAt: true,
  updatedAt: true,
  status: true,
  total: true,
});

const UpdateOrderInput = CreateOrderInput.partial();

// Derive types
type Order = z.infer<typeof OrderSchema>;
type CreateOrderInput = z.infer<typeof CreateOrderInput>;
type UpdateOrderInput = z.infer<typeof UpdateOrderInput>;
```

**Transforms and Refinements:**
```typescript
const RegistrationSchema = z.object({
  email: z.string()
    .email()
    .transform((e) => e.toLowerCase().trim()),

  password: z.string()
    .min(8)
    .refine((p) => /[A-Z]/.test(p), "Must contain uppercase")
    .refine((p) => /[0-9]/.test(p), "Must contain a number"),

  passwordConfirm: z.string(),

  birthDate: z.string()
    .transform((s) => new Date(s))
    .refine((d) => d < new Date(), "Must be in the past"),
}).refine(
  (data) => data.password === data.passwordConfirm,
  { message: "Passwords don't match", path: ["passwordConfirm"] },
);

// Discriminated union
const NotificationSchema = z.discriminatedUnion("type", [
  z.object({ type: z.literal("email"), to: z.string().email(), subject: z.string() }),
  z.object({ type: z.literal("slack"), channel: z.string(), message: z.string() }),
  z.object({ type: z.literal("webhook"), url: z.string().url(), payload: z.record(z.unknown()) }),
]);
```

**API Integration:**
```typescript
// FastAPI-style route validation
import { z } from "zod";

const CreateOrderSchema = z.object({
  customerId: z.string().uuid(),
  items: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1),
});

// Express middleware
function validate<T extends z.ZodType>(schema: T) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({
        error: "Validation failed",
        details: result.error.flatten().fieldErrors,
      });
    }
    req.body = result.data; // Now typed
    next();
  };
}

app.post("/orders", validate(CreateOrderSchema), (req, res) => {
  const input = req.body; // type: z.infer<typeof CreateOrderSchema>
  // ...
});
```

**Environment Variables:**
```typescript
const EnvSchema = z.object({
  NODE_ENV: z.enum(["development", "production", "test"]),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  REDIS_URL: z.string().url().optional(),
});

// Validate at startup — fail fast
const env = EnvSchema.parse(process.env);

export type Env = z.infer<typeof EnvSchema>;
export { env };
```

### Common Mistakes
1. **Duplicating types and schemas** — Defining TypeScript type AND Zod schema separately; derive type from schema
2. **Validating deep in the call chain** — Validate at the boundary; internal code trusts validated data
3. **Ignoring error structure** — Using `parse()` (throws) everywhere; use `safeParse()` for user-facing validation
4. **Not using transforms** — Manual `toLowerCase()` after validation; use `.transform()` in the schema
5. **Over-validating** — Validating the same data multiple times as it passes through layers

### Integration with Other Topics
- **TypeScript Type System** — Schemas generate TypeScript types
- **Error Handling** — Validation errors as typed discriminated unions
- **React** — Form validation with react-hook-form + Zod
- **API Design** — Request validation middleware
- **tRPC** — Zod schemas define API input types

---

## Resources

### Essential Reading
- Zod documentation (zod.dev)
- Valibot documentation (valibot.dev)
- Colin McDonnell — Zod design philosophy

### Online Resources
- @hookform/resolvers — Zod integration for react-hook-form
- zod-to-openapi — Generate OpenAPI from Zod schemas
- Total TypeScript — Runtime validation patterns

### Practice Exercises
1. Define a complex API schema with nested objects and discriminated unions
2. Integrate Zod with Express/Fastify using validation middleware
3. Set up react-hook-form with Zod resolver for a registration form
4. Validate environment variables at application startup
5. Compare Zod vs Valibot bundle size and API ergonomics
