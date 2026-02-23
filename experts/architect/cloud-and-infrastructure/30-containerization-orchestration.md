# Containerization and Orchestration

## Profile

### What It Is
Containerization packages applications with their dependencies into lightweight, portable units (containers) that run consistently across environments. Orchestration (Kubernetes) automates the deployment, scaling, networking, and management of containerized applications at scale.

### Origin and Evolution
- Linux containers (LXC, cgroups, namespaces) — kernel-level isolation (2008)
- Docker (2013) — made containers accessible with simple CLI and image format
- Kubernetes (2014, Google) — open-sourced container orchestration from Borg
- Container ecosystem: Helm (package manager), Istio (service mesh), ArgoCD (GitOps)
- Current: Kubernetes is the de facto standard; focus shifting to developer experience and platform engineering

### Key Authors and References
- **Solomon Hykes** — Docker creator
- **Google/CNCF** — Kubernetes
- **Kelsey Hightower** — *Kubernetes Up & Running*, "Kubernetes the Hard Way"
- **Brendan Burns** — *Designing Distributed Systems*, Kubernetes co-creator

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Container | Isolated process with its own filesystem, network, and resource limits |
| Image | Immutable template for creating containers (layers, Dockerfile) |
| Pod | Smallest deployable unit in Kubernetes (one or more containers) |
| Deployment | Manages replicas, rolling updates, and rollbacks |
| Service | Stable network endpoint for a set of pods |
| Namespace | Virtual cluster for resource isolation |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Immutable artifacts** — Build once, deploy everywhere. The same container image runs in dev, staging, and production.
2. **Declarative desired state** — Define what you want (3 replicas, 512MB RAM). Kubernetes continuously reconciles actual state to match.
3. **Self-healing** — Failed containers are automatically restarted. Unhealthy pods are replaced. The system converges to the desired state.
4. **Horizontal scaling** — Scale by adding more container instances, not by increasing resources of existing ones.
5. **Separation of concerns** — Developers define the application (Dockerfile). Operators define the deployment (Kubernetes manifests). Platform teams provide the infrastructure.

### When to Use vs. When to Avoid

**Use containers when:**
- Consistent environments across dev/staging/production are needed
- Applications need isolation without VM overhead
- Microservices architecture with many deployable units
- CI/CD pipelines benefit from reproducible builds

**Use Kubernetes when:**
- Running 10+ containers in production
- Auto-scaling, self-healing, and rolling updates are needed
- Multiple teams deploy independently
- Service discovery and load balancing are required

**Avoid Kubernetes when:**
- Running 1-3 simple services (use Docker Compose or managed services)
- Team lacks Kubernetes expertise
- Managed container services (ECS, Cloud Run) meet needs with less complexity

---

## Frameworks and Methodologies

### 1. Dockerfile Best Practices — step-by-step
1. Use multi-stage builds (build stage + runtime stage)
2. Pin base image versions (don't use `latest`)
3. Order layers by change frequency (dependencies before code)
4. Use .dockerignore to exclude unnecessary files
5. Run as non-root user
6. Set health check in Dockerfile
7. Keep images small (Alpine or distroless base)

### 2. Kubernetes Deployment Strategy — step-by-step
1. Define resource requests and limits for every container
2. Configure liveness and readiness probes
3. Use Deployments for stateless, StatefulSets for stateful workloads
4. Set pod disruption budgets for availability during updates
5. Use namespaces for environment/team isolation
6. Store config in ConfigMaps, secrets in Secrets
7. Set up horizontal pod autoscaler (HPA)

---

## How to Apply

### Decision Checklist
- [ ] Is the Docker image using multi-stage build and non-root user?
- [ ] Are resource requests/limits set for all containers?
- [ ] Are liveness and readiness probes configured?
- [ ] Is the deployment strategy defined (rolling update, blue-green)?
- [ ] Are secrets managed securely (not in plaintext manifests)?

### Implementation Patterns

**Multi-Stage Dockerfile:**
```dockerfile
# Build stage
FROM python:3.12-slim AS builder
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen --no-dev
COPY src/ src/

# Runtime stage
FROM python:3.12-slim
RUN useradd --create-home appuser
WORKDIR /app
COPY --from=builder /app/.venv /app/.venv
COPY --from=builder /app/src /app/src
USER appuser
ENV PATH="/app/.venv/bin:$PATH"
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s CMD curl -f http://localhost:8080/health || exit 1
CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8080"]
```

**Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
        - name: order-service
          image: registry.example.com/order-service:v1.2.3
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 15
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: order-service-secrets
                  key: database-url
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### Common Mistakes
1. **Running as root** — Container processes run as root by default; always specify a non-root user
2. **No resource limits** — A single pod can consume all node resources, starving other pods
3. **Using `latest` tag** — Non-deterministic deployments; always pin image versions
4. **Large images** — Including build tools in runtime images; use multi-stage builds
5. **No health probes** — Kubernetes can't determine pod health without probes
6. **Secrets in environment variables in manifests** — Use Kubernetes Secrets or external secret managers

### Integration with Other Topics
- **Cloud-Native Architecture** — Containers are the runtime for cloud-native applications
- **CI/CD Pipelines** — Build images in CI, deploy to Kubernetes in CD
- **Microservices** — Each microservice runs in its own container(s)
- **Service Discovery and Mesh** — Kubernetes provides built-in service discovery
- **Infrastructure as Code** — Kubernetes manifests and Helm charts are IaC
- **Observability** — Container-level metrics, logs, and traces

---

## Resources

### Essential Reading
- *Kubernetes Up & Running* (3rd ed.) — Burns, Beda, Hightower, Evenson
- *Docker Deep Dive* — Nigel Poulton
- *Kubernetes Patterns* — Ibryam & Huß

### Online Resources
- Kubernetes documentation (kubernetes.io/docs)
- Kelsey Hightower's "Kubernetes the Hard Way"
- Docker documentation (docs.docker.com)

### Practice Exercises
1. Write a multi-stage Dockerfile for a Python or Node.js application
2. Deploy to Kubernetes with resource limits, probes, and HPA
3. Implement a rolling update and verify zero-downtime deployment
4. Set up a namespace with resource quotas for team isolation
5. Configure secrets management with external-secrets-operator
