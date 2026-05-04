# security.md

## Purpose

Define universal security conventions that apply across all layers —
backend, frontend, database, and cloud. This skill is loaded on every
task regardless of project type.

---

## Conventions

### Secrets Management

- Never hardcode secrets, API keys, tokens, or passwords anywhere in code
- All secrets must live in environment variables loaded via a config system
- Use `pydantic-settings` (Python) or `dotenv` (Node.js) to load env vars
- Never commit `.env` files — always include in `.gitignore`
- Use a secrets manager in production: AWS Secrets Manager, GCP Secret Manager, or HashiCorp Vault
- Rotate secrets on any suspected exposure immediately

```python
# core/config.py — correct pattern
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str
    SECRET_KEY: str
    REDIS_URL: str

    class Config:
        env_file = ".env"

settings = Settings()
```

### Input Validation

- Validate all input at the API boundary — never trust client data
- Use Pydantic models (Python) or Zod schemas (TypeScript) for all inputs
- Reject requests with unexpected or extra fields
- Validate file uploads: check MIME type, file size, and extension
- Sanitize all user-supplied strings before using in queries or rendering

### SQL Injection Prevention

- Never use string concatenation or f-strings to build SQL queries
- Always use parameterized queries or ORM methods
- Never expose raw database error messages to clients

```python
# WRONG
query = f"SELECT * FROM users WHERE email = '{email}'"

# CORRECT
result = await db.execute(select(User).where(User.email == email))
```

### XSS Prevention (Web)

- Never render raw user-supplied HTML — always escape or sanitize
- Use `DOMPurify` for any case where HTML must be rendered from user input
- Set `Content-Security-Policy` headers on all web responses
- Never use `dangerouslySetInnerHTML` in React without sanitization

### CSRF Prevention

- Use `SameSite=Strict` or `SameSite=Lax` on all cookies
- Validate `Origin` and `Referer` headers on state-changing requests
- Use CSRF tokens for form submissions in non-SPA applications

### Authentication Security

- Enforce HTTPS only — never transmit credentials over HTTP
- Implement rate limiting on all auth endpoints (login, register, reset password)
- Lock accounts after 5 consecutive failed login attempts
- Always use secure, httpOnly cookies for refresh tokens
- Never store tokens in localStorage — vulnerable to XSS

### Authorization

- Always enforce authorization checks at the API layer — never only on the frontend
- Apply principle of least privilege — users get minimum permissions needed
- Never rely on security through obscurity (hidden endpoints are not protected endpoints)
- Always verify ownership before allowing access to a resource

```python
# Always verify the resource belongs to the requesting user
async def get_order(order_id: str, current_user: User = Depends(get_current_user)):
    order = await order_service.get(order_id)
    if order.user_id != current_user.id:
        raise ForbiddenError("Access denied")
    return order
```

### HTTP Security Headers

Always set these headers on all API and web responses:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

### Dependency Security

- Run `pip audit` (Python) or `npm audit` (Node.js) before every release
- Never use packages with known critical vulnerabilities
- Pin dependency versions in production — avoid floating versions like `^1.0.0`
- Review new dependencies before adding — check download count, maintenance status

### Logging Security

- Never log passwords, tokens, API keys, or PII
- Never log full request bodies that may contain sensitive fields
- Mask sensitive fields in logs: `email: "f***@***.com"`
- Log authentication events: login success, login failure, token refresh, logout

---

## Anti-Patterns

- Never hardcode any secret or credential in source code
- Never trust client-supplied data without server-side validation
- Never build SQL queries with string concatenation
- Never store sensitive tokens in localStorage or sessionStorage
- Never expose stack traces, DB errors, or internal paths in API responses
- Never skip authorization checks assuming the frontend will handle it
- Never use HTTP in production — always HTTPS
- Never commit `.env` files or any file containing real credentials

---

## Security Checklist (Run Before Every Commit)

- [ ] No hardcoded secrets, keys, or passwords in diff
- [ ] No `.env` files staged
- [ ] All new inputs validated with Pydantic or Zod
- [ ] No raw string SQL queries
- [ ] No sensitive data in log statements
- [ ] Authorization checks present on all new endpoints
- [ ] No `dangerouslySetInnerHTML` without sanitization

---

## Ready-to-Use Prompt

```
Task: Security review and hardening for [module or full project]
Skill: core/security, backend/auth, backend/error-handling

SCOPE: [single endpoint / module / full project]

CHECKLIST — verify each item:
Secrets:
  - [ ] No hardcoded secrets anywhere in codebase
  - [ ] All secrets loaded via environment config
  - [ ] .env excluded from git

Input Validation:
  - [ ] All inputs validated at API boundary
  - [ ] File uploads validated for type and size
  - [ ] No raw string SQL queries

Auth & Authorization:
  - [ ] Rate limiting on auth endpoints
  - [ ] Authorization checks on all protected endpoints
  - [ ] Ownership verified before resource access
  - [ ] Tokens stored securely (httpOnly cookie, not localStorage)

Headers:
  - [ ] Security headers set on all responses

Logging:
  - [ ] No sensitive data in log statements
  - [ ] Auth events logged

Dependencies:
  - [ ] pip audit / npm audit run and clean

DONE WHEN:
- All checklist items pass
- No secrets exposed in code or logs
- Code review skill applied before commit
```
