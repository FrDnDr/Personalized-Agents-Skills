# API Design Skill

## Purpose

Guide the agent in designing, structuring, and implementing APIs across all stacks
in a modular, consistent, and reusable template-driven way.

---

## Conventions

### General (All Stacks)

- Always separate concerns: routes, controllers, services, models are never in one file
- Every endpoint must have: input validation, error handling, and a consistent response shape
- Response format is always:

```json
{
  "success": true,
  "data": {},
  "error": null,
  "message": "string"
}
```

- Version all APIs from the start: `/api/v1/`
- Use plural nouns for resources: `/users`, `/orders`, `/products`
- Never expose internal error stack traces in responses
- Always document endpoints inline with comments or docstrings

### Module Structure (All Stacks)

/api
/v1
/users
router.py / router.js
controller.py / controller.js
service.py / service.js
schema.py / schema.js
/orders
...
/shared
response.py / response.js # Shared response formatter
errors.py / errors.js # Custom error classes
middleware.py / middleware.js # Auth, logging, validation

### REST API (Generic)

- GET — retrieve, never mutate
- POST — create new resource
- PUT — full update
- PATCH — partial update
- DELETE — remove resource
- Always return appropriate HTTP status codes:
  - 200 OK, 201 Created, 204 No Content
  - 400 Bad Request, 401 Unauthorized, 403 Forbidden
  - 404 Not Found, 409 Conflict, 422 Unprocessable Entity
  - 500 Internal Server Error

### FastAPI (Python)

- Always use `APIRouter` per module, never define routes in `main.py`
- Use Pydantic models for all request and response schemas
- Dependency injection for DB sessions, auth, and shared services
- Async endpoints by default (`async def`)
- Always use `HTTPException` for error responses
- Group routers in `main.py` using `app.include_router()`
- Example module template:

```python
  # router.py
  from fastapi import APIRouter, Depends, HTTPException
  from .service import UserService
  from .schema import UserCreate, UserResponse

  router = APIRouter(prefix="/users", tags=["users"])

  @router.post("/", response_model=UserResponse, status_code=201)
  async def create_user(payload: UserCreate, service: UserService = Depends()):
      return await service.create(payload)
```

### Express (Node.js)

- Always use `express.Router()` per module
- Use middleware for validation (Zod or Joi)
- Centralized error handler in `middleware/errorHandler.js`
- Always use `async/await` with try/catch or an `asyncWrapper` utility
- Example module template:

```javascript
// router.js
const express = require("express");
const router = express.Router();
const { getUsers, createUser } = require("./controller");
const { validate } = require("../shared/middleware");
const { createUserSchema } = require("./schema");

router.get("/", getUsers);
router.post("/", validate(createUserSchema), createUser);

module.exports = router;
```

### GraphQL

- Always use schema-first design — define `.graphql` schema files first
- Separate resolvers per type into their own files
- Use DataLoader for batching and avoiding N+1 queries
- Never put business logic in resolvers — delegate to service layer
- Always validate input with custom scalars or input types
- Example structure:
  /graphql
  /schema
  user.graphql
  order.graphql
  /resolvers
  user.resolver.js
  order.resolver.js
  /dataloaders
  user.loader.js

### gRPC (when applicable)

- Define `.proto` files first before any implementation
- One service per domain module
- Always handle error codes explicitly using gRPC status codes
- Never skip input validation on server-side handlers

---

## Anti-Patterns

- Never put route logic, business logic, and DB calls in the same file
- Never return raw database objects directly — always map to a response schema
- Never use generic error messages like `"Something went wrong"` without a code
- Never skip versioning — `/api/users` with no version is not acceptable
- Never hardcode base URLs, ports, or credentials inside route files
- Never mix async and sync patterns in the same codebase without explicit reason

---

## Ready-to-Use Prompt

```
Task: Design and implement a [REST/FastAPI/Express/GraphQL] API module for [resource name]
Skills: api-design, backend, security, testing
Stack: [specify your stack]
REQUIREMENTS:

Endpoints needed: [list CRUD or specific endpoints]
Auth required: [yes/no, type]
Input validation: required on all endpoints
Response format: standard success/error envelope

CONSTRAINTS:

Follow modular folder structure per api-design.md
No business logic in route/controller layer
All endpoints must be versioned under /api/v1/
Use shared response formatter and error classes
Do not expose stack traces in error responses
Commit after each endpoint is implemented and tested

IF BLOCKED:

Document the blocker as a TODO comment
Move to the next endpoint
Do not ask for confirmation — make a decision and log reasoning

DONE WHEN:

All endpoints implemented and modularized
Input validation on every route
Error handling covers all edge cases
Inline documentation added
Code review skill applied before commit
```
