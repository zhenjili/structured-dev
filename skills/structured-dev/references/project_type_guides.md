# Project Type Guides

Reference for adapting feature list generation to different project types.

## Table of Contents
- [Web App](#web-app)
- [API](#api)
- [Mobile](#mobile)
- [CLI](#cli)
- [Library](#library)
- [Full-Stack](#full-stack)

---

## Web App

Single-page or multi-page web applications with a frontend focus.

**Typical scale**: 30-80 features
**Tech signals**: React, Vue, Svelte, Next.js, Angular, HTML/CSS/JS

### Category Distribution

| Category | Share | Notes |
|----------|-------|-------|
| setup | 5-10% | Build tooling, dev server, linting |
| core | 20-30% | Business logic, state management |
| ui | 30-40% | Components, pages, responsive design |
| api | 5-10% | API client, data fetching |
| data | 5-10% | Local storage, caching, state persistence |
| integration | 5-10% | Auth providers, analytics, CDN |
| testing | 10-15% | Component tests, E2E tests |
| polish | 5-10% | Animations, loading states, error boundaries |

### Common Commands
```json
{
  "test_command": "npm test",
  "build_command": "npm run build",
  "run_command": "npm run dev"
}
```

### Tips
- Start with routing and layout structure as setup features
- Break pages into individual features (one per page/view)
- Form validation and error states are separate features from the happy path
- Accessibility (keyboard nav, screen reader) belongs in polish

---

## API

Backend services exposing REST, GraphQL, or gRPC endpoints.

**Typical scale**: 25-60 features
**Tech signals**: FastAPI, Express, Django, Rails, Go net/http, Spring Boot

### Category Distribution

| Category | Share | Notes |
|----------|-------|-------|
| setup | 5-10% | Server scaffolding, middleware, config |
| core | 25-35% | Business logic, domain models |
| api | 25-35% | Endpoints, request/response handling |
| data | 15-20% | Database schema, migrations, queries |
| integration | 5-10% | External APIs, message queues |
| testing | 10-15% | API tests, integration tests |
| polish | 5% | Rate limiting, error responses, docs |

### Common Commands
```json
{
  "test_command": "pytest --tb=short",
  "build_command": "docker compose build",
  "run_command": "uvicorn app.main:app --reload"
}
```

### Tips
- One feature per resource CRUD is too coarse; split into create, read, update, delete
- Auth features come before any authenticated endpoints
- Database migrations are their own feature
- API documentation (OpenAPI/Swagger) can be a polish feature

---

## Mobile

iOS, Android, or cross-platform mobile apps.

**Typical scale**: 30-80 features
**Tech signals**: Flutter, React Native, SwiftUI, Kotlin/Jetpack Compose

### Category Distribution

| Category | Share | Notes |
|----------|-------|-------|
| setup | 5-10% | Project config, signing, dependencies |
| core | 20-30% | Business logic, models, services |
| ui | 30-40% | Screens, navigation, components |
| api | 10-15% | Backend communication, API client |
| data | 10-15% | Local DB, caching, offline support |
| integration | 5-10% | Push notifications, camera, location |
| testing | 5-10% | Widget/unit tests |
| polish | 5-10% | Animations, haptics, edge cases |

### Common Commands
```json
{
  "test_command": "flutter test",
  "build_command": "flutter build ios --debug",
  "run_command": "flutter run"
}
```

### Tips
- Navigation structure is a setup feature
- Each screen is typically 2-4 features (display, interaction, edge cases)
- Platform-specific features (iOS/Android) should be clearly labeled
- Offline capability deserves its own feature per data type

---

## CLI

Command-line tools and utilities.

**Typical scale**: 10-30 features
**Tech signals**: Click, argparse, Commander.js, clap (Rust), cobra (Go)

### Category Distribution

| Category | Share | Notes |
|----------|-------|-------|
| setup | 10-15% | Argument parsing, config loading |
| core | 40-50% | Main functionality, commands |
| data | 10-15% | File I/O, config persistence |
| integration | 5-10% | External tools, APIs |
| testing | 15-20% | Unit tests, integration tests |
| polish | 10-15% | Help text, error messages, colors |

### Common Commands
```json
{
  "test_command": "pytest",
  "build_command": "pip install -e .",
  "run_command": "mycli --help"
}
```

### Tips
- Each subcommand is its own feature
- Flag/option combinations are separate features from the base command
- Pipe/stdin support is a feature
- Exit codes and error messages are polish features

---

## Library

Reusable packages, SDKs, and frameworks.

**Typical scale**: 15-50 features
**Tech signals**: Published to npm/PyPI/crates.io, has public API surface

### Category Distribution

| Category | Share | Notes |
|----------|-------|-------|
| setup | 10-15% | Package config, build system, CI |
| core | 40-50% | Public API functions/classes |
| data | 5-10% | Data structures, serialization |
| testing | 20-25% | Unit tests, doc tests, examples |
| polish | 10-15% | Documentation, type definitions, error messages |

### Common Commands
```json
{
  "test_command": "npm test",
  "build_command": "npm run build",
  "run_command": "node examples/basic.js"
}
```

### Tips
- Each public API method/function is its own feature
- Type definitions and documentation are features, not afterthoughts
- Error handling for invalid inputs is its own feature per method
- Example code that demonstrates usage is a testing/polish feature
- No ui or api categories typically

---

## Full-Stack

Applications with tightly coupled frontend and backend.

**Typical scale**: 50-150 features
**Tech signals**: Next.js full-stack, Django + templates, Rails, MERN/MEAN stack

### Category Distribution

| Category | Share | Notes |
|----------|-------|-------|
| setup | 5% | Monorepo config, shared types, dev environment |
| core | 20-25% | Business logic (shared between FE/BE) |
| ui | 20-25% | Frontend components and pages |
| api | 15-20% | Backend endpoints, middleware |
| data | 10-15% | Database, ORM, migrations |
| integration | 5-10% | Third-party services |
| testing | 10-15% | Unit, integration, E2E |
| polish | 5-10% | Cross-cutting concerns |

### Common Commands
```json
{
  "test_command": "npm run test:all",
  "build_command": "npm run build",
  "run_command": "npm run dev"
}
```

### Tips
- Feature a vertical slice when possible: "User can sign up" covers API endpoint + UI form + database
- But split into separate features when the backend and frontend are complex individually
- Shared types/interfaces between FE/BE are a setup feature
- Database migrations should be separate features from the API that uses them
- E2E tests that cross the stack are integration or testing features
