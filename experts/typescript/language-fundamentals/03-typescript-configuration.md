# TypeScript Configuration

## Profile

### What It Is
TypeScript configuration through `tsconfig.json` controls how TypeScript checks and compiles code — including strictness levels, module systems, path aliases, target environments, and project references for monorepos. Proper configuration is the foundation of a reliable TypeScript project.

### Origin and Evolution
- `tsconfig.json` from TypeScript 1.5 (2015)
- `strict` flag (2.3) — umbrella for all strict checks
- Project references (3.0) — incremental builds for monorepos
- `paths` and `baseUrl` — module path aliases
- `moduleResolution: "bundler"` (5.0) — Modern resolution for bundled apps
- `verbatimModuleSyntax` (5.0) — Explicit import type enforcement
- Current: TypeScript 5.x with simplified modern defaults

### Key Authors and References
- **TypeScript team** — tsconfig reference documentation
- **Matt Pocock** — "TSConfig Cheat Sheet"
- **Total TypeScript** — Configuration best practices

### Key Concepts at a Glance
| Option | Description |
|--------|-------------|
| `strict` | Enables all strict type-checking options |
| `target` | Output JavaScript version (ES2022, ESNext) |
| `module` | Module system (ESNext, CommonJS, NodeNext) |
| `moduleResolution` | How modules are resolved (bundler, node16, nodenext) |
| `paths` | Path aliases for imports |
| `project references` | Multi-project setup for monorepos |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Start strict, stay strict** — Enable `strict: true` from day one. Relaxing strict later is easy; adding it to an existing project is painful.
2. **Match target to runtime** — Set `target` and `lib` to match your deployment environment. Don't transpile modern syntax for modern runtimes.
3. **Module resolution matches tooling** — Use `"bundler"` for Vite/webpack apps, `"nodenext"` for Node.js packages.
4. **Path aliases for clean imports** — `@/features/orders` instead of `../../../features/orders`. Configure in tsconfig AND bundler.
5. **Project references for monorepos** — Each package has its own tsconfig. References enable incremental builds and proper boundaries.

### When to Use vs. When to Avoid

**Custom tsconfig when:**
- Every TypeScript project (always have a tsconfig.json)
- Monorepos with multiple packages
- Libraries targeting multiple environments

**Default/simple config when:**
- Frameworks that manage tsconfig (Next.js, Vite create templates)
- Single-file scripts (use `tsx` or `ts-node` without config)

---

## Frameworks and Methodologies

### 1. New Project Configuration — step-by-step
1. Create `tsconfig.json` with `strict: true`
2. Set `target` and `lib` for runtime environment
3. Set `module` and `moduleResolution` for module system
4. Configure path aliases with `paths`
5. Set `outDir` and `rootDir` for build output
6. Enable `incremental` for faster rebuilds

### 2. Monorepo Configuration — step-by-step
1. Root `tsconfig.json` with shared settings
2. Each package: `tsconfig.json` extending root
3. Add `references` to root for project references
4. Enable `composite: true` in referenced projects
5. Use `tsc --build` for incremental monorepo builds

---

## How to Apply

### Decision Checklist
- [ ] Is `strict: true` enabled?
- [ ] Is `target` appropriate for the deployment runtime?
- [ ] Is `moduleResolution` matching the build tool?
- [ ] Are path aliases configured?
- [ ] Is `noUncheckedIndexedAccess` enabled?

### Implementation Patterns

**Modern App Configuration (Vite/bundled):**
```json
{
  "compilerOptions": {
    // Strictness
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "exactOptionalPropertyTypes": true,

    // Environment
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",

    // Modules
    "module": "ESNext",
    "moduleResolution": "bundler",
    "verbatimModuleSyntax": true,
    "resolveJsonModule": true,

    // Path aliases
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/features/*": ["./src/features/*"],
      "@/shared/*": ["./src/shared/*"]
    },

    // Output
    "outDir": "dist",
    "declaration": true,
    "sourceMap": true,

    // Other
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "incremental": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**Node.js Library Configuration:**
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,

    "target": "ES2022",
    "lib": ["ES2022"],

    "module": "NodeNext",
    "moduleResolution": "nodenext",

    "outDir": "dist",
    "rootDir": "src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,

    "composite": true,
    "incremental": true,

    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

**Monorepo Configuration:**
```json
// Root tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "declaration": true,
    "declarationMap": true,
    "composite": true,
    "skipLibCheck": true
  },
  "references": [
    { "path": "packages/shared" },
    { "path": "packages/api" },
    { "path": "packages/web" }
  ]
}

// packages/shared/tsconfig.json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src/**/*"]
}

// packages/api/tsconfig.json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "references": [
    { "path": "../shared" }
  ],
  "include": ["src/**/*"]
}
```

**Key Strict Options Explained:**
```typescript
// noUncheckedIndexedAccess: true
const arr = [1, 2, 3];
const first = arr[0];  // type: number | undefined (not just number)

const obj: Record<string, string> = {};
const val = obj["key"]; // type: string | undefined

// exactOptionalPropertyTypes: true
interface Config {
  debug?: boolean;
}
const config: Config = { debug: undefined }; // Error! Must omit or set to boolean

// noImplicitOverride: true
class Base {
  greet() { return "hello"; }
}
class Derived extends Base {
  override greet() { return "hi"; } // Must use 'override' keyword
}
```

### Common Mistakes
1. **Not using strict mode** — Catching bugs only at runtime that TypeScript could catch at compile time
2. **Wrong moduleResolution** — Using `"node"` (legacy) instead of `"bundler"` or `"nodenext"`
3. **Path aliases only in tsconfig** — Must also configure in bundler (Vite, webpack) and test runner (Vitest)
4. **`skipLibCheck: false`** — Checking third-party `.d.ts` files slows compilation and may find false positives
5. **No `noUncheckedIndexedAccess`** — Array access and object index access assumed non-undefined

### Integration with Other Topics
- **Module Systems** — tsconfig controls ESM/CJS behavior
- **Build Tools** — tsc, esbuild, swc consume tsconfig
- **Monorepo** — Project references for multi-package TypeScript
- **TypeScript Type System** — Strict options affect type checking behavior
- **Path Aliases** — Cleaner imports require tsconfig + bundler config

---

## Resources

### Essential Reading
- TypeScript tsconfig reference (typescriptlang.org/tsconfig)
- Matt Pocock — "TSConfig Cheat Sheet" (totaltypescript.com)
- *Effective TypeScript* — Configuration chapters

### Online Resources
- TypeScript release notes (each version adds new options)
- Total TypeScript — tsconfig deep dive
- TypeScript project references documentation

### Practice Exercises
1. Create a strict tsconfig for a new React + Vite project
2. Set up monorepo with 3 packages using project references
3. Enable `noUncheckedIndexedAccess` and fix all resulting errors
4. Configure path aliases that work in tsconfig, Vite, and Vitest
5. Migrate from `moduleResolution: "node"` to `"bundler"`
