# error-handling.md

## Purpose

Define global error handling conventions — covering custom exception classes,
error response format, global exception handlers, and what never to expose
in error responses.

---

## Conventions

### Custom Exception Classes

Define all application exceptions in `shared/errors.py`.
Never raise `HTTPException` inside the service layer — raise custom exceptions only.

```python
# shared/errors.py

class AppError(Exception):
    """Base exception for all application errors."""
    def __init__(self, message: str, code: str = "APP_ERROR"):
        self.message = message
        self.code = code
        super().__init__(message)

class NotFoundError(AppError):
    def __init__(self, resource: str):
        super().__init__(f"{resource} not found", code="NOT_FOUND")

class ConflictError(AppError):
    def __init__(self, message: str):
        super().__init__(message, code="CONFLICT")

class UnauthorizedError(AppError):
    def __init__(self, message: str = "Unauthorized"):
        super().__init__(message, code="UNAUTHORIZED")

class ForbiddenError(AppError):
    def __init__(self, message: str = "Access denied"):
        super().__init__(message, code="FORBIDDEN")

class ValidationError(AppError):
    def __init__(self, message: str):
        super().__init__(message, code="VALIDATION_ERROR")

class ExternalServiceError(AppError):
    def __init__(self, service: str):
        super().__init__(f"{service} is unavailable", code="EXTERNAL_SERVICE_ERROR")
```

### Global Exception Handlers

Register all handlers in `main.py` — never handle exceptions inline in routes.

```python
# main.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse
from app.shared.errors import (
    NotFoundError, ConflictError, UnauthorizedError,
    ForbiddenError, ValidationError, ExternalServiceError
)

app = FastAPI()

@app.exception_handler(NotFoundError)
async def not_found_handler(request: Request, exc: NotFoundError):
    return JSONResponse(status_code=404, content={
        "success": False, "error": exc.code, "message": exc.message
    })

@app.exception_handler(ConflictError)
async def conflict_handler(request: Request, exc: ConflictError):
    return JSONResponse(status_code=409, content={
        "success": False, "error": exc.code, "message": exc.message
    })

@app.exception_handler(UnauthorizedError)
async def unauthorized_handler(request: Request, exc: UnauthorizedError):
    return JSONResponse(status_code=401, content={
        "success": False, "error": exc.code, "message": exc.message
    })

@app.exception_handler(ForbiddenError)
async def forbidden_handler(request: Request, exc: ForbiddenError):
    return JSONResponse(status_code=403, content={
        "success": False, "error": exc.code, "message": exc.message
    })

@app.exception_handler(Exception)
async def generic_handler(request: Request, exc: Exception):
    # Log the full exception internally
    logger.error(f"Unhandled exception: {exc}", exc_info=True)
    # Never expose internal details to client
    return JSONResponse(status_code=500, content={
        "success": False, "error": "INTERNAL_ERROR",
        "message": "An unexpected error occurred"
    })
```

### Error Response Format

All error responses follow the standard envelope:

```json
{
  "success": false,
  "data": null,
  "error": "ERROR_CODE",
  "message": "Human-readable description"
}
```

### Exception to HTTP Status Map

| Exception Class        | HTTP Status |
| ---------------------- | ----------- |
| `NotFoundError`        | 404         |
| `ConflictError`        | 409         |
| `UnauthorizedError`    | 401         |
| `ForbiddenError`       | 403         |
| `ValidationError`      | 422         |
| `ExternalServiceError` | 502         |
| Unhandled `Exception`  | 500         |

### What to Never Expose in Error Responses

- Stack traces
- Internal file paths or line numbers
- Database error messages or query details
- ORM model internals
- Environment variable names or values
- Third-party service error details verbatim

### Logging Rules

- Log ALL exceptions at ERROR level with full stack trace — internally only
- Log request ID and user ID with every error log for traceability
- Never log passwords, tokens, or PII in error logs
- Use structured logging (JSON format) for production

---

## Anti-Patterns

- Never raise `HTTPException` in the service layer — use custom exceptions
- Never return stack traces or internal paths in API responses
- Never use a single generic exception for all error types — be specific
- Never swallow exceptions silently — always log before handling
- Never return 200 status with an error body — use correct HTTP status codes
- Never expose database constraint violation messages directly to clients

---

## Ready-to-Use Prompt

```
Task: Implement error handling for [module or full project]
Skill: backend/error-handling, backend/rest-api

REQUIREMENTS:
- Scope: [single module / full project]
- Custom exceptions needed: [list error scenarios]
- External services that can fail: [list]

CONSTRAINTS:
- All custom exceptions extend AppError base class
- Global exception handlers registered in main.py only
- Service layer raises custom exceptions — never HTTPException
- Standard error envelope: success/data/error/message
- Never expose stack traces, DB errors, or internal paths in responses
- Log all exceptions internally with full stack trace

DONE WHEN:
- Custom exception classes defined in shared/errors.py
- Global handlers registered for all exception types
- Catch-all 500 handler logs internally but returns generic message
- All error responses follow standard envelope format
- Code review skill applied before commit
```
