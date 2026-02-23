# Cloud-Native Architecture (12-Factor)

## Profile

### What It Is
Cloud-Native Architecture is an approach to building and running applications that fully exploits cloud computing advantages — elasticity, resilience, and manageability. The 12-Factor App methodology provides a set of principles for building SaaS applications that are portable, scalable, and suitable for modern cloud platforms.

### Origin and Evolution
- 12-Factor App methodology by Adam Wiggins (Heroku, 2011)
- Cloud Native Computing Foundation (CNCF) established 2015
- Containerization (Docker) and orchestration (Kubernetes) became the runtime standard
- Beyond 12 factors: 15-factor methodology adds API-first, telemetry, auth/authz
- Current: cloud-native is the default architecture for new applications

### Key Authors and References
- **Adam Wiggins** — 12-Factor App methodology
- **CNCF** — Cloud Native definition and landscape
- **Cornelia Davis** — *Cloud Native Patterns*
- **Kelsey Hightower** — Kubernetes and cloud-native evangelism

### Key Concepts at a Glance
| Factor | Principle |
|--------|-----------|
| Codebase | One codebase tracked in version control |
| Dependencies | Explicitly declare and isolate dependencies |
| Config | Store config in the environment |
| Backing Services | Treat backing services as attached resources |
| Build, Release, Run | Strictly separate build and run stages |
| Processes | Execute the app as stateless processes |
| Port Binding | Export services via port binding |
| Concurrency | Scale out via the process model |
| Disposability | Fast startup, graceful shutdown |
| Dev/Prod Parity | Keep development and production as similar as possible |
| Logs | Treat logs as event streams |
| Admin Processes | Run admin tasks as one-off processes |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Design for failure** — Cloud infrastructure is inherently unreliable. Instances can be terminated, networks can partition. Applications must handle this gracefully.
2. **Stateless and disposable** — Application instances should be stateless (state in external services) and disposable (start fast, stop gracefully).
3. **Configuration from environment** — No hardcoded configuration. Everything environment-specific comes from environment variables or external config services.
4. **Observable by default** — Structured logging, health checks, metrics, and distributed tracing are not afterthoughts; they're built in from the start.
5. **Automate everything** — Build, test, deploy, scale, recover — all automated. Manual processes are errors waiting to happen.

### When to Use vs. When to Avoid

**Use when:**
- Deploying to cloud platforms (AWS, GCP, Azure)
- Microservices or distributed architecture
- Need for auto-scaling and high availability
- Multiple environments (dev, staging, production)
- CI/CD and DevOps practices are in place

**Avoid when:**
- Deploying to a single on-premises server
- Legacy application that can't be refactored
- Small scripts or utilities
- Extreme low-latency requirements where cloud adds unacceptable overhead

---

## Frameworks and Methodologies

### 1. 12-Factor Compliance Audit — step-by-step
1. Audit each factor against the current application
2. Score: compliant, partially compliant, non-compliant
3. Prioritize: config externalization and statelessness first
4. Implement changes incrementally (one factor at a time)
5. Add health checks and structured logging early
6. Validate with container deployment (Docker)

### 2. Cloud-Native Migration — step-by-step
1. Containerize the application (Dockerfile, multi-stage build)
2. Externalize configuration (env vars, config maps)
3. Move state to external services (database, cache, object storage)
4. Add health check endpoints (liveness, readiness)
5. Implement structured logging (JSON to stdout)
6. Deploy to Kubernetes with proper resource limits
7. Add auto-scaling policies (HPA)

---

## How to Apply

### Decision Checklist
- [ ] Is configuration externalized (env vars, config maps)?
- [ ] Are processes stateless (state in external stores)?
- [ ] Are health checks implemented (liveness and readiness)?
- [ ] Are logs structured and sent to stdout?
- [ ] Can instances start quickly and shut down gracefully?
- [ ] Is the build/deploy pipeline fully automated?

### Implementation Patterns

**Configuration from Environment:**
```python
from pydantic_settings import BaseSettings

class AppConfig(BaseSettings):
    database_url: str
    redis_url: str
    log_level: str = "INFO"
    max_workers: int = 4
    api_key: str  # Sensitive — from env/secret

    class Config:
        env_prefix = "APP_"

config = AppConfig()  # Reads APP_DATABASE_URL, APP_REDIS_URL, etc.
```

**Health Check Endpoints:**
```python
@app.get("/health/live")
def liveness():
    """Is the process running? Restart if not."""
    return {"status": "alive"}

@app.get("/health/ready")
def readiness():
    """Can the process accept traffic?"""
    checks = {
        "database": check_database(),
        "cache": check_redis(),
    }
    healthy = all(checks.values())
    status_code = 200 if healthy else 503
    return JSONResponse(
        status_code=status_code,
        content={"status": "ready" if healthy else "not_ready", "checks": checks}
    )
```

**Graceful Shutdown:**
```python
import signal
import asyncio

class GracefulShutdown:
    def __init__(self):
        self.shutting_down = False
        signal.signal(signal.SIGTERM, self._handle_sigterm)

    def _handle_sigterm(self, signum, frame):
        self.shutting_down = True

    async def shutdown(self, app):
        # Stop accepting new requests
        # Wait for in-flight requests to complete
        # Close database connections
        # Close message consumers
        await app.db_pool.close()
        await app.redis.close()
```

**Structured Logging:**
```python
import structlog

logger = structlog.get_logger()

# Outputs JSON to stdout — collected by log aggregator
logger.info("order_placed",
    order_id="order_123",
    customer_id="cust_456",
    total=99.99,
    items_count=3
)
# {"event": "order_placed", "order_id": "order_123", "customer_id": "cust_456", ...}
```

### Common Mistakes
1. **Storing state in the filesystem** — Uploaded files, sessions, temp data on local disk are lost when the instance restarts
2. **Hardcoded configuration** — Database URLs, API keys in code instead of environment variables
3. **No health checks** — Kubernetes can't determine if the app is healthy; broken instances keep receiving traffic
4. **Slow startup** — Loading large models or caches synchronously at startup delays scaling and recovery
5. **Log files instead of streams** — Writing logs to files instead of stdout makes log aggregation difficult in containers
6. **Ignoring graceful shutdown** — SIGTERM causes immediate exit, dropping in-flight requests

### Integration with Other Topics
- **Containerization and Orchestration** — Docker + Kubernetes are the runtime for cloud-native apps
- **CI/CD Pipelines** — Automated build/deploy pipelines are essential
- **Observability** — Cloud-native apps must be observable (logs, metrics, traces)
- **Microservices** — Microservices are a cloud-native architectural style
- **Serverless** — Serverless functions take cloud-native principles to the extreme
- **Infrastructure as Code** — Cloud infrastructure is defined and managed as code

---

## Resources

### Essential Reading
- *Cloud Native Patterns* — Cornelia Davis
- 12factor.net — The 12-Factor App methodology
- *Kubernetes in Action* — Marko Luksa

### Online Resources
- CNCF Cloud Native Landscape (landscape.cncf.io)
- CNCF Cloud Native Trail Map
- Kelsey Hightower's "Kubernetes the Hard Way"

### Practice Exercises
1. Audit an existing application against the 12 factors
2. Containerize an application with a multi-stage Dockerfile
3. Externalize all configuration to environment variables
4. Add liveness and readiness health check endpoints
5. Implement structured JSON logging to stdout
