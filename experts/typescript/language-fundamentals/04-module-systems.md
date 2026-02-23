# Module Systems

## Profile

### What It Is
JavaScript/TypeScript module systems define how code is organized, imported, and exported. The ecosystem has two main systems: ESM (ECMAScript Modules, the standard) and CJS (CommonJS, Node.js legacy). Understanding both, and how they interact, is essential for building libraries, applications, and handling the ongoing migration from CJS to ESM.

### Origin and Evolution
- No modules (pre-2009) — Global variables, script tags, IIFE pattern
- CommonJS (2009) — `require()`/`module.exports` for Node.js
- AMD/UMD (2010s) — Asynchronous module definition for browsers
- ES Modules (ES2015) — `import`/`export` standard
- Node.js ESM support (2019, stable 2021) — `.mjs` extension, `"type": "module"`
- Current: ESM is the standard; CJS is legacy but still widely used in Node.js ecosystem

### Key Authors and References
- **TC39** — ECMAScript modules specification
- **Node.js team** — ESM implementation in Node.js
- **Sindre Sorhus** — ESM-only package advocacy
- **Andrew Branch** — "Are the types wrong?" (arethetypeswrong.github.io)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| ESM | `import`/`export`, static analysis, tree-shakeable, browser-native |
| CJS | `require()`/`module.exports`, dynamic, synchronous, Node.js default |
| Barrel exports | `index.ts` that re-exports from submodules |
| Tree shaking | Dead code elimination based on static imports |
| Dynamic import | `import("./module")` returns a Promise (code splitting) |
| Dual package | Library that supports both ESM and CJS consumers |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **ESM is the standard** — New code should use ESM. CJS is legacy. Import/export is statically analyzable, tree-shakeable, and browser-compatible.
2. **Named exports over default exports** — Named exports enable auto-import, refactoring, and tree shaking. Default exports require the consumer to name the import.
3. **Barrel files with caution** — `index.ts` re-exports are convenient but can break tree shaking and slow down bundling if they re-export too much.
4. **Dynamic imports for code splitting** — Use `import()` to split bundles at route boundaries. Don't load what the user doesn't need yet.
5. **Understand the CJS-ESM boundary** — ESM can import CJS. CJS cannot `require()` ESM (only dynamic `import()`). Libraries need to handle this for dual-package support.

### When to Use vs. When to Avoid

**ESM always** for new projects, libraries, and applications.
**CJS** only when targeting legacy Node.js environments or when dependencies require it.
**Dual package** when publishing a library consumed by both ESM and CJS users.

---

## Frameworks and Methodologies

### 1. ESM Migration — step-by-step
1. Add `"type": "module"` to package.json
2. Change `require()` to `import`, `module.exports` to `export`
3. Add file extensions to relative imports (`.js`, not `.ts` in output)
4. Update tsconfig: `module: "nodenext"`, `moduleResolution: "nodenext"`
5. Update tools (Jest → Vitest, ts-node → tsx)
6. Test all import paths

### 2. Library Publishing — step-by-step
1. Decide: ESM-only (recommended) or dual CJS+ESM
2. Configure `exports` field in package.json
3. Set `types` field for TypeScript declarations
4. Build with tsc, tsup, or unbuild
5. Verify with `arethetypeswrong.github.io`
6. Test in both ESM and CJS consumers

---

## How to Apply

### Decision Checklist
- [ ] Is the project using ESM (`"type": "module"` or ESM build output)?
- [ ] Are named exports preferred over default exports?
- [ ] Are barrel files limited to public API boundaries?
- [ ] Is dynamic import used for code splitting?
- [ ] Are library exports properly configured in package.json?

### Implementation Patterns

**ESM Import/Export:**
```typescript
// Named exports (preferred)
export function createOrder(items: CartItem[]): Order {
  // ...
}
export type { Order, CartItem };

// Import
import { createOrder } from "./order-service.js";
import type { Order } from "./types.js";  // Type-only import

// Namespace import
import * as orderService from "./order-service.js";
orderService.createOrder([]);

// Re-export (barrel file)
// features/orders/index.ts
export { createOrder, cancelOrder } from "./order-service.js";
export { OrderList, OrderDetail } from "./components.js";
export type { Order, OrderItem } from "./types.js";
```

**Dynamic Imports (Code Splitting):**
```typescript
// Route-based code splitting in React
import { lazy, Suspense } from "react";

const OrderPage = lazy(() => import("./pages/OrderPage.js"));
const AdminPage = lazy(() => import("./pages/AdminPage.js"));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/orders" element={<OrderPage />} />
        <Route path="/admin" element={<AdminPage />} />
      </Routes>
    </Suspense>
  );
}

// Conditional dynamic import
async function loadAnalytics() {
  if (shouldTrack()) {
    const { init } = await import("./analytics.js");
    init();
  }
}
```

**Library Package.json Exports:**
```json
{
  "name": "my-lib",
  "version": "1.0.0",
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.js",
      "require": "./dist/utils.cjs"
    }
  },
  "files": ["dist"],
  "scripts": {
    "build": "tsup src/index.ts src/utils.ts --format esm,cjs --dts"
  }
}
```

**Tree Shaking Optimization:**
```typescript
// BAD: barrel file that imports everything
// utils/index.ts
export * from "./string-utils.js";   // 50 functions
export * from "./date-utils.js";     // 30 functions
export * from "./math-utils.js";     // 20 functions
// Consumer imports 1 function but bundler may include all 100

// GOOD: direct imports
import { formatDate } from "./utils/date-utils.js";

// GOOD: package.json sideEffects for tree shaking
{
  "sideEffects": false  // Tells bundler all modules are pure
}

// Mark specific files as having side effects
{
  "sideEffects": ["./src/polyfills.js", "*.css"]
}
```

**verbatimModuleSyntax:**
```typescript
// With verbatimModuleSyntax: true (recommended)

// Must use 'import type' for type-only imports
import type { User } from "./types.js";     // Removed at compile time
import { createUser } from "./service.js";   // Kept at runtime

// Mixed: use inline type
import { createUser, type User } from "./service.js";

// Export type must be explicit
export type { User };
export { createUser };
```

### Common Mistakes
1. **Missing file extensions** — ESM requires `.js` in imports (even for `.ts` source files in Node.js)
2. **Giant barrel files** — Re-exporting everything breaks tree shaking; export only the public API
3. **Default export for everything** — Harder to auto-import and refactor; use named exports
4. **CJS in ESM project** — `require()` doesn't work in ESM; use `import` or `createRequire`
5. **No `exports` field in libraries** — Consumers can import internal files; use `exports` to control public API

### Integration with Other Topics
- **TypeScript Configuration** — `module` and `moduleResolution` control module behavior
- **Build Tools** — Bundlers (Vite, esbuild, webpack) handle module resolution and tree shaking
- **Monorepo** — Internal packages need proper module configuration
- **React** — Lazy loading uses dynamic imports for code splitting
- **Node.js** — ESM support in Node requires specific configuration

---

## Resources

### Essential Reading
- Node.js ESM documentation (nodejs.org/api/esm.html)
- TypeScript Module Theory (typescriptlang.org/docs/handbook/modules)
- Sindre Sorhus — Pure ESM package guide

### Online Resources
- arethetypeswrong.github.io — Verify package type correctness
- TypeScript module resolution docs
- "Dual package hazard" Node.js documentation

### Practice Exercises
1. Migrate a CJS Node.js project to ESM
2. Configure a library with dual CJS/ESM output using tsup
3. Set up code splitting with dynamic imports in a React app
4. Verify a library's type exports with arethetypeswrong
5. Optimize barrel files to preserve tree shaking
