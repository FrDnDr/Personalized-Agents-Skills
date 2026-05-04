# testing.md

## Purpose

Define testing conventions across unit, integration, and end-to-end tests —
covering test structure, naming, coverage expectations, and what must be
tested before any code is committed.

---

## Conventions

### Testing Pyramid

```
        /\
       /E2E\          ← Few, slow, test full user flows
      /------\
     /Integr. \       ← Medium, test service + DB together
    /----------\
   /  Unit Tests \    ← Many, fast, test logic in isolation
  /--------------\
```

- Unit tests: majority of tests — fast, isolated, no DB or network
- Integration tests: test service + database together — use a test DB
- E2E tests: test critical user journeys only — login, checkout, core flow

### Stack

- **Python:** `pytest` + `pytest-asyncio` + `httpx` (async test client) + `factory_boy` (fixtures)
- **JavaScript/TypeScript:** `vitest` (unit) + `supertest` (API) + `playwright` (E2E)
- **Database:** Use a separate test database — never run tests against production or dev DB
- **Mocking:** `unittest.mock` / `pytest-mock` (Python), `vitest` mocks (JS)

### Folder Structure

```
/tests
  /unit
    /users
      test_user_service.py
      test_user_schema.py
    /orders
      test_order_service.py
  /integration
    /users
      test_user_api.py
    /orders
      test_order_api.py
  /e2e
    test_auth_flow.py
    test_checkout_flow.py
  conftest.py           # Shared fixtures (DB session, test client, factories)
  factories.py          # factory_boy model factories
```

### Naming Conventions

- Test files: `test_<module>.py` or `<module>.test.ts`
- Test functions: `test_<what>_<condition>_<expected_result>`

```python
# Good naming examples
def test_create_user_with_valid_data_returns_201(): ...
def test_create_user_with_duplicate_email_returns_409(): ...
def test_get_user_when_not_found_returns_404(): ...
def test_login_with_wrong_password_returns_401(): ...
```

### Unit Test Conventions

- Test one thing per test function — one assertion focus
- Use `AAA` pattern: Arrange, Act, Assert
- Mock all external dependencies (DB, APIs, email, Redis)
- Tests must be deterministic — no randomness, no time.now() without mocking

```python
# Example unit test — service layer
import pytest
from unittest.mock import AsyncMock, MagicMock
from app.api.v1.users.service import UserService
from app.shared.errors import ConflictError

@pytest.mark.asyncio
async def test_create_user_with_duplicate_email_raises_conflict():
    # Arrange
    mock_db = AsyncMock()
    service = UserService(mock_db)
    service._get_by_email = AsyncMock(return_value=MagicMock())  # Simulate existing user

    # Act & Assert
    with pytest.raises(ConflictError):
        await service.create(UserCreate(email="existing@test.com", password="Test1234"))
```

### Integration Test Conventions

- Use a real test database — spin up with Docker or use SQLite in-memory for speed
- Reset database state between tests — use transactions that rollback or truncate tables
- Test the full request → response cycle via the HTTP client
- Test both happy path and error cases for every endpoint

```python
# Example integration test — API layer
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_create_user_returns_201(client: AsyncClient):
    response = await client.post("/api/v1/users/", json={
        "email": "new@test.com",
        "password": "Test1234!"
    })
    assert response.status_code == 201
    assert response.json()["success"] is True
    assert "id" in response.json()["data"]

@pytest.mark.asyncio
async def test_create_user_duplicate_email_returns_409(client: AsyncClient):
    await client.post("/api/v1/users/", json={"email": "dup@test.com", "password": "Test1234!"})
    response = await client.post("/api/v1/users/", json={"email": "dup@test.com", "password": "Test1234!"})
    assert response.status_code == 409
    assert response.json()["error"] == "CONFLICT"
```

### Coverage Expectations

| Layer         | Minimum Coverage |
| ------------- | ---------------- |
| Service layer | 90%              |
| API endpoints | 85%              |
| Utilities     | 80%              |
| Overall       | 80%              |

- Run coverage before every PR: `pytest --cov=app --cov-report=term-missing`
- Coverage below minimum blocks the commit

### What Must Always Be Tested

For every new endpoint or feature:

- [ ] Happy path (valid input, expected output)
- [ ] Invalid input (missing fields, wrong types, out of range values)
- [ ] Auth errors (401 for unauthenticated, 403 for wrong role)
- [ ] Not found (404 when resource doesn't exist)
- [ ] Conflict (409 when duplicate resource)
- [ ] Edge cases specific to the feature logic

### Fixtures and Factories

- Define shared fixtures in `conftest.py`
- Use `factory_boy` to generate test data — never hardcode test data inline
- Create one factory per model

```python
# factories.py
import factory
from app.models.user import User

class UserFactory(factory.Factory):
    class Meta:
        model = User

    id = factory.Faker("uuid4")
    email = factory.Faker("email")
    name = factory.Faker("name")
    role = "user"
```

---

## Anti-Patterns

- Never test implementation details — test behavior and outcomes
- Never share state between tests — each test must be fully independent
- Never write tests that depend on execution order
- Never skip error case tests — only testing the happy path is insufficient
- Never use production or dev database for tests
- Never hardcode test data inline — use factories
- Never mock what you own — mock external dependencies only
- Never commit code with failing tests

---

## Ready-to-Use Prompt

```
Task: Write tests for [module, endpoint, or feature]
Skill: core/testing, backend/rest-api, database/schema-design

REQUIREMENTS:
- Test type: [unit / integration / e2e / all]
- Module: [describe what's being tested]
- Coverage target: [service: 90% / endpoints: 85%]

TEST CASES TO COVER:
- [ ] Happy path
- [ ] Invalid input (missing fields, wrong types)
- [ ] Auth errors (401, 403)
- [ ] Not found (404)
- [ ] Conflict or duplicate (409)
- [ ] Edge cases: [describe specific ones]

CONSTRAINTS:
- AAA pattern: Arrange, Act, Assert
- One assertion focus per test
- Mock all external dependencies in unit tests
- Use test DB for integration tests — never production
- Use factories for test data — no hardcoded values
- Tests must be deterministic and order-independent

DONE WHEN:
- All test cases listed above covered
- Coverage meets minimum thresholds
- All tests pass: pytest --cov=app
- Code review skill applied before commit
```
