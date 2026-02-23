# Code Quality Tools

## Profile

### What It Is
Python code quality tools automate style enforcement, bug detection, security scanning, and formatting. The modern stack centers on `ruff` (linting + formatting), `mypy`/`pyright` (type checking), `bandit` (security), and `pre-commit` for running them automatically before each commit.

### Origin and Evolution
- pylint (2001) — comprehensive but slow linter
- flake8 (2010) — pyflakes + pycodestyle + mccabe, plugin ecosystem
- black (2018) — opinionated auto-formatter ("uncompromising")
- isort (2013) — import sorting
- bandit (2015) — security-focused linter
- ruff (2023, Charlie Marsh/Astral) — Rust-based, replaces flake8 + isort + black + pyupgrade
- Current: ruff for linting + formatting, mypy/pyright for types, pre-commit for automation

### Key Authors and References
- **Charlie Marsh / Astral** — ruff creator
- **Łukasz Langa** — black creator
- **Holger Krekel** — pre-commit ecosystem
- **Python Packaging Authority** — Packaging and tooling standards

### Key Concepts at a Glance
| Tool | Purpose |
|------|---------|
| ruff | Fast linter (500+ rules) and formatter; replaces flake8, isort, black |
| mypy / pyright | Static type checking |
| bandit | Security vulnerability detection |
| pre-commit | Git hooks for running tools before commit |
| vulture | Dead code detection |
| deptry | Unused/missing dependency detection |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Automate everything** — No manual style reviews. Tools enforce style, formatting, and basic correctness. Reviewers focus on logic and design.
2. **One tool if possible** — ruff replaces flake8, isort, black, pyupgrade, and more. Fewer tools = simpler config = faster execution.
3. **Pre-commit prevents bad commits** — Run linting, formatting, and type checking before code reaches the repository. Fix issues before they exist in history.
4. **CI is the safety net** — Pre-commit catches most issues locally. CI ensures nothing slips through (forgot to install hooks).
5. **Fix, don't warn** — `ruff check --fix` auto-fixes what it can. Reduce noise; if the tool can fix it, let it.

### When to Use vs. When to Avoid

**Full quality stack when:**
- Team projects with multiple contributors
- Libraries published for others to use
- Projects with CI/CD pipelines

**Minimal tooling when:**
- Solo scripts or notebooks
- Quick prototypes
- Exploratory data analysis

---

## Frameworks and Methodologies

### 1. Quality Stack Setup — step-by-step
1. Install ruff: `uv add --dev ruff`
2. Configure ruff in `pyproject.toml` (rules, line-length, target-version)
3. Install and configure mypy or pyright
4. Set up pre-commit with `.pre-commit-config.yaml`
5. Add to CI pipeline (GitHub Actions, GitLab CI)
6. Gradually increase strictness

### 2. Adopting in Existing Projects — step-by-step
1. Run `ruff check .` to see current violations
2. Auto-fix what you can: `ruff check --fix .`
3. Format: `ruff format .`
4. Commit the bulk fix
5. Enable rules incrementally (start permissive, add strict rules)
6. Add `# noqa` only with explanation for intentional violations

---

## How to Apply

### Decision Checklist
- [ ] Is ruff configured for linting and formatting?
- [ ] Is a type checker (mypy/pyright) running in CI?
- [ ] Is pre-commit configured and running?
- [ ] Are security checks (bandit/ruff S rules) enabled?
- [ ] Is the CI pipeline failing on quality violations?

### Implementation Patterns

**pyproject.toml Configuration:**
```toml
[tool.ruff]
target-version = "py312"
line-length = 99

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "F",    # pyflakes
    "I",    # isort
    "N",    # pep8-naming
    "UP",   # pyupgrade
    "B",    # flake8-bugbear
    "SIM",  # flake8-simplify
    "S",    # bandit (security)
    "RUF",  # ruff-specific rules
    "PTH",  # pathlib
    "T20",  # flake8-print (no print statements)
    "ERA",  # eradicate (commented-out code)
]
ignore = [
    "S101",  # assert used (OK in tests)
]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["S101", "S106"]  # Allow assert and hardcoded passwords in tests

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.mypy]
strict = true
warn_return_any = true
warn_unused_configs = true
plugins = ["pydantic.mypy"]

[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false
```

**Pre-commit Configuration:**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.3.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies:
          - pydantic>=2.5
          - types-requests

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: [--maxkb=500]
      - id: no-commit-to-branch
        args: [--branch=main]
```

**CI Pipeline (GitHub Actions):**
```yaml
# .github/workflows/quality.yml
name: Code Quality
on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v1

      - name: Install dependencies
        run: uv sync --dev

      - name: Lint
        run: uv run ruff check .

      - name: Format check
        run: uv run ruff format --check .

      - name: Type check
        run: uv run mypy src/

      - name: Tests
        run: uv run pytest --cov=src --cov-report=xml

      - name: Security check
        run: uv run bandit -r src/ -c pyproject.toml
```

**Makefile for Developer Convenience:**
```makefile
.PHONY: lint format type-check test quality

lint:
	ruff check . --fix

format:
	ruff format .

type-check:
	mypy src/

test:
	pytest --cov=src -v

quality: lint format type-check test
	@echo "All quality checks passed"

install-hooks:
	pre-commit install
```

### Common Mistakes
1. **Too many tools** — Running flake8 + pylint + black + isort separately; consolidate with ruff
2. **Not auto-fixing** — Manual fix for style issues that ruff can auto-fix
3. **Ignoring type errors** — Adding `# type: ignore` everywhere instead of fixing types
4. **No pre-commit hooks** — CI catches issues after commit; pre-commit catches them before
5. **Overly strict from day one** — Enable all rules on a legacy codebase; adopt gradually

### Integration with Other Topics
- **Pythonic Code** — Ruff enforces PEP 8 and Pythonic patterns
- **Type Hints** — mypy/pyright validate type annotations
- **CI/CD** — Quality checks as CI pipeline stages
- **Testing** — Coverage reporting with pytest-cov
- **Project Structure** — All tool config in pyproject.toml

---

## Resources

### Essential Reading
- ruff documentation (docs.astral.sh/ruff)
- mypy documentation (mypy.readthedocs.io)
- pre-commit documentation (pre-commit.com)

### Online Resources
- Astral blog — ruff development updates
- Real Python — Code quality tutorials
- bandit documentation (bandit.readthedocs.io)

### Practice Exercises
1. Set up ruff with 10+ rule categories and fix all violations
2. Configure pre-commit with ruff, mypy, and standard hooks
3. Set up a GitHub Actions CI pipeline for quality checks
4. Convert a project from flake8+black+isort to ruff only
5. Enable mypy strict mode and fix all type errors in a module
