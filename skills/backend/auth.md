# auth.md

## Purpose

Define authentication and authorization conventions — covering JWT, OAuth2,
session handling, password security, and role-based access control (RBAC).

---

## Conventions

### Authentication Strategy

- Default: **JWT (JSON Web Tokens)** for stateless APIs
- OAuth2 for third-party login (Google, GitHub, etc.)
- Never roll custom crypto — use established libraries only

### JWT Conventions

- Access token expiry: **15 minutes**
- Refresh token expiry: **7 days**
- Store access token in memory (not localStorage) on web clients
- Store refresh token in httpOnly cookie — never in localStorage
- Always sign with RS256 (asymmetric) for production, HS256 acceptable for dev
- Token payload must include: `sub` (user id), `exp`, `iat`, `role`
- Never store sensitive data in token payload — it is base64 encoded, not encrypted

```python
# core/security.py
from datetime import datetime, timedelta
from jose import jwt
from passlib.context import CryptContext

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

def create_access_token(data: dict) -> str:
    expire = datetime.utcnow() + timedelta(minutes=15)
    return jwt.encode({**data, "exp": expire}, SECRET_KEY, algorithm="HS256")
```

### Dependency Injection for Auth

- Always use FastAPI dependencies for protected routes
- Never check auth inside controller or service logic
- Create reusable `get_current_user` and `require_role` dependencies

```python
# shared/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/api/v1/auth/token")

async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    # decode and validate token
    # return user or raise 401
    ...

def require_role(role: str):
    async def role_checker(user: User = Depends(get_current_user)):
        if user.role != role:
            raise HTTPException(status_code=403, detail="Insufficient permissions")
        return user
    return role_checker
```

### Password Rules

- Minimum 8 characters, require mixed case + number
- Always hash with bcrypt — never store plaintext passwords
- Never log passwords anywhere under any circumstance
- Rate limit login endpoints — max 5 attempts per minute per IP

### Role-Based Access Control (RBAC)

- Define roles as an Enum: `admin`, `user`, `viewer`
- Roles stored in DB and included in JWT payload
- Permission checks happen at the route level via dependencies — not in service
- Document which roles can access which endpoints inline on the router

### OAuth2 (Third-Party Login)

- Use `authlib` library for OAuth2 flows
- Always validate the `state` parameter to prevent CSRF
- Never trust email from OAuth provider without verifying it is confirmed
- Create or link user account on first OAuth login — never silently fail

### Refresh Token Flow

```
1. Client sends expired access token + valid refresh token
2. Server validates refresh token from httpOnly cookie
3. Server issues new access token + rotates refresh token
4. Old refresh token is invalidated immediately (rotation)
```

---

## Anti-Patterns

- Never store tokens in localStorage — vulnerable to XSS
- Never put auth logic inside service layer — use dependencies
- Never use symmetric keys (HS256) in production without rotation strategy
- Never skip token expiry — tokens must always expire
- Never trust user-supplied role claims without DB verification
- Never log or return the full token in error messages
- Never skip rate limiting on login and token refresh endpoints

---

## Ready-to-Use Prompt

```
Task: Implement authentication for [project name]
Skill: backend/auth, backend/rest-api, core/security

REQUIREMENTS:
- Auth type: [JWT / OAuth2 / both]
- Login method: [email+password / Google / GitHub]
- Roles needed: [list roles]
- Protected endpoints: [list or describe]

CONSTRAINTS:
- JWT: 15min access token, 7 day refresh token in httpOnly cookie
- Passwords hashed with bcrypt only
- Auth checks via FastAPI dependencies — not in service layer
- Rate limit login endpoint
- Never store tokens in localStorage
- Refresh token rotation on every refresh

DONE WHEN:
- Login endpoint returns access + refresh tokens correctly
- Protected routes reject invalid/expired tokens with 401
- Role-restricted routes return 403 for unauthorized roles
- Password hashing verified
- Rate limiting applied to login endpoint
- Code review skill applied before commit
```
