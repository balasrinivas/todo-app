# GitHub Copilot Instructions for ToDo App Monorepo

## Project Overview

This monorepo contains a full-stack ToDo application with:
- **Backend (/api)**: Python FastAPI REST API
- **Frontend (/web)**: React 18 TypeScript application
- **Database**: SQL Server Express (localhost:1433, Windows Authentication)
- **Architecture**: Route → Service → Repository pattern with strict separation of concerns
- **ORM**: SQLAlchemy 2.x async with Alembic migrations
- **Authentication**: None required (single-user application)

All code must adhere to enterprise production standards with comprehensive documentation, proper error handling, and maintainable architecture.

## Architecture Rules

### Backend (Python/FastAPI)
- **Layer Separation**: Strictly enforce Route → Service → Repository pattern
  - Routes handle HTTP requests/responses only
  - Services contain business logic
  - Repositories handle data access
- **Async Operations**: All database operations must use async/await with SQLAlchemy 2.x
- **Dependency Injection**: Use FastAPI's dependency injection system
- **No Direct Database Access**: All data operations through Repository layer

### Frontend (React/TypeScript)
- **Component Structure**: Functional components with hooks
- **State Management**: React hooks for local state, Context API for global state if needed
- **API Integration**: Centralized API client with proper error handling
- **Type Safety**: Strict TypeScript usage with no `any` types

### Shared Rules
- **Configuration**: All configuration via environment variables (.env files)
- **No Hardcoded Values**: Database connections, API endpoints, timeouts, etc. must be configurable
- **Secrets Management**: All sensitive data in .env files, never committed to git

## Coding Standards (Shared)

### Documentation
- **Python**: Every function, class, and module must have comprehensive docstrings following Google style
- **TypeScript**: Every function and component must have JSDoc comments
- **Self-Documenting Code**: Use descriptive variable/function names; avoid abbreviations
- **No Comments for Obvious Code**: Code should be readable without additional comments

### Error Handling
- **Explicit Handling**: Never swallow exceptions silently
- **Custom Exceptions**: Define specific exception types for business logic errors
- **Logging**: Use proper logging frameworks (Python: logging module, TypeScript: console with structured logging)
- **No print()**: Replace all print statements with appropriate logging

### Async Programming
- **Python**: Use async/await for all I/O operations, database queries, and API calls
- **TypeScript**: Use async/await for all API calls and asynchronous operations
- **Proper Error Propagation**: Async functions must handle and propagate errors correctly

### Code Quality
- **Linting**: Follow project-specific linting rules (black/flake8 for Python, ESLint for TypeScript)
- **Type Safety**: Strict typing in both languages
- **Imports**: Organize imports alphabetically, separate standard library, third-party, and local imports
- **Constants**: Define magic numbers and strings as named constants

## Git Workflow

### Branching Strategy
- **Main Branch**: `main` - production-ready code only
- **Feature Branches**: `feature/description-of-feature` - all new development
- **Bug Fixes**: `fix/description-of-bug`
- **No Direct Commits**: All changes merged via Pull Requests

### Commit Messages
- **Conventional Commits**: Always use format `type(scope): description`
  - `feat:` - new features
  - `fix:` - bug fixes
  - `test:` - testing related changes
  - `refactor:` - code refactoring
  - `chore:` - maintenance tasks
  - `docs:` - documentation updates
- **Descriptive**: Include what changed and why
- **Atomic**: Each commit should contain a single logical change

### Pull Requests
- **Code Review**: Required for all changes
- **Tests**: All tests must pass before merge
- **Documentation**: Update docs for API changes
- **Squash Merges**: Use squash merge to maintain clean history

## Error Handling Strategy

### Backend (FastAPI)
- **HTTP Status Codes**:
  - `201 Created` - successful resource creation
  - `204 No Content` - successful deletion
  - `400 Bad Request` - malformed request
  - `404 Not Found` - resource not found
  - `422 Unprocessable Entity` - validation errors
  - `500 Internal Server Error` - unexpected server errors
- **Response Models**: Use Pydantic models for all API responses
- **Validation**: Leverage Pydantic for request validation
- **Exception Handlers**: Global exception handlers for consistent error responses

### Frontend (React)
- **API Errors**: Handle HTTP errors gracefully with user-friendly messages
- **Loading States**: Show loading indicators during async operations
- **Retry Logic**: Implement retry for transient failures
- **Error Boundaries**: Use React Error Boundaries for component-level error handling

### Shared
- **Logging**: Log all errors with appropriate levels (ERROR, WARNING, INFO)
- **User Feedback**: Provide clear error messages to users
- **Monitoring**: Design for observability (logs, metrics, traces)

## Testing Strategy

### Backend (Python)
- **Unit Tests**: Test individual functions and classes
- **Integration Tests**: Test service and repository layers
- **API Tests**: Test endpoints with FastAPI TestClient
- **Database Tests**: Use test database with fixtures
- **Coverage**: Aim for >90% code coverage
- **Async Testing**: Use pytest-asyncio for async test functions

### Frontend (TypeScript)
- **Unit Tests**: Test components and utilities with Jest/React Testing Library
- **Integration Tests**: Test user interactions and API calls
- **E2E Tests**: Use Playwright for critical user journeys
- **Mocking**: Mock API calls and external dependencies

### Shared
- **Test Naming**: `test_function_name_describes_what_it_tests`
- **Arrange-Act-Assert**: Follow AAA pattern in all tests
- **CI/CD**: All tests run in CI pipeline before merge
- **Test Data**: Use realistic test data, avoid magic numbers

## API Design Standards

### RESTful Design
- **Resource-Based URLs**: `/todos`, `/todos/{id}`
- **HTTP Methods**: GET, POST, PUT, DELETE appropriately
- **Consistent Naming**: Use plural nouns for resources
- **Versioning**: Include API version in URL path (`/api/v1/todos`)

### Request/Response Format
- **JSON**: All requests and responses in JSON format
- **Consistent Structure**: Standard response envelope with data, errors, metadata
- **Pagination**: Implement cursor-based pagination for list endpoints
- **Filtering/Sorting**: Support query parameters for filtering and sorting

### Validation
- **Input Validation**: Validate all inputs at API boundary
- **Business Rules**: Validate business rules in service layer
- **Error Messages**: Provide specific, actionable error messages

### Performance
- **Database Optimization**: Use indexes, avoid N+1 queries
- **Caching**: Implement appropriate caching strategies
- **Rate Limiting**: Consider rate limiting for public endpoints
- **Async Processing**: Use background tasks for long-running operations

### Security
- **Input Sanitization**: Sanitize all user inputs
- **CORS**: Configure CORS appropriately
- **HTTPS**: Enforce HTTPS in production
- **Data Validation**: Never trust client-side validation alone 
