# Product Requirements Document (PRD) - Todo Management Application

## 1. Executive Summary

This project aims to build a single-user Todo management application for local development, using a React 18 frontend and a FastAPI backend with SQL Server Express. The first delivery (Phase 1) focuses on Todo management, including create/read/update/delete (CRUD) operations for tasks with fixed priority and status values. Phase 2 introduces Category management and linking of Todos to categories.

Key outcomes:
- Personal task tracking for one user
- Simple, predictable interface and API
- Enterprise-grade backend patterns (Route-Service-Repository) and frontend patterns
- Testable, maintainable architecture with explicit non-goals to avoid scope creep

## 2. Goals and Non-Goals

### 2.1 Goals

- Build a fully functional Todo CRUD backend API and frontend UI (Phase 1)
- Implement category CRUD and todo-category assignment (Phase 2)
- Enforce data validation and error handling in API layer
- Provide a clean, intuitive user experience for personal productivity
- Keep the system local-workflow friendly (no authentication, single-user)

### 2.2 Non-Goals

- User authentication, login, registration
- Multi-user accounts or permissions
- Complex category hierarchies or shared tasks
- Mobile native apps (web only)
- Cloud deployment or multi-tenant infrastructure

## 3. User Personas

| Persona | Description | Needs |
|---|---|---|
| Solo Planner | A self-managed individual tracking personal tasks | Quick CRUD for todos, stable state, sorting by priority/status/due date |
| Batching Worker | Someone who enters and completes work in sessions | Fast bulk updates and status transitions |
| Overloaded Organizer | Needs visual organization by category once tasks expand | Category assignment and filtered lists (Phase 2) |

## 4. Functional Requirements

### 4.1 Phase 1: Todo Management (No Categories)

#### 4.1.1 Todo Entity

- Fields:
  - `id` (GUID/UUID or int PK)
  - `title` (string, required, max length 255)
  - `description` (string, optional)
  - `priority` (enum: Low, Medium, High)
  - `status` (enum: Pending, InProgress, Done)
  - `due_date` (date, optional)
  - `created_at` (timestamp)
  - `updated_at` (timestamp)

#### 4.1.2 Todo CRUD API

- GET `/api/v1/todos` : list all todos
- GET `/api/v1/todos/{id}` : retrieve single todo
- POST `/api/v1/todos` : create todo
- PUT `/api/v1/todos/{id}` : update todo
- DELETE `/api/v1/todos/{id}` : delete todo

Validation:
- `title` required
- `priority` only Low, Medium, High
- `status` only Pending, InProgress, Done
- `due_date` cannot be earlier than creation date (optional)

#### 4.1.3 Todo Frontend

- View todo list with columns: title, priority, status, due date
- Create / edit todo forms with fields above
- Controls for mark as done, delete, filter/sort by priority/status/due date
- Empty state UI when no todos exist

### 4.2 Phase 2: Category Management (+ Todo Category assignment)

#### 4.2.1 Category Entity

- Fields:
  - `id` (GUID/UUID or int PK)
  - `name` (string, required, unique, max length 100)
  - `created_at` (timestamp)
  - `updated_at` (timestamp)

#### 4.2.2 Category CRUD API

- GET `/api/v1/categories` : list categories
- GET `/api/v1/categories/{id}` : retrieve category
- POST `/api/v1/categories` : create category
- PUT `/api/v1/categories/{id}` : update category
- DELETE `/api/v1/categories/{id}` : delete category

#### 4.2.3 Todo-Category Behavior

- Add optional `category_id` on Todo (nullable)
- API include category info in todo details/list
- Frontend: assign category when creating/editing a todo
- Filter todo list by selected category

#### 4.2.4 Phase 2 Frontend Enhancements

- Category management view
- Todo form includes category dropdown
- Category filter control

## 5. Non-Functional Requirements

### 5.1 Performance

- API response times < 200ms for common operations on local machine
- Frontend render updates within 50ms for normal list sizes (<=200 todos)

### 5.2 Reliability

- 99% successful operation ratio in local workflow
- Data integrity with ACID operations via SQL Server transactions
- Graceful error handling (HTTP 400/404/422/500)

### 5.3 Maintainability

- Clean Route-Service-Repository pattern in backend
- 90%+ test coverage across backend and frontend
- Documentation in README and PRD
- Linting: black/flake8 (backend), ESLint/Prettier (frontend)

### 5.4 Security

- Input validation and explicit error codes
- CORS restricted to local origins only
- No secrets in code, .env for config

## 6. Data Requirements

### 6.1 Todo Table

- `todo` or `todos` table with columns above
- indexes on `priority`, `status`, `due_date`

### 6.2 Category Table

- `category` or `categories` table with columns above
- unique index on `name`

### 6.3 Relations

- `todos.category_id` foreign key to `categories.id` (nullable)
- on delete set null or restrict (phase decision: set null)

## 7. API Requirements Overview

- Base path: `/api/v1`
- JSON request/response for all endpoints
- Standard response envelope:
  - `{ "success": true, "data": ..., "error": null }`
  - on failure: `{ "success": false, "data": null, "error": {"code":"...","message":"..."}}`
- HTTP status mapping:
  - 200 OK (read/update)
  - 201 Created (create)
  - 204 No Content (delete)
  - 400 Bad Request (validation)
  - 404 Not Found
  - 422 Unprocessable Entity (schema validation)
  - 500 Internal Server Error

## 8. UI Requirements Overview

- Single-page Todo app
- Main view: todo list with controls for create/edit/delete
- Phase 1 details:
  - Todo creation/edit modal or inline form
  - Filters:
    - status tab/buttons (Pending/InProgress/Done)
    - priority tabs (Low/Medium/High)
    - due date sorting
- Phase 2 additions:
  - Category management panel (list, create, update, delete)
  - Category assignment dropdown in todo form
  - Category filter dropdown

### 8.1 Accessibility

- Use semantic HTML elements
- Keyboard navigation support
- ARIA labels for controls and forms

## 9. Out of Scope

- User authentication and session management
- Multiple users / multi-tenancy
- Due date reminders/notifications
- Offline synchronization, PWA offline support
- Advanced reporting or analytics
- Nested category hierarchies
- External integrations (email/calendar)
