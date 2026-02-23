# CI/CD Pipelines

## Profile

### What It Is
Continuous Integration (CI) automatically builds and tests code on every commit. Continuous Delivery (CD) extends CI to automatically deploy to staging. Continuous Deployment goes further, automatically deploying to production. Together, CI/CD pipelines are automated workflows that build, test, and deliver software reliably and frequently.

### Origin and Evolution
- Continuous Integration coined by Grady Booch (1991), practiced in XP (Beck, 1999)
- Martin Fowler's "Continuous Integration" article (2006) formalized practices
- Jenkins (2011), Travis CI, CircleCI popularized CI/CD
- Modern: GitHub Actions, GitLab CI, cloud-native pipelines, GitOps
- Current: platform engineering provides golden-path CI/CD templates

### Key Authors and References
- **Jez Humble & David Farley** — *Continuous Delivery* (2010)
- **Martin Fowler** — "Continuous Integration" article
- **Gene Kim** — *The DevOps Handbook*, *Accelerate*
- **Nicole Forsgren** — DORA metrics research

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Pipeline | Automated sequence: build → test → deploy |
| Trunk-Based Development | Short-lived branches, frequent merges to main |
| Green Build | Pipeline passes; code is safe to deploy |
| Artifact | Immutable build output (container image, package) |
| DORA Metrics | Deployment frequency, lead time, MTTR, change failure rate |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Every commit is a release candidate** — Every push triggers the pipeline. If it passes, it's deployable.
2. **Fast feedback** — The pipeline should catch failures within minutes. Slow pipelines kill productivity and morale.
3. **Build once, deploy everywhere** — Build the artifact once, promote it through environments. Don't rebuild per environment.
4. **Automate everything** — If a human can forget to do it, automate it. Manual steps are errors waiting to happen.
5. **Pipeline as code** — Pipeline definitions live in the repository alongside the application code. Versioned, reviewed, tested.

### When to Use vs. When to Avoid
**Always use CI.** CD maturity scales with team readiness and operational confidence.

---

## Frameworks and Methodologies

### 1. Pipeline Design — step-by-step
1. **Commit stage** (<5 min): lint, format, type check, unit tests
2. **Integration stage** (<10 min): integration tests, contract tests
3. **Build stage**: build artifact (Docker image), push to registry
4. **Deploy staging**: deploy to staging, run smoke tests
5. **Acceptance stage**: E2E tests, security scan, performance tests
6. **Deploy production**: canary/blue-green deploy, health checks

### 2. DORA Metrics Optimization — step-by-step
1. Measure current DORA metrics (deployment frequency, lead time, MTTR, change failure rate)
2. Identify bottlenecks (slow tests, manual approvals, long code review)
3. Optimize pipeline speed (parallelize, cache, reduce test scope)
4. Increase deployment frequency (smaller batches, trunk-based development)
5. Reduce MTTR with automated rollback and observability
6. Track metrics continuously and set improvement targets

---

## How to Apply

### Decision Checklist
- [ ] Does every commit trigger the pipeline?
- [ ] Is the commit-to-deploy time under 15 minutes?
- [ ] Are artifacts built once and promoted through environments?
- [ ] Is the pipeline defined as code in the repository?
- [ ] Are DORA metrics measured and improving?

### Implementation Patterns

**GitHub Actions Pipeline:**
```yaml
name: CI/CD
on: [push]

jobs:
  fast-checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.12" }
      - run: pip install uv && uv sync
      - run: uv run ruff check .
      - run: uv run mypy src/
      - run: uv run pytest tests/unit/ -x --timeout=60

  integration:
    needs: fast-checks
    runs-on: ubuntu-latest
    services:
      postgres: { image: "postgres:16", env: { POSTGRES_PASSWORD: test } }
    steps:
      - uses: actions/checkout@v4
      - run: uv run pytest tests/integration/ --timeout=120

  build-and-push:
    needs: integration
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t myapp:${{ github.sha }} .
      - run: docker push registry.example.com/myapp:${{ github.sha }}

  deploy-staging:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - run: kubectl set image deployment/myapp myapp=registry.example.com/myapp:${{ github.sha }}
      - run: kubectl rollout status deployment/myapp --timeout=300s
```

### Common Mistakes
1. **Slow pipelines** — 30+ minute pipelines delay feedback and reduce deployment frequency
2. **No pipeline for PRs** — Only running CI on main; bugs merge before they're caught
3. **Environment-specific builds** — Rebuilding for each environment introduces inconsistency
4. **Manual deployment steps** — "SSH into the server and run deploy.sh" is not CD
5. **Ignoring flaky tests** — Flaky tests erode confidence in the pipeline

### Integration with Other Topics
- **Continuous Testing** — Tests are the backbone of CI/CD pipelines
- **Deployment Strategies** — CD implements blue-green, canary, rolling deploys
- **Containerization** — Docker images are the standard CI/CD artifact
- **GitOps** — Git as the single source of truth for deployments
- **Observability** — Monitoring validates deployments
- **Infrastructure as Code** — IaC changes flow through CI/CD

---

## Resources

### Essential Reading
- *Continuous Delivery* — Jez Humble & David Farley
- *Accelerate* — Forsgren, Humble, Kim (DORA research)
- *The DevOps Handbook* — Kim, Humble, Debois, Willis

### Online Resources
- DORA metrics (dora.dev)
- GitHub Actions documentation
- GitLab CI/CD documentation

### Practice Exercises
1. Set up a CI pipeline that runs lint, tests, and build on every push
2. Add a staging deployment stage with smoke tests
3. Measure and optimize pipeline execution time
4. Implement build artifact promotion (same image through environments)
5. Track DORA metrics for your team over one month
