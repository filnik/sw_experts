# Build Tools and Bundlers

## Profile

### What It Is
TypeScript build tools compile, bundle, and optimize TypeScript code for different targets — browsers, Node.js, and libraries. The ecosystem includes `tsc` (type checking), `esbuild`/`swc` (fast compilation), `Vite` (dev server + build), and `tsup` (library bundling).

### Origin and Evolution
- tsc (2012) — TypeScript compiler, type checking + JS output
- webpack (2014) — Module bundler, complex configuration
- Rollup (2016) — ES module bundler for libraries
- esbuild (2020, Evan Wallace) — Go-based, 10-100x faster than webpack
- swc (2020, Donny/Vercel) — Rust-based TypeScript/JSX compiler
- Vite (2020, Evan You) — Dev server with esbuild, production with Rollup
- tsup (2021) — Zero-config library bundler (esbuild-based)
- Current: Vite for apps, tsup for libraries, tsc for type checking only

### Key Authors and References
- **Evan You** — Vite creator (also Vue.js)
- **Evan Wallace** — esbuild creator
- **Donny (강동윤)** — swc creator
- **Vercel** — Turbopack (Rust-based bundler)

### Key Concepts at a Glance
| Tool | Use Case |
|------|----------|
| tsc | Type checking (use separately from compilation) |
| Vite | App dev server + production build |
| esbuild | Fast TypeScript compilation, used by Vite |
| swc | Fast TypeScript compilation, used by Next.js |
| tsup | Library bundling (ESM + CJS output) |
| Rollup | Library bundling (tree-shaking, plugins) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Separate type checking from compilation** — Use `tsc --noEmit` for type checking. Use esbuild/swc/Vite for fast compilation. They do different jobs.
2. **Vite for applications** — Vite provides instant dev server (HMR), optimized production builds, and plugin ecosystem. Don't configure webpack for new projects.
3. **tsup for libraries** — Zero-config bundling to ESM and CJS with declaration files. Don't hand-configure Rollup for simple libraries.
4. **esbuild/swc for speed** — These tools skip type checking for speed. Run `tsc --noEmit` separately (in CI or watch mode).
5. **Don't over-bundle** — Node.js apps often don't need bundling. Use `tsx` for development, `tsc` for production compilation. Bundle only when deploying to browsers or packaging libraries.

### When to Use vs. When to Avoid

**Vite**: React/Vue/Svelte apps, any browser-targeted application
**tsup**: Publishing npm libraries with ESM + CJS support
**tsc only**: Node.js apps where bundling isn't needed
**esbuild/swc**: Custom build pipelines, replacing Babel

---

## Frameworks and Methodologies

### 1. Application Build Setup — step-by-step
1. Use Vite: `npm create vite@latest`
2. Configure `vite.config.ts` (plugins, aliases, proxy)
3. Run `tsc --noEmit` separately for type checking
4. Configure production build output
5. Add to CI: type check + build + test

### 2. Library Build Setup — step-by-step
1. Use tsup: `npm install tsup -D`
2. Configure `tsup.config.ts` (entry, format, dts)
3. Set package.json `exports` field
4. Build: ESM + CJS + declaration files
5. Verify with `arethetypeswrong`

---

## How to Apply

### Decision Checklist
- [ ] Is type checking separate from compilation (tsc --noEmit)?
- [ ] Is Vite used for application development?
- [ ] Is tsup/Rollup used for library bundling?
- [ ] Are build outputs configured in package.json exports?
- [ ] Is the build fast enough for developer experience?

### Implementation Patterns

**Vite Configuration:**
```typescript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src"),
    },
  },
  server: {
    port: 3000,
    proxy: {
      "/api": {
        target: "http://localhost:8080",
        changeOrigin: true,
      },
    },
  },
  build: {
    outDir: "dist",
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
        },
      },
    },
  },
});
```

**tsup for Libraries:**
```typescript
// tsup.config.ts
import { defineConfig } from "tsup";

export default defineConfig({
  entry: ["src/index.ts"],
  format: ["esm", "cjs"],        // Dual output
  dts: true,                      // Generate .d.ts
  sourcemap: true,
  clean: true,
  splitting: false,
  treeshake: true,
  target: "es2022",
  outDir: "dist",
});

// package.json
// {
//   "exports": {
//     ".": {
//       "types": "./dist/index.d.ts",
//       "import": "./dist/index.js",
//       "require": "./dist/index.cjs"
//     }
//   },
//   "files": ["dist"],
//   "scripts": {
//     "build": "tsup",
//     "type-check": "tsc --noEmit",
//     "dev": "tsup --watch"
//   }
// }
```

**Build Scripts Pattern:**
```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "preview": "vite preview",
    "type-check": "tsc --noEmit",
    "type-check:watch": "tsc --noEmit --watch"
  }
}
```

**Node.js App (no bundler):**
```json
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "type-check": "tsc --noEmit"
  }
}
```

**esbuild Custom Build:**
```typescript
// build.mjs
import esbuild from "esbuild";

await esbuild.build({
  entryPoints: ["src/index.ts"],
  bundle: true,
  outfile: "dist/index.js",
  platform: "node",
  target: "node20",
  format: "esm",
  sourcemap: true,
  external: ["@prisma/client"],  // Don't bundle Prisma
  minify: process.env.NODE_ENV === "production",
});
```

**Build Performance Comparison:**
```
Tool          | 1000 TS files | Type checking
--------------|---------------|-------------
tsc           | ~5-10s        | Yes
esbuild       | ~0.1-0.3s     | No
swc           | ~0.2-0.5s     | No
Vite (dev)    | instant (HMR) | No (use tsc separately)
webpack + ts  | ~10-30s       | Optional
```

### Common Mistakes
1. **tsc for compilation** — Using tsc as the build tool for apps; slow. Use Vite/esbuild for compilation, tsc for type checking.
2. **webpack for new projects** — Complex configuration; Vite does the same with minimal config
3. **No type checking in CI** — esbuild/swc skip types; must run `tsc --noEmit` separately
4. **Bundling Node.js apps** — Node.js can run TypeScript directly (tsx) or compiled JS (tsc); bundling is usually unnecessary
5. **No declaration files for libraries** — Publishing without `.d.ts`; consumers lose type safety

### Integration with Other Topics
- **TypeScript Configuration** — tsconfig controls type checking and output
- **Module Systems** — Build output format (ESM, CJS) configured in build tools
- **Monorepo** — Turborepo orchestrates builds across packages
- **CI/CD** — Build step in deployment pipeline
- **Performance** — Build speed affects developer experience

---

## Resources

### Essential Reading
- Vite documentation (vitejs.dev)
- tsup documentation (tsup.egoist.dev)
- esbuild documentation (esbuild.github.io)

### Online Resources
- swc documentation (swc.rs)
- Rollup documentation (rollupjs.org)
- Turbopack documentation (turbo.build/pack)

### Practice Exercises
1. Set up a Vite React app with path aliases and API proxy
2. Configure tsup for a library with ESM + CJS + types output
3. Compare build times: tsc vs esbuild vs swc for the same project
4. Set up separate type checking (tsc) and compilation (esbuild) scripts
5. Verify library package exports with arethetypeswrong
