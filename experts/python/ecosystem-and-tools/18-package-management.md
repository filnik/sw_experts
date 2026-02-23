# Package Management

## Profile

### What It Is
Python package management covers tools for installing packages, managing virtual environments, resolving dependencies, and publishing libraries. The ecosystem has converged on `uv` (from Astral) as the modern all-in-one tool, while `pip` and `poetry` remain widely used.

### Origin and Evolution
- easy_install / setuptools (2004) — first package installer
- pip (2008) — standard package installer
- virtualenv (2007) — isolated environments
- pip-tools (2012) — dependency pinning with compile/sync
- poetry (2018) — dependency management + packaging + publishing
- pipenv (2017) — Pipfile-based, Ken Reitz
- uv (2024, Astral) — Rust-based, replaces pip, pip-tools, virtualenv, poetry
- Current: uv gaining rapid adoption; pip still the default; poetry mature

### Key Authors and References
- **Charlie Marsh / Astral** — uv, ruff
- **Sébastien Eustace** — poetry creator
- **Donald Stufft** — pip maintainer
- **Python Packaging Authority (PyPA)** — Standards and specifications

### Key Concepts at a Glance
| Tool | Purpose |
|------|---------|
| uv | All-in-one: install, venv, lock, publish (Rust-based, very fast) |
| pip | Standard package installer |
| poetry | Dependency management, packaging, publishing |
| virtualenv/venv | Isolated Python environments |
| pyproject.toml | Standardized project metadata (PEP 621) |
| Lock file | Exact dependency versions for reproducibility |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Always use virtual environments** — Never install packages globally. Each project gets its own environment with its own dependencies.
2. **Lock for applications, range for libraries** — Applications pin exact versions (lock file). Libraries specify compatible ranges so they work with different dependency versions.
3. **One tool does it all** — uv handles venv creation, package installation, dependency resolution, and locking. Fewer tools = simpler workflow.
4. **pyproject.toml is the standard** — All project metadata, dependencies, and tool configuration in one file (PEP 621).
5. **Reproducible builds** — Lock files ensure the same versions are installed everywhere (dev, CI, production).

### When to Use vs. When to Avoid

**uv**: New projects, performance-sensitive CI, replacing pip+virtualenv+pip-tools
**poetry**: Projects already using it, teams comfortable with its workflow
**pip + venv**: Simple scripts, tutorials, environments where only stdlib is available

---

## Frameworks and Methodologies

### 1. Project Setup with uv — step-by-step
1. `uv init my-project` — Create project with pyproject.toml
2. `uv venv` — Create virtual environment
3. `uv add fastapi sqlalchemy` — Add dependencies
4. `uv add --dev pytest ruff mypy` — Add dev dependencies
5. `uv lock` — Generate lock file
6. `uv sync` — Install all dependencies from lock file
7. CI: `uv sync --frozen` — Install exactly from lock file (fail if outdated)

### 2. Publishing a Library — step-by-step
1. Configure build system in pyproject.toml
2. Set version, description, classifiers
3. Build: `uv build` or `python -m build`
4. Test publish: `uv publish --repository testpypi`
5. Publish: `uv publish`
6. Tag release in git

---

## How to Apply

### Decision Checklist
- [ ] Is a virtual environment used for the project?
- [ ] Is pyproject.toml the single source of dependency metadata?
- [ ] Is there a lock file for reproducible installs?
- [ ] Are dev dependencies separated from production dependencies?
- [ ] Is CI using `--frozen` to fail on lock file drift?

### Implementation Patterns

**uv Workflow:**
```bash
# Create project
uv init my-api
cd my-api

# Virtual environment (auto-created by most uv commands)
uv venv
source .venv/bin/activate

# Add dependencies
uv add fastapi uvicorn sqlalchemy
uv add --dev pytest pytest-asyncio ruff mypy

# Lock and sync
uv lock        # Generates uv.lock
uv sync        # Installs from lock file

# Run commands in the venv
uv run pytest
uv run python -m my_api.main

# Update a dependency
uv add "sqlalchemy>=2.1"
uv lock        # Re-resolves

# CI: strict mode
uv sync --frozen  # Fails if lock file is outdated
```

**pyproject.toml for Applications:**
```toml
[project]
name = "my-api"
version = "1.0.0"
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.110",
    "uvicorn[standard]>=0.27",
    "sqlalchemy[asyncio]>=2.0",
    "asyncpg>=0.29",
    "pydantic-settings>=2.1",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-asyncio>=0.23",
    "pytest-cov>=4.1",
    "ruff>=0.3",
    "mypy>=1.8",
]

[project.scripts]
my-api = "my_api.main:run"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

**pyproject.toml for Libraries:**
```toml
[project]
name = "my-lib"
version = "0.5.0"
description = "A useful library"
readme = "README.md"
license = {text = "MIT"}
requires-python = ">=3.10"
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]
# Libraries use ranges, not pins
dependencies = [
    "httpx>=0.25,<1.0",
    "pydantic>=2.0,<3.0",
]

[project.urls]
Homepage = "https://github.com/user/my-lib"
Documentation = "https://my-lib.readthedocs.io"
```

**Poetry Workflow (for existing projects):**
```bash
# Create project
poetry new my-project
cd my-project

# Add dependencies
poetry add fastapi sqlalchemy
poetry add --group dev pytest ruff

# Install
poetry install

# Run
poetry run pytest

# Publish
poetry build
poetry publish
```

**Dependency Resolution Comparison:**
```python
# Application (pinned in lock file):
# uv.lock / poetry.lock
# fastapi==0.110.0
# uvicorn==0.27.1
# sqlalchemy==2.0.25
# Every sub-dependency pinned to exact version

# Library (ranges in pyproject.toml):
# [project.dependencies]
# httpx>=0.25,<1.0    # Works with 0.25, 0.26, ..., 0.99
# pydantic>=2.0,<3.0  # Works with any Pydantic v2

# Version specifiers:
# >=1.0      — 1.0 or newer
# >=1.0,<2.0 — 1.x only (compatible range)
# ~=1.4      — >=1.4,<2.0 (compatible release)
# ==1.4.2    — exactly this version (applications only)
```

### Common Mistakes
1. **No virtual environment** — Installing globally causes dependency conflicts between projects
2. **No lock file** — `pip install -r requirements.txt` without pinned sub-dependencies; builds differ
3. **Pinning library dependencies** — Library uses `==` instead of ranges; forces users into version conflicts
4. **requirements.txt without pyproject.toml** — Duplicating dependency information; use pyproject.toml as source of truth
5. **Not separating dev dependencies** — Installing pytest and ruff in production

### Integration with Other Topics
- **Project Structure** — pyproject.toml defines project metadata and dependencies
- **CI/CD** — Lock file ensures reproducible CI builds
- **Code Quality** — Dev dependencies include ruff, mypy, pytest
- **Containerization** — `uv sync --frozen` in Dockerfile for deterministic builds
- **Python Performance** — Dependency resolution speed (uv vs pip)

---

## Resources

### Essential Reading
- uv documentation (docs.astral.sh/uv)
- Python Packaging User Guide (packaging.python.org)
- poetry documentation (python-poetry.org)

### Online Resources
- PEP 621 — Project metadata
- PEP 517/518 — Build system specification
- Real Python — Packaging tutorials

### Practice Exercises
1. Set up a project with uv: init, add dependencies, lock, sync
2. Convert a requirements.txt project to pyproject.toml + uv
3. Publish a small library to TestPyPI with uv or poetry
4. Configure CI with `uv sync --frozen` for reproducible builds
5. Compare dependency resolution time: uv vs pip vs poetry
