# rest-api.md

## Purpose

Define FastAPI conventions for building REST APIs — covering router structure,
dependency injection, request/response handling, and module organization.

---

## Conventions

### Folder Structure

```
/app
  main.py               # App entry point, router registration only
  /api
    /v1
      /users
        router.py       # Route definitions
        controller.py   # Request handling, delegates to service
        service.py      # Business logic
        schema.py       # Pydantic request/response models
        dependencies.py # Route-specific dependencies
      /orders
        ...
  /core
    config.py           # Settings via pydantic-settings
    database.py         # DB engine, session factory
    security.py         # Password hashing, token utils
  /shared
    response.py         # Standard response formatter
    errors.py           # Custom exception classes
    middleware.py       # Auth, logging, CORS middleware
  /models
    user.py             # SQLAlchemy ORM models
    order.py
```

### Router Conventions

- Every module has its own `APIRouter` in `router.py`
- Routers are registered in `main.py` only — never in other files
- Always set `prefix` and `tags` on the router

```python
# router.py
from fastapi import APIRouter
from .controller import create_user, get_user

router = APIRouter(prefix="/users", tags=["users"])

router.post("/", response_model=UserResponse, status_code=201)(create_user)
router.get("/{user_id}", response_model=UserResponse)(get_user)
```

```python
# main.py
from fastapi import FastAPI
from app.api.v1.users.router import router as users_router
from app.api.v1.orders.router import router as orders_router

app = FastAPI()
app.include_router(users_router, prefix="/api/v1")
app.include_router(orders_router, prefix="/api/v1")
```

### Controller Conventions

- Controllers handle HTTP concerns only: parsing request, calling service, returning response
- No business logic in controllers — delegate everything to the service layer
- Always use async def
- Always use dependency injection for DB sessions and auth

```python
# controller.py
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.core.database import get_db
from .service import UserService
from .schema import UserCreate, UserResponse

async def create_user(
    payload: UserCreate,
    db: AsyncSession = Depends(get_db),
) -> UserResponse:
    service = UserService(db)
    return await service.create(payload)
```

### Service Conventions

- All business logic lives in the service layer
- Services receive a DB session via constructor injection
- Services raise custom exceptions — never HTTPException
- Services never import from routers or controllers

```python
# service.py
from sqlalchemy.ext.asyncio import AsyncSession
from app.shared.errors import NotFoundError, ConflictError
from .schema import UserCreate

class UserService:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def create(self, payload: UserCreate):
        existing = await self._get_by_email(payload.email)
        if existing:
            raise ConflictError("Email already registered")
        # create and return user
```

### Schema Conventions

- Separate schemas for request (input) and response (output)
- Never return ORM models directly — always map to response schema
- Use `model_config = ConfigDict(from_attributes=True)` for ORM compatibility
- Naming: `UserCreate`, `UserUpdate`, `UserResponse`, `UserListResponse`

### Standard Response Format

All endpoints return a consistent envelope:

```python
# shared/response.py
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar("T")

class APIResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None
    message: Optional[str] = None
```

### HTTP Status Codes

| Situation                 | Status Code |
| ------------------------- | ----------- |
| Successful GET            | 200         |
| Successful POST (created) | 201         |
| Successful DELETE         | 204         |
| Validation error          | 422         |
| Unauthorized              | 401         |
| Forbidden                 | 403         |
| Not found                 | 404         |
| Conflict (duplicate)      | 409         |
| Server error              | 500         |

---

## Anti-Patterns

- Never put business logic in routers or controllers
- Never import `HTTPException` in the service layer — use custom errors
- Never return raw ORM objects from endpoints
- Never define routes directly in `main.py`
- Never use synchronous `def` for endpoints that do I/O — always `async def`
- Never skip response_model on endpoints — always declare output schema
- Never hardcode DB connection strings — use pydantic-settings config

---

## Ready-to-Use Prompt

```
Task: Implement [resource name] API module with [list of endpoints]
Skill: backend/rest-api, backend/auth, backend/error-handling, database/schema-design

REQUIREMENTS:
- Resource: [name]
- Endpoints: [GET list / GET by id / POST / PUT / PATCH / DELETE]
- Auth required: [yes/no]
- Validation rules: [describe input constraints]

CONSTRAINTS:
- Follow modular folder structure: router / controller / service / schema
- No business logic in controller layer
- Async endpoints only
- Standard APIResponse envelope for all responses
- Custom exceptions in service — never HTTPException
- response_model declared on every endpoint
- Versioned under /api/v1/

IF BLOCKED:
- Document blocker as TODO comment
- Log reasoning for non-obvious design decisions

DONE WHEN:
- All endpoints implemented and registered
- Input validation via Pydantic schemas
- Service layer handles all business logic
- Error cases return correct status codes
- Code review skill applied before commit
```
