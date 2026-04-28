# Application Security (OWASP)

## Profile

### What It Is
Application Security encompasses the practices, tools, and principles for identifying, fixing, and preventing security vulnerabilities in software applications. The OWASP (Open Web Application Security Project) Top 10 provides the industry-standard awareness document for the most critical web application security risks.

### Origin and Evolution
- OWASP founded 2001 as a nonprofit for software security
- OWASP Top 10 first published 2003, updated regularly (latest: 2021)
- Shift-left security: integrating security into development rather than testing after
- DevSecOps: security as part of CI/CD pipelines
- Current: API security (OWASP API Top 10), supply chain security, AI security

### Key Authors and References
- **OWASP Foundation** — Top 10, ASVS, Testing Guide, Cheat Sheet Series
- **Jim Manico** — OWASP leader, application security educator
- **Adam Shostack** — *Threat Modeling: Designing for Security*
- **Gary McGraw** — *Software Security: Building Security In*

### Key Concepts at a Glance
| OWASP Top 10 (2021) | Risk |
|---------------------|------|
| A01 | Broken Access Control |
| A02 | Cryptographic Failures |
| A03 | Injection |
| A04 | Insecure Design |
| A05 | Security Misconfiguration |
| A06 | Vulnerable and Outdated Components |
| A07 | Identification and Authentication Failures |
| A08 | Software and Data Integrity Failures |
| A09 | Security Logging and Monitoring Failures |
| A10 | Server-Side Request Forgery (SSRF) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Defense in depth** — Never rely on a single security control. Layer multiple defenses so that if one fails, others still protect the system.
2. **Least privilege** — Every component, user, and process should have the minimum permissions necessary to perform its function.
3. **Secure by default** — Default configurations should be secure. Users must explicitly opt into less secure options, not the other way around.
4. **Validate all input** — All input from external sources is untrusted. Validate, sanitize, and encode at every boundary.
5. **Shift left** — Integrate security into design and development, not just testing. Security bugs found early cost orders of magnitude less to fix.

### When to Use vs. When to Avoid

**Always apply:**
- Input validation and output encoding
- Authentication and authorization checks
- Secure configuration defaults
- Dependency vulnerability scanning

**Scale security investment based on:**
- Data sensitivity (PII, financial, health data)
- Exposure (public internet vs. internal)
- Regulatory requirements (SOC2, HIPAA, PCI-DSS)
- Threat model (who would attack and why)

---

## Frameworks and Methodologies

### 1. Threat Modeling (STRIDE) — step-by-step
1. Diagram the system: components, data flows, trust boundaries
2. Identify threats using STRIDE: Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege
3. For each threat, assess likelihood and impact
4. Define mitigations for high-risk threats
5. Prioritize: high-impact, high-likelihood threats first
6. Review threat model with every significant architecture change

### 2. Secure Development Lifecycle — step-by-step
1. Design: threat modeling, security requirements
2. Develop: secure coding guidelines, code review for security
3. Test: SAST (static analysis), DAST (dynamic testing), dependency scanning
4. Deploy: secure configuration, secrets management
5. Operate: monitoring, logging, incident response
6. Iterate: vulnerability management, patch management

---

## How to Apply

### Decision Checklist
- [ ] Is input validation applied at every entry point?
- [ ] Are SQL queries parameterized (no string concatenation)?
- [ ] Is output encoding applied before rendering user content?
- [ ] Are authentication and session management using proven libraries?
- [ ] Are secrets managed securely (not in code or config files)?
- [ ] Are dependencies scanned for known vulnerabilities?

### Implementation Patterns

**Injection Prevention:**
```python
# BAD: SQL injection vulnerable
query = f"SELECT * FROM users WHERE email = '{user_input}'"

# GOOD: Parameterized query
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (user_input,))

# BAD: Command injection
os.system(f"convert {filename} output.png")

# GOOD: Use subprocess with list arguments
subprocess.run(["convert", filename, "output.png"], check=True)
```

**XSS Prevention:**
```python
# Output encoding — always encode user data in HTML context
from markupsafe import escape

@app.route("/profile")
def profile():
    user_name = get_user_name()  # Could contain <script>alert(1)</script>
    return f"<h1>Welcome, {escape(user_name)}</h1>"

# Content Security Policy header
@app.after_request
def add_security_headers(response):
    response.headers["Content-Security-Policy"] = "default-src 'self'; script-src 'self'"
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    return response
```

**Access Control:**
```python
# Broken Access Control — checking only authentication, not authorization
@app.get("/orders/{order_id}")
def get_order(order_id: str, current_user: User = Depends(get_current_user)):
    order = order_repo.find(order_id)
    # BAD: Any authenticated user can see any order
    return order

    # GOOD: Verify the user owns this order
    if order.customer_id != current_user.id and not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Forbidden")
    return order
```

### Common Mistakes
1. **Client-side validation only** — Server must validate all input; client-side is for UX only
2. **Rolling your own crypto** — Use established libraries (bcrypt, argon2) for password hashing
3. **Secrets in source code** — API keys, passwords committed to Git
4. **Overly permissive CORS** — `Access-Control-Allow-Origin: *` on APIs with authentication
5. **No rate limiting on auth endpoints** — Brute force attacks on login and password reset

### Integration with Other Topics
- **Authentication and Authorization** — Core application security domain
- **Zero Trust Architecture** — Application-level security in a zero trust model
- **API Design** — API-specific security (OWASP API Top 10)
- **CI/CD Pipelines** — Security scanning in the pipeline (SAST, DAST, SCA)
- **Software Supply Chain Security** — Dependency and build security
- **Clean Code** — Secure code is a dimension of clean code

---

## Resources

### Essential Reading
- OWASP Top 10 (owasp.org/www-project-top-ten)
- *Threat Modeling: Designing for Security* — Adam Shostack
- OWASP Application Security Verification Standard (ASVS)
- OWASP Cheat Sheet Series

### Online Resources
- OWASP WebGoat (practice vulnerable application)
- PortSwigger Web Security Academy (free)
- Snyk vulnerability database

### Practice Exercises
1. Perform a STRIDE threat model for a web application
2. Find and fix injection vulnerabilities in a sample application
3. Implement Content Security Policy headers
4. Set up SAST and dependency scanning in a CI pipeline
5. Conduct a code review focused on OWASP Top 10 risks
