# Software Supply Chain Security

## Profile

### What It Is
Software Supply Chain Security protects the integrity of the software development and delivery pipeline — from source code and dependencies to build systems and artifact distribution. It addresses risks from compromised dependencies, tampered builds, and malicious code injection throughout the software lifecycle.

### Origin and Evolution
- SolarWinds attack (2020) — compromised build pipeline affected 18,000 organizations
- Log4Shell (2021) — critical vulnerability in ubiquitous open-source dependency
- SLSA framework (Supply-chain Levels for Software Artifacts, Google, 2021)
- Executive Order 14028 (US, 2021) — mandated SBOM and supply chain security
- Current: SBOM adoption, sigstore for artifact signing, dependency review automation

### Key Authors and References
- **Google** — SLSA framework (slsa.dev)
- **CISA** — Software supply chain security guidelines
- **Tidelift** — Open-source supply chain management
- **NIST** — SSDF (Secure Software Development Framework)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| SBOM | Software Bill of Materials — inventory of all components |
| SLSA | Framework for supply chain integrity (levels 1-4) |
| Dependency Scanning | Automated detection of known vulnerabilities in dependencies |
| Artifact Signing | Cryptographic signatures proving artifact provenance |
| Provenance | Verifiable record of how an artifact was built |
| Lock Files | Pinned dependency versions for reproducible builds |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Know your dependencies** — Maintain a complete inventory (SBOM) of every component in your software, including transitive dependencies.
2. **Verify integrity at every stage** — Sign artifacts, verify signatures, use checksums. Trust nothing that isn't cryptographically verified.
3. **Minimize the attack surface** — Fewer dependencies mean fewer attack vectors. Remove unused dependencies, prefer well-maintained libraries.
4. **Automate vulnerability detection** — Integrate dependency scanning into CI/CD. Don't rely on manual reviews to catch vulnerable components.
5. **Reproducible builds** — The same source code should always produce the same binary. This enables verification and auditing.

### When to Use vs. When to Avoid

**Always apply:**
- Dependency vulnerability scanning (Dependabot, Snyk, Trivy)
- Lock files for reproducible builds
- Automated dependency updates with testing

**Scale investment based on:**
- Sensitivity of the application (critical infrastructure, financial)
- Regulatory requirements (SBOM mandates)
- Organization size and supply chain complexity
- Threat model (targeted attacks vs. opportunistic)

---

## Frameworks and Methodologies

### 1. SLSA Implementation — step-by-step
1. **Level 1**: Documented build process, SBOM generated
2. **Level 2**: Version-controlled build configuration, signed provenance
3. **Level 3**: Hardened build platform, isolated build environment
4. **Level 4**: Reproducible builds, two-party review of all changes
5. Implement incrementally — each level adds security
6. Integrate provenance verification into deployment pipeline

### 2. Dependency Management — step-by-step
1. Audit current dependencies: remove unused, identify alternatives for unmaintained
2. Pin versions in lock files (package-lock.json, uv.lock, Cargo.lock)
3. Enable automated vulnerability scanning (Dependabot, Renovate, Snyk)
4. Configure automated dependency updates with CI testing
5. Review new dependency additions: maintenance, security history, license
6. Generate SBOM as part of the build process

---

## How to Apply

### Decision Checklist
- [ ] Are all dependencies pinned with lock files?
- [ ] Is automated vulnerability scanning in CI/CD?
- [ ] Is there a process for reviewing new dependencies?
- [ ] Are build artifacts signed and provenance tracked?
- [ ] Is an SBOM generated for releases?
- [ ] Are container base images scanned and updated?

### Implementation Patterns

**CI Pipeline with Supply Chain Security:**
```yaml
# GitHub Actions example
name: Secure Build
on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      id-token: write  # For signing
      contents: read
    steps:
      - uses: actions/checkout@v4

      # Dependency vulnerability scanning
      - name: Audit dependencies
        run: npm audit --audit-level=high

      # Container image scanning
      - name: Scan image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:${{ github.sha }}'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

      # Generate SBOM
      - name: Generate SBOM
        uses: anchore/sbom-action@v0
        with:
          artifact-name: sbom.spdx.json

      # Sign artifact with Sigstore
      - name: Sign image
        uses: sigstore/cosign-installer@v3
      - run: cosign sign --yes myregistry/myapp:${{ github.sha }}
```

**Dependency Review Policy:**
```python
# Pre-commit or CI check for new dependencies
class DependencyReview:
    BLOCKED_LICENSES = ["GPL-3.0", "AGPL-3.0"]
    MIN_MAINTENANCE_SCORE = 0.5

    def review_new_dependency(self, package: str) -> ReviewResult:
        info = self.registry.get_package_info(package)

        checks = {
            "license_ok": info.license not in self.BLOCKED_LICENSES,
            "maintained": info.last_publish_days < 365,
            "no_known_vulns": len(info.vulnerabilities) == 0,
            "sufficient_adoption": info.weekly_downloads > 1000,
            "has_security_policy": info.has_security_md,
        }

        return ReviewResult(
            approved=all(checks.values()),
            checks=checks,
        )
```

**SBOM Generation:**
```bash
# Generate SBOM in SPDX format
syft packages dir:. -o spdx-json > sbom.spdx.json

# Generate SBOM in CycloneDX format
cdxgen -o sbom.cdx.json

# Verify SBOM against known vulnerabilities
grype sbom:sbom.spdx.json
```

### Common Mistakes
1. **Ignoring transitive dependencies** — Your direct dependency is secure, but its dependency isn't
2. **No lock files** — `npm install` without lock file can pull different (possibly compromised) versions
3. **Outdated base images** — Container base images with known vulnerabilities
4. **No build reproducibility** — Builds depend on external state (latest tags, mutable sources)
5. **Manual dependency updates** — Dependencies go months without updates, accumulating vulnerabilities

### Integration with Other Topics
- **CI/CD Pipelines** — Supply chain security is a CI/CD pipeline concern
- **Containerization** — Container image scanning and signing
- **Application Security** — Vulnerable dependencies are a top security risk
- **DevOps** — Automated dependency management is a DevOps practice
- **Code Quality Tools** — Linters and scanners catch security issues
- **GitOps** — Version control and audit trails for all changes

---

## Resources

### Essential Reading
- SLSA framework (slsa.dev)
- NIST SSDF (Secure Software Development Framework)
- Sigstore documentation (sigstore.dev)

### Online Resources
- GitHub Supply Chain Security features
- Snyk Learn — supply chain security courses
- CISA Securing the Software Supply Chain guides

### Practice Exercises
1. Set up Dependabot or Renovate for automated dependency updates
2. Generate an SBOM for a project using Syft or CycloneDX
3. Sign a container image with Cosign/Sigstore
4. Implement a dependency review check in CI that blocks risky additions
5. Scan a project for known vulnerabilities and remediate critical findings
