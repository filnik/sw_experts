# Observability and Monitoring

## Profile

### What It Is
Observability is the ability to understand a system's internal state from its external outputs. It relies on three pillars — logs, metrics, and traces — to provide comprehensive visibility into system behavior. Monitoring uses this data to detect, alert, and diagnose issues in production systems.

### Origin and Evolution
- Traditional monitoring: Nagios, Zabbix (health checks and alerts)
- Three pillars formalized: logs (ELK), metrics (Prometheus), traces (Jaeger/Zipkin)
- OpenTelemetry (2019) — unified observability standard for all three pillars
- Modern: structured logging, high-cardinality metrics, distributed tracing
- Current: observability-driven development, SLO-based alerting, eBPF-based observability

### Key Authors and References
- **Charity Majors** — *Observability Engineering* (O'Reilly, 2022)
- **Cindy Sridharan** — *Distributed Systems Observability* (2018)
- **Google SRE Team** — Monitoring chapters in SRE book
- **OpenTelemetry** — CNCF observability standard

### Key Concepts at a Glance
| Pillar | What | Tool Examples |
|--------|------|---------------|
| Logs | Discrete events with context | ELK, Loki, CloudWatch Logs |
| Metrics | Numeric measurements over time | Prometheus, Datadog, CloudWatch |
| Traces | Request flow across services | Jaeger, Zipkin, X-Ray |
| SLI/SLO | Service Level Indicators/Objectives | Measured via metrics |
| Alerting | Notification when SLOs are at risk | PagerDuty, OpsGenie |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Observe, don't just monitor** — Monitoring tells you when something is broken. Observability tells you why and helps you ask questions you didn't know to ask.
2. **Three pillars, one picture** — Logs, metrics, and traces are complementary. Correlate them (trace ID in logs and metrics) for full understanding.
3. **SLO-based alerting** — Alert on user impact (SLO breach), not internal metrics. Users don't care about CPU usage; they care about latency and errors.
4. **Structured, not grep-able** — Structured logs (JSON with key-value pairs) enable queries. Unstructured text requires regex and guessing.
5. **Instrument everything, alert selectively** — Collect data broadly for debugging but alert narrowly to avoid alert fatigue.

### When to Use vs. When to Avoid

**Full observability stack:**
- Distributed systems / microservices
- Production systems with SLA requirements
- Systems where debugging requires cross-service correlation

**Simpler monitoring:**
- Single-service applications
- Development/staging environments
- When operational complexity needs to be minimized

---

## Frameworks and Methodologies

### 1. Observability Implementation — step-by-step
1. Define SLIs for each service (latency, error rate, throughput)
2. Set SLOs for each SLI (99.9% of requests < 200ms)
3. Instrument code with OpenTelemetry (metrics, traces, logs)
4. Set up log aggregation (structured JSON logs to central store)
5. Set up metrics collection and dashboards
6. Set up distributed tracing with trace ID propagation
7. Configure alerts on SLO error budgets

### 2. RED/USE Method — step-by-step
**RED (for services):** Rate (requests/sec), Errors (failed requests/sec), Duration (latency)
**USE (for resources):** Utilization, Saturation, Errors (per resource: CPU, memory, disk, network)

---

## How to Apply

### Decision Checklist
- [ ] Are SLIs and SLOs defined for each service?
- [ ] Are logs structured (JSON) with correlation IDs?
- [ ] Are metrics collected for RED (services) and USE (resources)?
- [ ] Is distributed tracing propagated across service boundaries?
- [ ] Are alerts based on SLO breaches, not arbitrary thresholds?

### Implementation Patterns

**OpenTelemetry Instrumentation (Python):**
```python
from opentelemetry import trace, metrics
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor

tracer = trace.get_tracer("order-service")
meter = metrics.get_meter("order-service")

order_counter = meter.create_counter("orders.placed", description="Orders placed")
order_duration = meter.create_histogram("orders.duration_ms", description="Order processing time")

@app.post("/orders")
async def create_order(request: CreateOrderRequest):
    with tracer.start_as_current_span("create_order") as span:
        span.set_attribute("customer_id", request.customer_id)
        start = time.time()
        order = await order_service.create(request)
        duration = (time.time() - start) * 1000
        order_counter.add(1, {"status": "success"})
        order_duration.record(duration)
        return order
```

**Structured Logging:**
```python
import structlog
logger = structlog.get_logger()

logger.info("order_placed",
    order_id="ord_123",
    customer_id="cust_456",
    total=99.99,
    trace_id=get_current_trace_id(),
)
# Output: {"event": "order_placed", "order_id": "ord_123", ...}
```

### Common Mistakes
1. **Alert fatigue** — Too many alerts; team ignores them. Alert on SLO breaches, not every metric
2. **Unstructured logs** — `print(f"Error processing order {order_id}")` is un-queryable
3. **No trace correlation** — Logs without trace IDs can't be correlated across services
4. **Vanity dashboards** — Dashboards showing metrics nobody acts on
5. **Monitoring without observability** — Knowing something is broken but not why

### Integration with Other Topics
- **Site Reliability Engineering** — SRE defines SLOs that observability measures
- **Microservices** — Distributed tracing is essential for microservices debugging
- **CI/CD** — Observability validates deployments
- **Resilience Patterns** — Circuit breaker state changes should emit metrics
- **Cloud-Native** — Cloud-native apps emit telemetry to stdout/OpenTelemetry
- **Incident Management** — Observability data drives incident investigation

---

## Resources

### Essential Reading
- *Observability Engineering* — Charity Majors, Liz Fong-Jones, George Miranda
- *Distributed Systems Observability* — Cindy Sridharan
- *Site Reliability Engineering* — Google (monitoring chapters)

### Online Resources
- OpenTelemetry documentation (opentelemetry.io)
- Grafana + Prometheus + Loki stack documentation
- Charity Majors' blog on observability

### Practice Exercises
1. Instrument a service with OpenTelemetry (metrics + traces)
2. Set up structured logging with trace ID propagation
3. Define SLIs/SLOs and create an error budget dashboard
4. Trace a request across 3 services using distributed tracing
5. Create an alert that fires only when the SLO error budget is at risk
