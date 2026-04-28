# Python Project Structure

## Profile

### What It Is
Python project structure defines how source code, tests, configuration, and dependencies are organized in a Python project. Modern Python uses `pyproject.toml` as the single configuration file, the `src` layout for packages, and build backends like hatchling or setuptools for distribution.

### Origin and Evolution
- `setup.py` era (2000s-2010s) — imperative build configuration
- `setup.cfg` (2016) — declarative configuration alongside setup.py
- PEP 517/518 (2017) — Build system abstraction, `pyproject.toml`
- PEP 621 (2021) — Standardized project metadata in pyproject.toml
- src layout advocacy — Hynek Schlawack's "Testing & Packaging" article
- Current: `pyproject.toml` only, uv/poetry for dependency management, src layout

### Key Authors and References
- **Hynek Schlawack** — src layout advocacy, Python packaging best practices
- **Brett Cannon** — Python packaging evolution
- **Python Packaging Authority (PyPA)** — packaging.python.org
- **Astral (Charlie Marsh)** — uv, ruff

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| src layout | Source code under `src/` directory; prevents accidental local imports |
| pyproject.toml | Single config file for build, tools, metadata (PEP 621) |
| Namespace packages | PEP 420; packages without `__init__.py` for large organizations |
| Build backend | hatchling, setuptools, flit — builds distributions |
| Virtual environment | Isolated Python environment per project |
| Lock file | Pinned dependency versions for reproducibility |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **`pyproject.toml` is the one config file** — Build metadata, tool configuration (ruff, mypy, pytest), and dependencies all in one file. No `setup.py`, no `setup.cfg`, no `tox.ini`.
2. **Use the src layout** — `src/mypackage/` prevents importing the local package during testing instead of the installed one. It forces proper installation.
3. **Pin dependencies for applications, range for libraries** — Applications lock exact versions (`uv.lock`). Libraries specify compatible ranges (`>=1.0,<2.0`).
4. **Separate dev dependencies** — Production dependencies in `[project.dependencies]`, dev tools in `[project.optional-dependencies]` or `[tool.uv]`.
5. **One package, one responsibility** — Each package/module should have a clear purpose. Use `__init__.py` to define the public API.

### When to Use vs. When to Avoid

**Use src layout when:**
- Building a distributable library
- Running tests against the installed package (not local source)
- Working on larger projects with CI/CD

**Use flat layout when:**
- Simple scripts or single-file projects
- Quick prototyping where structure overhead isn't worth it
- Internal tools not distributed as packages

---

## Frameworks and Methodologies

### 1. New Project Setup — step-by-step
1. Create project directory and initialize git
2. Create `pyproject.toml` with project metadata (PEP 621)
3. Create `src/package_name/` with `__init__.py`
4. Create `tests/` directory
5. Set up virtual environment with `uv venv` or `python -m venv`
6. Install project in editable mode: `uv pip install -e ".[dev]"`
7. Configure tools in `pyproject.toml` (ruff, mypy, pytest)
8. Add pre-commit hooks

### 2. Monorepo Structure — step-by-step
1. Root `pyproject.toml` for workspace configuration
2. Each package in `packages/package-name/` with its own `pyproject.toml`
3. Shared dependencies at workspace level
4. Inter-package dependencies via workspace references
5. CI builds and tests each package independently

---

## How to Apply

### Decision Checklist
- [ ] Is `pyproject.toml` the single configuration file?
- [ ] Is src layout used for distributable packages?
- [ ] Are dev dependencies separated from production dependencies?
- [ ] Is there a lock file for reproducible installs?
- [ ] Is the public API defined in `__init__.py`?

### Implementation Patterns

**Application Project Structure:**
```
my-app/
├── pyproject.toml
├── uv.lock                    # Locked dependencies
├── README.md
├── src/
│   └── my_app/
│       ├── __init__.py
│       ├── main.py            # Entry point
│       ├── config.py           # Settings (Pydantic BaseSettings)
│       ├── domain/
│       │   ├── __init__.py
│       │   ├── models.py
│       │   └── services.py
│       ├── api/
│       │   ├── __init__.py
│       │   ├── routes.py
│       │   └── dependencies.py
│       └── infrastructure/
│           ├── __init__.py
│           ├── database.py
│           └── external_api.py
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   ├── test_models.py
│   │   └── test_services.py
│   └── integration/
│       └── test_api.py
└── scripts/
    └── seed_data.py
```

**pyproject.toml:**
```toml
[project]
name = "my-app"
version = "0.1.0"
description = "My application"
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.110",
    "uvicorn>=0.27",
    "sqlalchemy>=2.0",
    "pydantic>=2.5",
    "httpx>=0.27",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-asyncio>=0.23",
    "pytest-cov>=4.1",
    "mypy>=1.8",
    "ruff>=0.3",
    "pre-commit>=3.6",
]

[project.scripts]
my-app = "my_app.main:run"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
target-version = "py312"
line-length = 99

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "B", "SIM", "RUF"]

[tool.mypy]
strict = true
warn_return_any = true

[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"
```

**Library Project Structure:**
```
my-lib/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── my_lib/
│       ├── __init__.py        # Public API exports
│       ├── py.typed            # PEP 561 marker
│       ├── core.py
│       ├── models.py
│       └── _internal.py       # Private module (underscore prefix)
├── tests/
│   └── test_core.py
└── docs/
    └── index.md
```

**`__init__.py` as Public API:**
```python
# src/my_lib/__init__.py
"""My Library — does useful things."""

from my_lib.core import process, transform
from my_lib.models import Config, Result

__all__ = ["process", "transform", "Config", "Result"]
```

### Common Mistakes
1. **No src layout** — Importing local package instead of installed one during tests
2. **Multiple config files** — `setup.py` + `setup.cfg` + `tox.ini` + `pyproject.toml`; consolidate into pyproject.toml
3. **No `__init__.py` curation** — Exposing internal modules as public API; define exports explicitly
4. **requirements.txt for everything** — Use pyproject.toml for dependencies; lock files for pinning
5. **Circular imports** — Modules importing each other; restructure with clear dependency direction

### Integration with Other Topics
- **Package Management** — uv, poetry for dependency resolution and virtual environments
- **Code Quality Tools** — Ruff, mypy configured in pyproject.toml
- **Testing** — pytest configuration in pyproject.toml; test discovery from tests/
- **CI/CD** — Build and publish pipeline for packages
- **Clean Architecture** — Project structure reflects architectural layers

---

## Resources

### Essential Reading
- Python Packaging User Guide (packaging.python.org)
- Hynek Schlawack — "Testing & Packaging" (hynek.me)
- PEP 621 — Project metadata specification

### Online Resources
- uv documentation (docs.astral.sh)
- Hatch documentation (hatch.pypa.io)
- Real Python — Python packages tutorials

### Practice Exercises
1. Create a project with src layout, pyproject.toml, and all tools configured
2. Convert a setup.py project to pyproject.toml only
3. Set up a monorepo with 2 packages using workspace references
4. Define a clean public API in `__init__.py` with `__all__`
5. Configure ruff, mypy, and pytest all in pyproject.toml
