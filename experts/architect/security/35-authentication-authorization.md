# Authentication and Authorization

## Profile

### What It Is
Authentication verifies identity ("who are you?"). Authorization determines permissions ("what can you do?"). Together, they form the access control foundation of every secure application, encompassing identity management, session handling, token-based auth, role/attribute-based access control, and OAuth2/OIDC flows.

### Origin and Evolution
- Basic/Digest HTTP authentication (1990s)
- Session-based authentication with cookies (2000s)
- OAuth 2.0 (2012) — delegated authorization standard
- OpenID Connect (2014) — identity layer on top of OAuth2
- JWT (JSON Web Tokens) became ubiquitous for stateless auth
- Current: passkeys (WebAuthn/FIDO2), zero-trust identity, decentralized identity

### Key Authors and References
- **IETF** — OAuth 2.0 (RFC 6749), JWT (RFC 7519), OAuth 2.1 (draft)
- **OpenID Foundation** — OpenID Connect specification
- **NIST** — Digital Identity Guidelines (SP 800-63)
- **Auth0/Okta** — Identity platform documentation and best practices

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| OAuth 2.0 | Delegated authorization framework (access tokens) |
| OpenID Connect | Identity layer on OAuth2 (ID tokens, user info) |
| JWT | JSON Web Token — compact, signed token format |
| RBAC | Role-Based Access Control (admin, editor, viewer) |
| ABAC | Attribute-Based Access Control (policies based on attributes) |
| MFA | Multi-Factor Authentication (something you know + have + are) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Separate authentication from authorization** — Identity verification and permission checking are distinct concerns. Don't conflate them.
2. **Use proven standards** — OAuth2, OIDC, SAML are battle-tested. Don't build custom auth protocols.
3. **Tokens are credentials** — JWTs, session IDs, API keys must be transmitted, stored, and handled with the same care as passwords.
4. **Least privilege by default** — Start with no permissions. Grant only what's needed, when it's needed.
5. **Auth decisions at the boundary** — Check authentication and authorization at the system boundary (API gateway, middleware). Don't scatter checks throughout business logic.

### When to Use vs. When to Avoid

**OAuth2/OIDC:** Third-party integrations, social login, mobile apps, SPAs
**Session-based:** Server-rendered web apps, simple setups
**JWT:** Microservices, stateless APIs, short-lived tokens
**API Keys:** Machine-to-machine communication, public APIs
**RBAC:** Clear role hierarchy, small number of roles
**ABAC:** Complex policies based on multiple attributes

---

## Frameworks and Methodologies

### 1. Auth Architecture Design — step-by-step
1. Identify authentication methods needed (password, social, SSO, MFA)
2. Choose identity provider (build vs. buy: Auth0, Keycloak, Cognito)
3. Design token strategy (JWT for APIs, sessions for web, refresh tokens)
4. Define authorization model (RBAC for simple, ABAC for complex)
5. Implement at API gateway/middleware level
6. Secure token storage (httpOnly cookies, secure storage)
7. Plan for token revocation and session management

### 2. RBAC/ABAC Implementation — step-by-step
1. Define roles from business requirements (not technical roles)
2. Map permissions to roles (read:orders, write:orders, admin:users)
3. Assign roles to users at the appropriate scope (global, org, project)
4. Implement permission checks in middleware or decorators
5. For ABAC: define policies as rules (user.department == resource.department)
6. Centralize policy evaluation (OPA, Casbin, Cedar)

---

## How to Apply

### Decision Checklist
- [ ] Are passwords hashed with a strong algorithm (bcrypt, argon2)?
- [ ] Are tokens short-lived with refresh token rotation?
- [ ] Is MFA available for sensitive operations?
- [ ] Are authorization checks enforced at every protected endpoint?
- [ ] Is there a token revocation mechanism?

### Implementation Patterns

**JWT Authentication Middleware:**
```python
from fastapi import Depends, HTTPException
from fastapi.security import HTTPBearer
import jwt

security = HTTPBearer()

async def get_current_user(token: str = Depends(security)) -> User:
    try:
        payload = jwt.decode(
            token.credentials,
            key=settings.jwt_public_key,
            algorithms=["RS256"],
            audience=settings.jwt_audience,
        )
        user_id = payload["sub"]
        roles = payload.get("roles", [])
        return User(id=user_id, roles=roles)
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

**Role-Based Authorization:**
```python
def require_role(*allowed_roles: str):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, current_user: User = Depends(get_current_user), **kwargs):
            if not any(role in current_user.roles for role in allowed_roles):
                raise HTTPException(status_code=403, detail="Insufficient permissions")
            return await func(*args, current_user=current_user, **kwargs)
        return wrapper
    return decorator

@app.delete("/users/{user_id}")
@require_role("admin")
async def delete_user(user_id: str, current_user: User = Depends(get_current_user)):
    await user_service.delete(user_id)
```

**OAuth2 Authorization Code Flow:**
```python
# 1. Redirect user to authorization server
redirect_url = (
    f"{AUTH_SERVER}/authorize?"
    f"client_id={CLIENT_ID}&"
    f"redirect_uri={REDIRECT_URI}&"
    f"response_type=code&"
    f"scope=openid profile email&"
    f"state={generate_state()}"
)

# 2. Handle callback — exchange code for tokens
async def oauth_callback(code: str, state: str):
    verify_state(state)
    token_response = await http_client.post(f"{AUTH_SERVER}/token", data={
        "grant_type": "authorization_code",
        "code": code,
        "redirect_uri": REDIRECT_URI,
        "client_id": CLIENT_ID,
        "client_secret": CLIENT_SECRET,
    })
    tokens = token_response.json()
    # tokens contains: access_token, id_token, refresh_token
    user_info = decode_id_token(tokens["id_token"])
    return create_session(user_info)
```

### Common Mistakes
1. **Storing passwords in plaintext** — Always hash with bcrypt/argon2 with salt
2. **Long-lived access tokens** — Access tokens should be short-lived (5-15 min); use refresh tokens
3. **JWT in localStorage** — Vulnerable to XSS; use httpOnly cookies for web applications
4. **No authorization check on resources** — Checking "is authenticated" but not "is authorized to access this resource"
5. **Hardcoded secrets** — API keys and JWT secrets in source code

### Integration with Other Topics
- **Application Security (OWASP)** — Auth failures are OWASP Top 10 A07
- **Zero Trust Architecture** — Identity is the new perimeter
- **API Design** — Auth is a core API concern
- **API Gateway** — Gateway handles token validation
- **Multi-Tenancy** — Tenant identification through auth tokens
- **Microservices** — Token propagation across service boundaries

---

## Resources

### Essential Reading
- OAuth 2.0 Simplified — Aaron Parecki (oauth.com)
- *Identity and Data Security for Web Development* — LeBlanc & Messerschmidt
- NIST SP 800-63 Digital Identity Guidelines

### Online Resources
- Auth0 documentation and architecture guides
- OAuth.net — community resources
- PortSwigger authentication vulnerabilities labs

### Practice Exercises
1. Implement JWT authentication with refresh token rotation
2. Build RBAC with role hierarchy (admin > editor > viewer)
3. Implement OAuth2 authorization code flow with PKCE
4. Add MFA (TOTP) to an existing authentication system
5. Design authorization for a multi-tenant application
