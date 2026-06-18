# Architecture & Engineering Standards

This document defines how systems are structured, built, and scaled. It serves as the single source of truth for architecture, setup, and engineering standards across projects.

## System Structure

```text
project-root/
├── frontend/          # Next.js Client Application
├── auth/              # Node.js (ExpressKit)
├── backend/           # FastAPI Core Backend
├── worker/            # (optional) Background jobs
├── infra/             # Docker & deployment configs
├── docs/              # Documentation
├── .env.example
└── README.md
```

## Layer Rules & Request Flow

To maintain loose coupling and clarity over cleverness, the application layers must strictly adhere to these responsibilities:

- **Routes** → Endpoints only
- **Controllers** → Parsing only
- **Services** → ALL business logic
- **Models** → DB interaction only
- **Middleware** → Cross-cutting concerns (auth, logging)

### Standard Request Flow

```text
Frontend
   ↓
Auth Service (JWT Validation)
   ↓
Route
   ↓
Middleware
   ↓
Controller
   ↓
Service
   ↓
Database
```

## API Standard

All APIs must return this unified JSON response structure:

```json
{
 "success": true,
 "data": {},
 "error": null
}
```

## Code & Git Standards

### Code Rules
- **No `any`** in TypeScript.
- **Strict typing** in Python.
- Write **small functions**.
- **No silent failures** (handle or surface all errors).

### Git Workflow
Branch naming conventions:
- `feature/xyz`
- `fix/abc`
- `chore/cleanup`

## Anti-Patterns
DO NOT DO THE FOLLOWING:
- Business logic in controllers.
- Shared DB across services.
- Tight coupling between domains.
- Skipping input validation.

## Decision Framework

Before merging a PR or introducing a new pattern, ask:
1. Is it scalable?
2. Is it simple?
3. Is it readable?
4. What if it fails?

> **Final Note:** This system is designed so that any engineer can onboard quickly, code remains predictable, and features scale without rewrites.
