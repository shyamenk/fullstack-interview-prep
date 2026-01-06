# FastAPI Mastery

## Overview

FastAPI is a modern, fast (high-performance), web framework for building APIs with Python 3.6+ based on standard Python type hints. It is built on top of Starlette for the web parts and Pydantic for the data parts.

## Study Checklist

### Core Concepts
- [ ] **Path Parameters & Query Parameters** (Validation, Optional vs Required)
- [ ] **Request Body** (Pydantic Models, Nested Models)
- [ ] **Response Models** (Filtering data, Output validation)
- [ ] **Dependency Injection** (The `Depends` system)

### Advanced Features
- [ ] **Middleware** (CORS, Custom Middleware)
- [ ] **Background Tasks** (Post-response processing)
- [ ] **Security** (OAuth2 with Password/Bearer, JWT)
- [ ] **WebSockets** (Real-time communication)
- [ ] **Event Handlers** (Startup/Shutdown logic)

### Database & Testing
- [ ] **SQLAlchemy / SQLModel** Integration
- [ ] **Async Database Drivers** (asyncpg, motor)
- [ ] **Testing** with `TestClient` (Pytest)

### Deployment
- [ ] **Uvicorn** (ASGI Server)
- [ ] **Gunicorn** (Process Manager)
- [ ] **Docker** (Containerization)

## Quick Reference

| Feature | Description |
|---------|-------------|
| **Async/Await** | Native support for asynchronous code (`async def`) |
| **Pydantic** | Data validation and serialization using Python type hints |
| **Auto Docs** | Automatically generates Swagger UI (`/docs`) and ReDoc (`/redoc`) |
| **Dependency Injection** | Powerful system to manage dependencies (`Depends`) |
| **Fast** | Performance on par with NodeJS and Go (thanks to Starlette & Pydantic) |
| **Standard Based** | Based on open standards: OpenAPI (Swagger) and JSON Schema |

## Resources

- [Questions & Answers](./questions-and-answers.md)
- [Official Documentation](https://fastapi.tiangolo.com/)

## Progress Tracker

- [ ] Create a basic API with CRUD operations
- [ ] Implement JWT Authentication
- [ ] Connect to a Database (PostgreSQL/MongoDB)
- [ ] Write Unit Tests for endpoints
- [ ] Deploy using Docker and Uvicorn/Gunicorn
