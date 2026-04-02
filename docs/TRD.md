# Technical Requirements Document (TRD) - Todo Management Application

## 1. Architecture Overview

### 1.1 Architecture Summary

- Backend: Python 3.11, FastAPI, SQLAlchemy 2.x async
- Frontend: React 18, TypeScript, Vite, TanStack Query v5, Axios
- Database: SQL Server Express (localhost\SQLEXPRESS:1433)
- Layers: Route → Service → Repository (strict separation)
- Monorepo:
  - `/api` backend
  - `/web` frontend

### 1.2 Layer Diagram (text-based)

```
Web UI (React)
  └─ hooks/useTodos + useCategories (TanStack Query)
      └─ api/todo.api.ts + api/category.api.ts (Axios)
          └─ Backend API (/api/v1/*)
              └─ routes/ (FastAPI endpoints)
                  └─ services/ (business logic)
                      └─ repositories/ (SQLAlchemy async DB access)
                          └─ database (SQL Server Express)
```

### 1.3 Cross-cutting concerns

- CORS
- Logging
- Configuration via .env
- Error handling
- Validation
- Testing (unit, integration, edge-cases)

## 2. Complete Folder Structures

### 2.1 Backend: `/api`

```
api/
├── app/
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── todo.py
│   │   └── category.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── todo.py
│   │   └── category.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   ├── todo_repository.py
│   │   └── category_repository.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── todo_service.py
│   │   └── category_service.py
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── todo_router.py
│   │   └── category_router.py
│   └── exceptions/
│       ├── __init__.py
│       ├── errors.py
│       └── handlers.py
├── tests/
│   ├── conftest.py
│   ├── unit/
│   │   ├── test_todo_service.py
│   │   └── test_category_service.py
│   ├── integration/
│   │   ├── test_todo_endpoints.py
│   │   └── test_category_endpoints.py
│   └── migrations/
├── alembic/
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
├── alembic.ini
├── requirements.txt
└── .env
```

### 2.2 Frontend: `/web`

```
web/
├── src/
│   ├── api/
│   │   ├── client.ts
│   │   ├── todo.api.ts
│   │   └── category.api.ts
│   ├── hooks/
│   │   ├── useTodos.ts
│   │   └── useCategories.ts
│   ├── components/
│   │   ├── todo/
│   │   │   ├── TodoList.tsx
│   │   │   ├── TodoItem.tsx
│   │   │   └── TodoForm.tsx
│   │   └── category/
│   │       ├── CategoryList.tsx
│   │       ├── CategoryItem.tsx
│   │       └── CategoryForm.tsx
│   ├── types/
│   │   └── index.ts
│   ├── utils/
│   │   └── formatters.ts
│   ├── App.tsx
│   └── main.tsx
├── tests/
│   ├── setup.ts
│   ├── todo.test.tsx
│   └── category.test.tsx
├── vite.config.ts
├── tsconfig.json
├── vitest.config.ts
├── package.json
└── .env
```

## 3. API Contract

Base path: `/api/v1`

### 3.1 Todo API

| Method | URL | Request Body | Success Response | Success Status | Error Statuses |
|---|---|---|---|---|---|
| GET | /todos | none | `{success: true, data:[TodoSchema], error:null}` | 200 | 500, 404 |
| GET | /todos/{id} | none | `{success:true,data:TodoSchema,error:null}` | 200 | 404, 500 |
| POST | /todos | `CreateTodoSchema` | `{success:true,data:TodoSchema,error:null}` | 201 | 400, 422, 500 |
| PUT | /todos/{id} | `UpdateTodoSchema` | `{success:true,data:TodoSchema,error:null}` | 200 | 400, 404, 422, 500 |
| DELETE | /todos/{id} | none | `{success:true,data:null,error:null}` | 204 | 404, 500 |

#### 3.1.1 Todo Schemas

`TodoSchema`:
- id: int
- title: str
- description: str | null
- priority: str (Low|Medium|High)
- status: str (Pending|InProgress|Done)
- due_date: date | null
- category_id: int | null (Phase 2)
- created_at: datetime
- updated_at: datetime

