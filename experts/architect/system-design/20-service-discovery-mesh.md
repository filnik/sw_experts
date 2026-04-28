# Service Discovery and Service Mesh

## Profile

### What It Is
Service Discovery is the mechanism by which services locate each other in a dynamic environment where instances scale up/down and IP addresses change. A Service Mesh is a dedicated infrastructure layer that handles service-to-service communication, providing observability, security (mTLS), and traffic management without changes to application code.

### Origin and Evolution
- Service discovery originated with DNS and evolved to purpose-built registries (Eureka, Consul, ZooKeeper)
- Kubernetes built-in service discovery (kube-dns, CoreDNS) simplified container environments
- Service mesh emerged from sidecar proxy patterns at Lyft (Envoy, 2016) and Google/IBM (Istio, 2017)
- Linkerd (originally Buoyant, 2016) was the first service mesh
- Modern: eBPF-based meshes (Cilium), ambient mesh (Istio without sidecars), simplified meshes

### Key Authors and References
- **Matt Klein** — Envoy proxy creator (Lyft)
- **William Morgan** — Linkerd creator, coined "service mesh"
- **Istio Team** — Google, IBM, Lyft collaboration
- **Kelsey Hightower** — Kubernetes and service mesh evangelism

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Service Registry | Central database of service instances and their locations |
| Health Check | Periodic verification that a service instance is alive |
| Sidecar Proxy | Proxy co-deployed with each service instance (e.g., Envoy) |
| Control Plane | Manages mesh configuration and policy (e.g., Istiod) |
| Data Plane | Sidecar proxies that handle actual traffic |
| mTLS | Mutual TLS authentication between services |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Dynamic location** — In cloud-native environments, service locations change constantly. Hard-coded addresses are a liability.
2. **Health-aware routing** — Only route traffic to healthy instances. Unhealthy instances should be automatically removed.
3. **Infrastructure-level concerns** — Retries, timeouts, mTLS, and observability are cross-cutting. Handle them in infrastructure, not application code.
4. **Zero-trust networking** — Services authenticate each other (mTLS). No implicit trust based on network location.
5. **Incremental adoption** — Service mesh can be adopted service by service. Don't require a big-bang migration.

### When to Use vs. When to Avoid

**Service Discovery — use when:**
- Services scale dynamically (containers, cloud)
- Multiple instances of the same service exist
- IP addresses are ephemeral

**Service Mesh — use when:**
- 10+ microservices with complex communication patterns
- Consistent security (mTLS) is required between all services
- Cross-cutting observability without code changes is needed
- Fine-grained traffic control (canary, A/B) is required

**Avoid service mesh when:**
- Few services (< 5) — the overhead isn't justified
- Team lacks Kubernetes operational maturity
- Latency from sidecar proxies is unacceptable
- Simpler solutions (library-based) suffice

---

## Frameworks and Methodologies

### 1. Service Discovery Setup — step-by-step
1. Choose discovery mechanism: DNS-based (Kubernetes) or registry-based (Consul, Eureka)
2. Register services on startup; deregister on shutdown
3. Implement health checks (HTTP, TCP, or gRPC health endpoints)
4. Configure client-side or server-side load balancing
5. Handle stale registrations (TTL, periodic re-registration)
6. Test failover: kill an instance and verify traffic reroutes

### 2. Service Mesh Adoption — step-by-step
1. Start with observability: inject sidecars to get metrics, traces, and logs
2. Enable mTLS between high-security services first
3. Add traffic management: timeouts, retries, circuit breakers via mesh config
4. Implement canary deployments using mesh traffic splitting
5. Extend to all services incrementally
6. Monitor sidecar resource consumption and latency overhead

---

## How to Apply

### Decision Checklist
- [ ] Is service discovery dynamic or can static config suffice?
- [ ] Does the environment justify a service mesh (10+ services)?
- [ ] Is the team prepared to operate the mesh control plane?
- [ ] Is the latency overhead of sidecars acceptable?
- [ ] Are mTLS and zero-trust requirements present?

### Implementation Patterns

**Kubernetes Service Discovery:**
```yaml
# Service definition — Kubernetes provides DNS-based discovery
apiVersion: v1
kind: Service
metadata:
  name: order-service
spec:
  selector:
    app: order-service
  ports:
    - port: 80
      targetPort: 8080
---
# Other services discover via DNS: http://order-service.default.svc.cluster.local
# Or simply: http://order-service (within same namespace)
```

**Consul Service Registration:**
```python
import consul

client = consul.Consul()

# Register service
client.agent.service.register(
    name="order-service",
    service_id="order-service-1",
    address="10.0.1.5",
    port=8080,
    check=consul.Check.http("http://10.0.1.5:8080/health", interval="10s"),
    tags=["v2", "production"],
)

# Discover service
_, services = client.health.service("order-service", passing=True)
healthy_instances = [
    f"{s['Service']['Address']}:{s['Service']['Port']}"
    for s in services
]
```

**Istio Traffic Management:**
```yaml
# Virtual Service — canary deployment
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: order-service
spec:
  hosts:
    - order-service
  http:
    - route:
        - destination:
            host: order-service
            subset: v1
          weight: 90
        - destination:
            host: order-service
            subset: v2
          weight: 10
      retries:
        attempts: 3
        perTryTimeout: 2s
      timeout: 10s
---
# Destination Rule — circuit breaker
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: order-service
spec:
  host: order-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: DEFAULT
        http1MaxPendingRequests: 100
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 60s
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

**Health Check Endpoint:**
```python
@app.get("/health")
def health_check():
    checks = {
        "database": check_db_connection(),
        "cache": check_redis_connection(),
        "disk": check_disk_space(),
    }
    all_healthy = all(checks.values())
    return JSONResponse(
        status_code=200 if all_healthy else 503,
        content={"status": "healthy" if all_healthy else "unhealthy", "checks": checks},
    )
```

### Common Mistakes
1. **Hard-coded service addresses** — Breaks in dynamic environments; use discovery
2. **No health checks** — Traffic routed to dead instances causes errors
3. **Service mesh for 3 services** — Operational overhead exceeds benefits for small deployments
4. **Ignoring sidecar overhead** — Each sidecar consumes CPU/memory and adds latency (1-3ms)
5. **Control plane as SPOF** — The mesh control plane must be highly available
6. **Not testing mesh policies** — mTLS or traffic rules that break connectivity in production

### Integration with Other Topics
- **Microservices** — Service discovery and mesh are essential microservices infrastructure
- **Containerization and Orchestration** — Kubernetes provides built-in service discovery
- **Resilience Patterns** — Mesh implements circuit breakers, retries, and timeouts
- **Observability** — Mesh provides distributed tracing and metrics without code changes
- **Zero Trust Architecture** — mTLS in the mesh implements zero-trust service-to-service
- **Deployment Strategies** — Mesh enables canary and blue-green deployments

---

## Resources

### Essential Reading
- *Istio in Action* — Christian Posta & Rinor Maloku
- *The Enterprise Path to Service Mesh Architectures* — Lee Calcote
- Envoy Proxy documentation

### Online Resources
- Istio documentation (istio.io)
- Linkerd documentation (linkerd.io)
- Consul documentation (consul.io)
- Matt Klein's blog on Envoy and service mesh

### Practice Exercises
1. Set up Kubernetes service discovery with DNS resolution between 2 services
2. Register a service with Consul and implement client-side discovery
3. Deploy Istio and configure mTLS between services
4. Implement canary deployment using Istio traffic splitting
5. Compare latency with and without sidecar proxies
