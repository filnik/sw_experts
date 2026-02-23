# Zero Trust Architecture

## Profile

### What It Is
Zero Trust is a security model based on the principle "never trust, always verify." It eliminates implicit trust based on network location, instead requiring continuous verification of identity, device health, and context for every access request, regardless of whether the request originates inside or outside the network perimeter.

### Origin and Evolution
- John Kindervag (Forrester) coined "Zero Trust" in 2010
- Google's BeyondCorp (2014) — first large-scale implementation
- NIST SP 800-207 formalized Zero Trust Architecture (2020)
- Accelerated by remote work (COVID-19) and cloud adoption
- Current: identity-centric security, continuous verification, microsegmentation

### Key Authors and References
- **John Kindervag** — Zero Trust concept creator (Forrester, 2010)
- **Google** — BeyondCorp papers (2014-2016)
- **NIST** — SP 800-207 "Zero Trust Architecture" (2020)
- **Chase Cunningham** — *Zero Trust Security* (O'Reilly)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Never Trust, Always Verify | Every request is authenticated and authorized regardless of origin |
| Least Privilege Access | Minimum necessary permissions for each request |
| Microsegmentation | Fine-grained network segmentation limiting lateral movement |
| Continuous Verification | Ongoing validation of trust, not just at login |
| Assume Breach | Design assuming attackers are already inside the network |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Verify explicitly** — Always authenticate and authorize based on all available data points: identity, location, device health, service, data classification.
2. **Use least privilege access** — Limit user access with just-in-time and just-enough access (JIT/JEA). Time-bound and scope-limited permissions.
3. **Assume breach** — Design as if the network is already compromised. Minimize blast radius, segment access, encrypt everything, and verify end-to-end.
4. **Identity is the new perimeter** — The network perimeter is dissolved. Identity (user, device, service) is the control plane.
5. **Continuous assessment** — Trust is not binary or permanent. Continuously evaluate risk signals and adjust access accordingly.

### When to Use vs. When to Avoid

**Adopt Zero Trust when:**
- Workforce is distributed/remote
- Applications run in the cloud
- Sensitive data requires protection beyond network boundaries
- Compliance mandates strong access controls
- The organization is targeted by sophisticated threats

**Implement incrementally:**
- Zero Trust is a journey, not a destination
- Start with identity and access management
- Then extend to network, device, and application layers
- Full implementation spans years for large organizations

---

## Frameworks and Methodologies

### 1. Zero Trust Implementation — step-by-step
1. Identify the protect surface (critical data, assets, applications, services)
2. Map the transaction flows (how subjects access the protect surface)
3. Build a Zero Trust architecture around the protect surface
4. Create Zero Trust policies (who, what, when, where, why, how)
5. Monitor and maintain — continuously log, analyze, and adjust
6. Iterate: expand to additional protect surfaces

### 2. Software-Level Zero Trust — step-by-step
1. Implement mutual TLS (mTLS) between all services
2. Require authentication tokens for every inter-service call
3. Validate tokens at every service boundary (no implicit trust)
4. Implement fine-grained authorization (RBAC/ABAC) at each service
5. Encrypt data at rest and in transit
6. Log all access for audit and anomaly detection

---

## How to Apply

### Decision Checklist
- [ ] Is every service-to-service call authenticated (mTLS or tokens)?
- [ ] Is authorization checked at every service boundary?
- [ ] Are permissions least-privilege and time-scoped?
- [ ] Is all traffic encrypted (TLS in transit, encryption at rest)?
- [ ] Are all access events logged for audit?

### Implementation Patterns

**Service-to-Service mTLS:**
```yaml
# Istio PeerAuthentication — require mTLS for all services
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT  # Reject any non-mTLS traffic

# Authorization policy — only order-service can call inventory-service
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: inventory-access
  namespace: production
spec:
  selector:
    matchLabels:
      app: inventory-service
  rules:
    - from:
        - source:
            principals: ["cluster.local/ns/production/sa/order-service"]
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/v1/inventory/*"]
```

**Token Validation at Every Service:**
```python
class ZeroTrustMiddleware:
    """Every service validates tokens independently — no implicit trust."""

    async def __call__(self, request: Request, call_next):
        token = request.headers.get("Authorization", "").replace("Bearer ", "")
        if not token:
            return JSONResponse(status_code=401, content={"error": "No token"})

        # Validate token locally (public key verification)
        try:
            claims = jwt.decode(token, key=PUBLIC_KEY, algorithms=["RS256"])
        except jwt.InvalidTokenError:
            return JSONResponse(status_code=401, content={"error": "Invalid token"})

        # Check token hasn't been revoked (real-time check)
        if await is_token_revoked(claims["jti"]):
            return JSONResponse(status_code=401, content={"error": "Token revoked"})

        # Continuous risk assessment
        risk_score = assess_risk(
            user_id=claims["sub"],
            ip=request.client.host,
            device=request.headers.get("X-Device-ID"),
        )
        if risk_score > RISK_THRESHOLD:
            return JSONResponse(status_code=403, content={"error": "Risk too high"})

        request.state.user = claims
        return await call_next(request)
```

**Microsegmentation with Network Policies:**
```yaml
# Kubernetes NetworkPolicy — only specific services can communicate
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: order-service-policy
spec:
  podSelector:
    matchLabels:
      app: order-service
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api-gateway
      ports:
        - port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: inventory-service
      ports:
        - port: 8080
    - to:
        - podSelector:
            matchLabels:
              app: payment-service
      ports:
        - port: 8080
```

### Common Mistakes
1. **Zero Trust = no trust** — Zero Trust doesn't mean no access; it means verified, authorized access
2. **Network-only approach** — Microsegmentation alone isn't Zero Trust without identity verification
3. **One-time verification** — Verifying only at login; trust must be continuously assessed
4. **Ignoring service-to-service** — Applying Zero Trust to user access but not inter-service communication
5. **Big-bang implementation** — Trying to implement everything at once; Zero Trust is incremental

### Integration with Other Topics
- **Authentication and Authorization** — Identity is the foundation of Zero Trust
- **Service Discovery and Mesh** — Service mesh implements mTLS and authorization policies
- **Application Security** — Defense in depth at the application layer
- **Microservices** — Each service independently verifies trust
- **Observability** — Comprehensive logging is essential for Zero Trust monitoring
- **Data Privacy** — Zero Trust protects sensitive data access

---

## Resources

### Essential Reading
- NIST SP 800-207 "Zero Trust Architecture"
- Google BeyondCorp papers (research.google)
- *Zero Trust Security* — Chase Cunningham

### Online Resources
- CISA Zero Trust Maturity Model
- Microsoft Zero Trust deployment guide
- Google BeyondCorp Enterprise documentation

### Practice Exercises
1. Implement mTLS between two services using Istio
2. Create Kubernetes NetworkPolicies for microsegmentation
3. Build middleware that validates tokens at every service boundary
4. Design a risk-scoring engine for continuous trust assessment
5. Audit an existing architecture against NIST Zero Trust principles