`CreateTodoSchema`:
- title: str (required,max 200)
- description: str optional
- priority: str (Low,Medium,High) default Medium
- status: str (Pending,InProgress,Done) default Pending
- due_date: date optional
- category_id: int optional (Phase 2)

`UpdateTodoSchema`: same as create, all fields optional except title maybe optional depending process

### 3.2 Category API (Phase 2)

| Method | URL | Request Body | Success Response | Success Status | Error Statuses |
|---|---|---|---|---|---|
| GET | /categories | none | `{success:true,data:[CategorySchema],error:null}` | 200 | 500 |
| GET | /categories/{id} | none | `{success:true,data:CategorySchema,error:null}` | 200 | 404, 500 |
| POST | /categories | `CreateCategorySchema` | `{success:true,data:CategorySchema,error:null}` | 201 | 400, 422, 500 |
| PUT | /categories/{id} | `UpdateCategorySchema` | `{success:true,data:CategorySchema,error:null}` | 200 | 400, 404, 422, 500 |
| DELETE | /categories/{id} | none | `{success:true,data:null,error:null}` | 204 | 404, 500 |

`CategorySchema`:
- id: int
- name: str
- created_at: datetime
- updated_at: datetime

`CreateCategorySchema`:
- name: str required, max 100

`UpdateCategorySchema`:
- name: str required

## 4. Database Schema

### 4.1 Phase 1: Todos (no category)

Table `todos`:
- id INT IDENTITY(1,1) PRIMARY KEY
- title NVARCHAR(200) NOT NULL
- description NVARCHAR(MAX) NULL
- priority NVARCHAR(10) NOT NULL DEFAULT 'Medium' CHECK (priority IN ('Low','Medium','High'))
- status NVARCHAR(20) NOT NULL DEFAULT 'Pending' CHECK (status IN ('Pending','InProgress','Done'))
- due_date DATE NULL
- created_at DATETIME2 NOT NULL DEFAULT GETUTCDATE()
- updated_at DATETIME2 NOT NULL DEFAULT GETUTCDATE()

### 4.2 Phase 2: Add categories

Table `categories`:
- id INT IDENTITY(1,1) PRIMARY KEY
- name NVARCHAR(100) NOT NULL UNIQUE
- created_at DATETIME2 NOT NULL DEFAULT GETUTCDATE()
- updated_at DATETIME2 NOT NULL DEFAULT GETUTCDATE()

Alter `todos`:
- add category_id INT NULL
- add FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL

## 5. SQLAlchemy Model Definitions

### 5.1 Base model

`app/models/base.py`:

```python
from sqlalchemy import Column, DateTime, Integer, func
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class TimestampMixin:
    id = Column(Integer, primary_key=True, autoincrement=True)
    created_at = Column(DateTime(timezone=False), server_default=func.sysdate(), nullable=False)
    updated_at = Column(DateTime(timezone=False), server_default=func.sysdate(), onupdate=func.sysdate(), nullable=False)
```

### 5.2 Todo model

`app/models/todo.py`:

```python
from sqlalchemy import Column, CheckConstraint, Date, Enum, String
from sqlalchemy.orm import relationship
from app.models.base import Base, TimestampMixin

class PriorityEnum(str, Enum):
    Low = 'Low'
    Medium = 'Medium'
    High = 'High'

class StatusEnum(str, Enum):
    Pending = 'Pending'
    InProgress = 'InProgress'
    Done = 'Done'

class Todo(Base, TimestampMixin):
    __tablename__ = 'todos'

    title = Column(String(200), nullable=False)
    description = Column(String, nullable=True)
    priority = Column(String(10), nullable=False, default='Medium')
    status = Column(String(20), nullable=False, default='Pending')
    due_date = Column(Date, nullable=True)
    category_id = Column(Integer, ForeignKey('categories.id'), nullable=True)

    __table_args__ = (
        CheckConstraint("priority IN ('Low', 'Medium', 'High')"),
        CheckConstraint("status IN ('Pending', 'InProgress', 'Done')"),
    )

    category = relationship('Category', back_populates='todos', lazy='selectin')
```

### 5.3 Category model

`app/models/category.py`:

