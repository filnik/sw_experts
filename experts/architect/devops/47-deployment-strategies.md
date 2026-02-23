# Deployment Strategies

## Profile

### What It Is
Deployment Strategies define how new versions of software are released to production, balancing speed, safety, and user experience. Strategies range from simple in-place updates to sophisticated progressive delivery techniques like canary, blue-green, and feature flags.

### Origin and Evolution
- Early: big-bang deployments with maintenance windows
- Blue-green deployments (Martin Fowler, ~2010)
- Canary releases (named after coal mine canaries)
- Feature flags for decoupling deployment from release
- Modern: progressive delivery, GitOps, automated rollback
- Current: deployment strategies as code (Argo Rollouts, Flagger)

### Key Authors and References
- **Jez Humble & David Farley** — *Continuous Delivery* (deployment patterns)
- **Martin Fowler** — Blue-green deployment article
- **James Governor** — "Progressive Delivery" term
- **Argo Rollouts / Flagger** — Kubernetes progressive delivery tools

### Key Concepts at a Glance
| Strategy | Risk | Rollback | Downtime | Cost |
|----------|------|----------|----------|------|
| Recreate | High | Slow | Yes | Low |
| Rolling Update | Medium | Medium | No | Low |
| Blue-Green | Low | Fast (switch) | No | 2x infra |
| Canary | Low | Fast (stop) | No | +10% infra |
| Feature Flags | Lowest | Instant (toggle) | No | Low |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Separate deployment from release** — Deployment puts code in production. Release makes it available to users. Feature flags decouple the two.
2. **Progressive delivery** — Expose new code to a small subset of users first. Gradually increase if metrics are healthy.
3. **Automated rollback** — If health metrics degrade, roll back automatically. Don't wait for a human to notice.
4. **Zero-downtime is the standard** — Users should never see a maintenance page. Rolling updates and blue-green make this achievable.
5. **Deployment should be boring** — When deployments are small, frequent, and automated, they become routine non-events.

### When to Use vs. When to Avoid

**Blue-Green**: When you need instant rollback and can afford double infrastructure
**Canary**: When you want gradual rollout with real traffic validation
**Feature Flags**: When specific features need independent release control
**Rolling Update**: Default for Kubernetes deployments; good enough for most cases

---

## Frameworks and Methodologies

### 1. Canary Deployment — step-by-step
1. Deploy new version alongside existing version
2. Route 5% of traffic to the new version
3. Monitor key metrics (error rate, latency, business KPIs)
4. If healthy, increase to 25%, 50%, 100%
5. If unhealthy, route 100% back to old version
6. Automate promotion/rollback based on metric thresholds

### 2. Feature Flag Lifecycle — step-by-step
1. Wrap new feature behind a flag (default: off)
2. Deploy code to production (flag off — no user impact)
3. Enable flag for internal users / beta testers
4. Enable for 10% of users, monitor metrics
5. Roll out to 100% when confident
6. Remove the flag and dead code (critical — don't accumulate flags)

---

## How to Apply

### Decision Checklist
- [ ] Is the deployment zero-downtime?
- [ ] Is there an automated rollback mechanism?
- [ ] Are health metrics monitored during deployment?
- [ ] Can deployment be separated from release (feature flags)?
- [ ] Is the deployment strategy codified (not manual)?

### Implementation Patterns

**Kubernetes Rolling Update:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1    # At most 1 pod down during update
      maxSurge: 1           # At most 1 extra pod during update
  minReadySeconds: 30       # Wait 30s after ready before continuing
```

**Argo Rollouts Canary:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
spec:
  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause: { duration: 5m }
        - analysis:
            templates: [{ templateName: success-rate }]
        - setWeight: 25
        - pause: { duration: 5m }
        - setWeight: 50
        - pause: { duration: 10m }
        - setWeight: 100
      canaryMetadata:
        labels:
          role: canary
```

**Feature Flag (Simple):**
```python
class FeatureFlags:
    def __init__(self, store: FlagStore):
        self.store = store

    def is_enabled(self, flag: str, user_id: str | None = None) -> bool:
        flag_config = self.store.get(flag)
        if not flag_config:
            return False
        if flag_config.enabled_for_all:
            return True
        if user_id and user_id in flag_config.user_allowlist:
            return True
        if flag_config.percentage > 0 and user_id:
            return hash(f"{flag}:{user_id}") % 100 < flag_config.percentage
        return False

# Usage
if flags.is_enabled("new_checkout_flow", user_id=current_user.id):
    return new_checkout(order)
else:
    return legacy_checkout(order)
```

### Common Mistakes
1. **No rollback plan** — Deploying without a tested rollback mechanism
2. **Feature flag debt** — Accumulating hundreds of flags without removing old ones
3. **Manual canary analysis** — Relying on humans to watch dashboards during canary
4. **Database migration incompatibility** — New code requires a DB migration that breaks old code during rollback
5. **Testing only in staging** — Production traffic reveals issues staging can't simulate

### Integration with Other Topics
- **CI/CD Pipelines** — Deployment strategies are the final pipeline stage
- **Observability** — Metrics drive canary promotion and automated rollback
- **Containerization** — Kubernetes provides deployment strategy primitives
- **Resilience Patterns** — Progressive delivery is a resilience mechanism
- **Site Reliability Engineering** — SRE manages deployment risk
- **GitOps** — Git-driven deployment with automated reconciliation

---

## Resources

### Essential Reading
- *Continuous Delivery* — Jez Humble & David Farley
- Argo Rollouts documentation
- LaunchDarkly's feature flag best practices

### Online Resources
- Martin Fowler — "Blue Green Deployment," "Canary Release" articles
- Flagger documentation (flagger.app)
- Feature flag platforms: LaunchDarkly, Unleash, Flagsmith

### Practice Exercises
1. Implement a blue-green deployment with instant rollback
2. Set up Argo Rollouts canary with automated metric analysis
3. Implement feature flags with percentage-based rollout
4. Practice a rollback scenario during a canary deployment
5. Design a database migration strategy compatible with rolling deployments
