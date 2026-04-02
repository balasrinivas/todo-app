---
name: security-review
description: >-
  Skill for security review in ToDo App monorepo. Covers FastAPI/SQLAlchemy
  backend and React/Axios frontend security checks, vulnerability ratings,
  and safe remediation code patterns.
triggers:
  - "security review"
  - "check for vulnerabilities"
  - "security audit"
  - "review for security"
  - "check security"
---

# Security Review Skill for ToDo App Monorepo

This skill is auto-loaded for security-related prompts. It evaluates backend and
frontend code and returns categorized findings plus corrected sample code.

## Review Output Format

Category | Location | Issue | Risk | Recommended Fix
---|---|---|---|---
CRITICAL | file:line | SQL injection via raw SQL | Data breach / unauthorized access | Use SQLAlchemy ORM/parameterized queries in repository layer

> Include corrected code snippets for every issue.

## Backend Security Checklist (FastAPI + SQLAlchemy Async)

- SQL injection risk
  - Avoid raw SQL in repository methods
  - Use parameterized queries and `text` with bound params if needed
  - Prefer ORM query APIs (select(), where(), etc.)
- Input validation
  - Ensure Pydantic request schemas have field constraints
  - Validate query params with `Query(...)` and path params with `Path(...)`
- Error handling
  - Do not leak stack traces in response
  - Use exception handlers (HTTPException) with generic messages
  - Log details server-side at ERROR level
- CORS configuration
  - Do not use `allow_origins=['*']` in production
  - Use specific allowed origins from env config
- Sensitive data exposure
  - Never send DB connection strings, secrets in API response
- Insecure dependencies
  - Review requirements.txt for known CVEs and fixed versions
- Authentication/authorization
  - Although not required, ensure no security assumptions allow access to sensitive data patterns or admin operations

## Frontend Security Checklist (React + Axios)

- XSS
  - Avoid `dangerouslySetInnerHTML`
  - Sanitize API data before rendering if using HTML content
- Data handling
  - Do not store sensitive data in localStorage/sessionStorage
  - Do not expose tokens or secrets in client code
- Input validation
  - Validate user-entered values before API calls
- API response handling
  - Treat all API data as untrusted and encode/sanitize in component output
- Hardcoded secrets/URLs
  - Use import.meta.env values for API endpoints, no constants in code
- Dependency security
  - Audit package versions for known vulnerabilities

## CORS Configuration Review Steps

1. Open backend config file, usually `app/config.py` or `main.py`.
2. Locate `CORSMiddleware` setup.
3. Ensure `allow_origins` is environment-driven and not wildcard in prod:
   - Good: `['https://todo-app.local', 'https://app.company.com']`
   - Bad: `['*']`
4. Check `allow_credentials=False` unless necessary; avoid exposing cookies.
5. Validate `allow_methods` and `allow_headers` include only expected values.
6. Verify `expose_headers` only includes needed headers.

## Common Fix Patterns

### FastAPI: SQL Injection Secure Repository Example

```python
from sqlalchemy import select
from app.models.todo import Todo
from sqlalchemy.ext.asyncio import AsyncSession

class TodoRepository:
    async def get_by_title(self, db: AsyncSession, title: str) -> list[Todo]:
        query = select(Todo).where(Todo.title == title)
        result = await db.execute(query)
        return result.scalars().all()
```

### FastAPI: Exception Handling Example

```python
from fastapi import HTTPException
from fastapi.responses import JSONResponse

@app.exception_handler(Exception)
async def global_exception_handler(request, exc):
    logger.error('Unhandled exception', exc_info=exc)
    return JSONResponse(
        status_code=500,
        content={'error': 'Internal server error'},
    )
```

### React: Prevent XSS Example

```tsx
import React from 'react'

interface TodoProps { data: string }

export function TodoItem({ data }: TodoProps) {
  return <div>{data}</div> // bare text, auto-escaped by React
}
```

### React: Secure Axios Base URL

```ts
import axios from 'axios'

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
})
```

## Finding Severity Guide

- CRITICAL: exploitable injection or data exfiltration risk
- HIGH: broken auth, RCE, mass data disclosure
- MEDIUM: validation gaps, insecure defaults, moderate logic flaws
- LOW: missing headers, non-sensitive info leaks, dev conveniences
- INFO: recommendations, maintenance tasks