```python
from sqlalchemy import Column, String
from sqlalchemy.orm import relationship
from app.models.base import Base, TimestampMixin

class Category(Base, TimestampMixin):
    __tablename__ = 'categories'

    name = Column(String(100), nullable=False, unique=True)
    todos = relationship('Todo', back_populates='category', lazy='selectin')
```

## 6. Alembic Strategy

### 6.1 Initial migration

1. Set SQLAlchemy URL in `alembic.ini` as env var `SQLALCHEMY_DATABASE_URL` in `alembic.env.py`.
2. In `alembic/env.py`, import all models so autogenerate sees metadata.
3. Generate revision:
   - `alembic revision --autogenerate -m "create todos table"`
4. Apply migration:
   - `alembic upgrade head`

### 6.2 Phase 2 migration

1. Add category model to ORM and metadata.
2. Generate revision:
   - `alembic revision --autogenerate -m "add categories table and todo.category_id fk"`
3. Review generated script:
   - create categories table
   - alter todos add category_id
   - add fk constraint ON DELETE SET NULL
4. Apply:
   - `alembic upgrade head`

### 6.3 Rollback

- Use `alembic downgrade -1` for previous step
- Always validate data integrity after downgrade

## 7. Environment Variables Reference

| Name | Purpose | Example | Required |
|---|---|---|---|
| SQLALCHEMY_DATABASE_URL | DB connection string | `mssql+aioodbc:///?odbc_connect=DRIVER={ODBC Driver 17 for SQL Server};SERVER=localhost\SQLEXPRESS;Database=TodoDb;Trusted_Connection=yes;` | yes |
| TEST_DATABASE_URL | Test DB connection string | `sqlite+aiosqlite:///:memory:` or SQL Server test DB | yes |
| VITE_API_URL | Frontend API base URL | `http://localhost:8000/api/v1` | yes |
| CORS_ORIGINS | Allowed origins (comma-delimited) | `http://localhost:5173` | yes |
| APP_ENV | app environment | `development` | yes |
| LOG_LEVEL | logging level | `INFO` | yes |

## 8. Error Response Standard

All endpoints return a consistent envelope:

- Success:

```json
{
  "success": true,
  "data": { ... },
  "error": null
}
```

- Error:

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "TODO_NOT_FOUND",
    "message": "Todo with id 123 not found",
    "details": null
  }
}
```

HTTP status mapping:
- 200 -> OK
- 201 -> Created
- 204 -> No Content
- 400 -> Bad Request
- 404 -> Not Found
- 422 -> Unprocessable Entity
- 500 -> Internal Server Error

## 9. Testing Architecture

### 9.1 Backend tests

- Unit tests (api/tests/unit): service + repository with mocks
  - Use `unittest.mock.AsyncMock` for async repository methods
- Integration tests (api/tests/integration): API endpoints via `httpx.AsyncClient`
- Fixtures in `api/tests/conftest.py`:
  - db_session AsyncSession test database
  - test_app with dependency overrides
  - async_client
- Test cases:
  - happy path
  - missing required input
  - invalid enums
  - not found
  - DB constraint violation
- For services, mock repository layer (no DB access in unit service tests)

### 9.2 Frontend tests

- Tests in `web/tests` run via Vitest
- Use React Testing Library and MSW for API mocking
- Custom render util with QueryClientProvider
- Cover:
  - initial list fetch
  - create/update/delete operations
  - loading and error states
  - form validation behavior

### 9.3 Edge cases

- Null/empty values and invalid enums in API
- Network failures in frontend UI
- Database constraint violations
- stale or deleted category references

## 10. CORS Configuration

### Backend `app/main.py` or `app/config.py`

- Allowed origin: `http://localhost:5173` (Vite dev)
- Disallow wildcard in production by default
- Enable CORS middleware:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[settings.cors_origins],
    allow_credentials=False,
    allow_methods=["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    allow_headers=["*"],
)
```

- Document CORS_ORIGINS in environment variable list.

## 11. Notes

- For quick local setup, use `.env`:
  - SQLALCHEMY_DATABASE_URL=...
  - VITE_API_URL=http://localhost:8000/api/v1
  - CORS_ORIGINS=http://localhost:5173
- Keep backend and frontend config consistent (API URL value).
