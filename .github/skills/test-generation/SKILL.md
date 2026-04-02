---
name: test-generation
description: >-
  Skill for generating and updating tests in the ToDo App monorepo.
  Includes backend pytest/pytest-asyncio/httpx patterns and frontend
  vitest/React Testing Library/MSW patterns.
triggers:
  - "write tests"
  - "add tests"
  - "generate test cases"
  - "update tests"
  - "create unit tests"
  - "add integration tests"
  - "testing"
  - "test coverage"
---

# Test Generation Skill for ToDo App Monorepo

This skill is loaded automatically when requests relate to test creation or
updates in either backend (/api) or frontend (/web).

## High-Level Testing Philosophy

- Follow **Arrange / Act / Assert**.
- One behavior per test (one assertion concept).
- Descriptive, self-documenting names:
  `test_should_<expected>_when_<condition>`.
- Cover happy path, error scenarios, boundary/error states, null/empty.
- No implementation details in frontend tests (user behavior only).
- Mock at repository layer for backend service tests.

## Backend (api) Testing Patterns

### Toolstack

- pytest
- pytest-asyncio
- httpx AsyncClient for FastAPI endpoint tests
- unittest.mock.AsyncMock for mocking async repository methods
- test DB from `.env.test` (SQLite in-memory or SQL Server test DB)

### Folder Layout

- `api/tests/unit/`
- `api/tests/integration/`
- `api/tests/conftest.py`

### `conftest.py` Patterns

- `event_loop` fixture (pytest-asyncio)
- `async_client` fixture for `AsyncClient(app)`
- `db_session` fixture for `AsyncSession` against test DB
- `mock_repository` fixture using `AsyncMock`

### Service Layer Tests

- Always mock repository layer.
- No DB calls in unit service tests.

Example:

```python
import pytest
from unittest.mock import AsyncMock

from app.services.todo_service import TodoService
from app.exceptions import TodoNotFoundError
from app.models import Todo

@pytest.mark.asyncio
async def test_should_return_todo_when_id_exists():
    # Arrange
    fake_todo = Todo(id=1, title='Test', completed=False)
    repo = AsyncMock()
    repo.get_by_id.return_value = fake_todo
    service = TodoService(repository=repo)

    # Act
    result = await service.get_todo_by_id(1)

    # Assert
    assert result.id == 1
    assert result.title == 'Test'

@pytest.mark.asyncio
async def test_should_raise_not_found_when_todo_missing():
    # Arrange
    repo = AsyncMock()
    repo.get_by_id.return_value = None
    service = TodoService(repository=repo)

    # Act / Assert
    with pytest.raises(TodoNotFoundError):
        await service.get_todo_by_id(999)
```

### Route / API Integration Tests

- Use `httpx.AsyncClient(app=app, base_url='http://test')`.
- Seed test DB using fixtures.

Example:

```python
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_should_create_todo_returns_201(async_client):
    # Arrange
    payload = {'title': 'New Task', 'completed': False}

    # Act
    response = await async_client.post('/api/v1/todos', json=payload)

    # Assert
    assert response.status_code == 201
    body = response.json()
    assert body['data']['title'] == 'New Task'
    assert body['data']['completed'] is False
```

### Exception and Error Cases

- Always assert correct HTTP status and error message.
- Test validation path on missing required fields -> 422.
- Test not-found path -> 404.
- Test database exception path -> 500.

## Frontend (web) Testing Patterns

### Toolstack

- vitest
- @testing-library/react (React Testing Library)
- MSW for API mocking
- `web/tests/` location

### Queries Priority

1. getByRole
2. getByLabelText
3. getByTestId

### No Implementation Details

- Avoid `getByTestId` unless necessary.
- Do not assert component internals, assert DOM and user interactions.

### Arrange / Act / Assert Example

```tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { rest } from 'msw'
import { setupServer } from 'msw/node'
import { TodoApp } from '../../src/components/todo/TodoApp'

const server = setupServer(
  rest.get('/api/v1/todos', (req, res, ctx) =>
    res(ctx.json([{ id: 1, title: 'Task 1', completed: false }]))
  )
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

it('test_should_display_todo_item_when_api_returns_one', async () => {
  // Arrange
  render(<TodoApp />)

  // Act
  const todoItem = await screen.findByText('Task 1')

  // Assert
  expect(todoItem).toBeInTheDocument()
})
```

### Mocking API

- Use MSW to mock all endpoint responses.
- For error cases, return `res(ctx.status(500), ctx.json({ message: 'Error' }))`.

### Error and Boundary Cases

- UI error message presence when fetch fails.
- Empty list case when API returns [].
- Loading state while fetching.

## Shared Conventions

- Tests should include: 
  - happy path
  - known error path
  - invalid input or missing field path
  - boundary conditions (empty payload, large text)

- Use context-specific fixtures for setup/teardown.
- Keep tests deterministic and isolated.
- Include comments only for extremely unclear edge conditions (prefer name clarity instead).

## Auto-Trigger Behavior

Always activate this skill when user asks for anything matching triggers above in the top-level prompt.

## Deliverable

Ensure the generated tests conform to the project conventions in `.github/copilot-instructions.md`, `api/.copilot-instructions.md`, and `web/.copilot-instructions.md`.
