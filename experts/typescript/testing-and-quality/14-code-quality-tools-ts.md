# Code Quality Tools (TypeScript)

## Profile

### What It Is
TypeScript code quality tools automate style enforcement, bug detection, and formatting. The modern stack includes ESLint with typescript-eslint (linting), Prettier or Biome (formatting), and integration through pre-commit hooks and CI pipelines.

### Origin and Evolution
- TSLint (2013) — TypeScript-specific linter (deprecated 2019)
- ESLint (2013) — JavaScript linter, extensible with plugins
- typescript-eslint (2019) — ESLint plugin for TypeScript (replaced TSLint)
- Prettier (2017) — Opinionated code formatter
- ESLint Flat Config (2023) — New configuration format
- Biome (2023) — Rust-based linter + formatter (Rome successor)
- Current: ESLint flat config + typescript-eslint + Prettier, or Biome for all-in-one

### Key Authors and References
- **Nicholas C. Zakas** — ESLint creator
- **James Henry** — typescript-eslint maintainer
- **Biome team** — Rome successor
- **Prettier team** — Opinionated formatter

### Key Concepts at a Glance
| Tool | Purpose |
|------|---------|
| ESLint + typescript-eslint | Linting with type-aware rules |
| Prettier | Code formatting (style only) |
| Biome | All-in-one linter + formatter (Rust-based, fast) |
| ESLint Flat Config | New config format (eslint.config.js) |
| Pre-commit hooks | Run tools before git commit |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Lint for bugs, format for style** — ESLint catches bugs and enforces patterns. Prettier/Biome handles formatting. Don't use ESLint for style.
2. **Type-aware linting** — typescript-eslint's type-aware rules catch bugs that regular rules miss (floating promises, unsafe any usage).
3. **Flat config for simplicity** — ESLint's new flat config (eslint.config.js) is simpler and more composable than legacy .eslintrc.
4. **Automate everything** — Pre-commit hooks format and lint. CI verifies. No manual style discussions in code review.
5. **Biome as alternative** — Biome is faster (Rust), handles both linting and formatting. Consider for new projects, especially monorepos.

### When to Use vs. When to Avoid

**ESLint + Prettier**: Established stack, huge plugin ecosystem, maximum flexibility
**Biome**: Fast, simple config, all-in-one, fewer plugins available
**ESLint only**: When team doesn't want Prettier's opinionated formatting

---

## Frameworks and Methodologies

### 1. Quality Stack Setup — step-by-step
1. Install ESLint + typescript-eslint + Prettier (or Biome)
2. Create eslint.config.js (flat config)
3. Configure Prettier (.prettierrc)
4. Set up pre-commit hooks (husky + lint-staged)
5. Add to CI pipeline
6. Configure editor integration (VSCode)

### 2. Biome Setup — step-by-step
1. Install: `npm install --save-dev @biomejs/biome`
2. Initialize: `npx biome init`
3. Configure `biome.json`
4. Replace ESLint + Prettier
5. Integrate with pre-commit and CI

---

## How to Apply

### Decision Checklist
- [ ] Is type-aware linting enabled (typescript-eslint)?
- [ ] Is formatting automated (Prettier or Biome)?
- [ ] Are pre-commit hooks configured?
- [ ] Does CI fail on lint/format violations?
- [ ] Are rules appropriate (not too strict, not too lenient)?

### Implementation Patterns

**ESLint Flat Config:**
```javascript
// eslint.config.js
import eslint from "@eslint/js";
import tseslint from "typescript-eslint";
import prettier from "eslint-config-prettier";

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,
  prettier,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      // Type-aware rules
      "@typescript-eslint/no-floating-promises": "error",
      "@typescript-eslint/no-misused-promises": "error",
      "@typescript-eslint/no-unsafe-assignment": "error",
      "@typescript-eslint/no-unsafe-return": "error",

      // Best practices
      "@typescript-eslint/no-unused-vars": ["error", {
        argsIgnorePattern: "^_",
        varsIgnorePattern: "^_",
      }],
      "@typescript-eslint/explicit-function-return-type": ["warn", {
        allowExpressions: true,
      }],
      "@typescript-eslint/consistent-type-imports": ["error", {
        prefer: "type-imports",
      }],

      // Disabled rules
      "@typescript-eslint/no-explicit-any": "warn",
    },
  },
  {
    files: ["**/*.test.ts", "**/*.spec.ts"],
    rules: {
      "@typescript-eslint/no-unsafe-assignment": "off",
      "@typescript-eslint/no-explicit-any": "off",
    },
  },
  {
    ignores: ["dist/", "node_modules/", "coverage/"],
  },
);
```

**Prettier Configuration:**
```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": false,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always"
}
```

**Biome Configuration:**
```json
{
  "$schema": "https://biomejs.dev/schemas/1.5.0/schema.json",
  "organizeImports": {
    "enabled": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedImports": "error",
        "noUnusedVariables": "warn"
      },
      "suspicious": {
        "noExplicitAny": "warn"
      },
      "style": {
        "useConst": "error",
        "noNonNullAssertion": "warn"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "files": {
    "ignore": ["dist/", "node_modules/", "coverage/"]
  }
}
```

**Pre-commit Hooks (husky + lint-staged):**
```json
// package.json
{
  "scripts": {
    "prepare": "husky"
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md,yml}": [
      "prettier --write"
    ]
  }
}
```

```bash
# .husky/pre-commit
npx lint-staged
```

**CI Pipeline:**
```yaml
# .github/workflows/quality.yml
name: Quality
on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci

      - name: Type Check
        run: npx tsc --noEmit

      - name: Lint
        run: npx eslint .

      - name: Format Check
        run: npx prettier --check .

      - name: Test
        run: npx vitest run --coverage
```

### Common Mistakes
1. **No type-aware rules** — Using ESLint without typescript-eslint type checking; misses important bugs
2. **ESLint for formatting** — ESLint formatting rules conflict with Prettier; use eslint-config-prettier
3. **Legacy .eslintrc** — Old config format; migrate to flat config (eslint.config.js)
4. **No pre-commit hooks** — CI catches issues after commit; pre-commit catches them before
5. **Too strict initially** — Enabling all rules on an existing codebase; adopt gradually

### Integration with Other Topics
- **TypeScript Configuration** — ESLint reads tsconfig for type-aware rules
- **CI/CD** — Quality checks as pipeline stages
- **Testing** — Linting test files with relaxed rules
- **Monorepo** — Shared ESLint config package
- **Code Review** — Automated quality checks reduce manual review burden

---

## Resources

### Essential Reading
- typescript-eslint documentation (typescript-eslint.io)
- ESLint Flat Config documentation
- Biome documentation (biomejs.dev)

### Online Resources
- Prettier documentation (prettier.io)
- ESLint plugin directory
- Josh Goldberg — "Learning TypeScript" (linting chapters)

### Practice Exercises
1. Set up ESLint flat config with typescript-eslint strict type-checked rules
2. Configure Biome as ESLint + Prettier replacement and compare
3. Set up husky + lint-staged for pre-commit quality checks
4. Create a shared ESLint config package for a monorepo
5. Enable type-aware rules and fix all violations in a project
