# feature_list.json Schema

Complete schema reference for the feature tracking file.

## Table of Contents
- [Top-Level Structure](#top-level-structure)
- [Project Section](#project-section)
- [Feature Object](#feature-object)
- [Categories](#categories)
- [Metadata Section](#metadata-section)
- [Immutability Rules](#immutability-rules)
- [Adaptive Sizing Guide](#adaptive-sizing-guide)
- [Example: Small Project](#example-small-project)
- [Example: Medium Project (excerpt)](#example-medium-project-excerpt)

---

## Top-Level Structure

```json
{
  "project": { ... },
  "features": [ ... ],
  "metadata": { ... }
}
```

## Project Section

Project-level metadata and environment configuration. This section replaces the need for a separate `init.sh` script.

```json
{
  "project": {
    "name": "my-project",
    "description": "Brief description of what the project does",
    "type": "web-app",
    "created": "2026-03-03T10:00:00Z",
    "tech_stack": ["python", "fastapi", "react", "postgresql"],
    "test_command": "pytest",
    "build_command": "npm run build",
    "run_command": "uvicorn main:app --reload",
    "notes": "Any special setup instructions or context"
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Project name |
| `description` | Yes | What the project does |
| `type` | Yes | One of: web-app, api, mobile, cli, library, full-stack |
| `created` | Yes | ISO 8601 timestamp |
| `tech_stack` | Yes | Array of key technologies |
| `test_command` | Yes | Command to run the test suite |
| `build_command` | No | Command to build the project (if applicable) |
| `run_command` | No | Command to run/serve the project |
| `notes` | No | Free-form notes about setup |

## Feature Object

Each feature represents a single, independently verifiable capability.

```json
{
  "id": 1,
  "category": "core",
  "description": "User can create a new account with email and password",
  "steps": [
    "POST /api/auth/register with valid email and password returns 201",
    "Response includes user ID and auth token",
    "Duplicate email returns 409 Conflict",
    "Invalid email format returns 422 Validation Error"
  ],
  "depends_on": [],
  "priority": "high",
  "passes": false
}
```

| Field | Required | Type | Description |
|-------|----------|------|-------------|
| `id` | Yes | integer | Unique, sequential, never reused |
| `category` | Yes | string | One of the 8 categories below |
| `description` | Yes | string | What this feature does (human-readable, concise) |
| `steps` | Yes | string[] | Concrete verification steps to confirm the feature works |
| `depends_on` | Yes | integer[] | IDs of features that must pass before this one can be attempted |
| `priority` | Yes | string | "high", "medium", or "low" |
| `passes` | Yes | boolean | Whether the feature has been verified as working |

### Writing Good Descriptions

- Start with what the user/system can do: "User can...", "System automatically...", "API returns..."
- Be specific enough to verify: "Login form validates email format" not "Login works"
- One capability per feature: if you write "and" in the description, consider splitting

### Writing Good Steps

Steps are the verification contract. They must be:
- **Concrete**: "Run `pytest tests/test_auth.py` and all tests pass" not "Test authentication"
- **Ordered**: Steps should follow a logical sequence
- **Independent**: Each step should be verifiable on its own
- **Observable**: Describe what you can see/check, not internal implementation

## Categories

Eight categories for organizing features. Use what fits - not every project needs all categories.

| Category | Description | Examples |
|----------|-------------|----------|
| `setup` | Project scaffolding, tooling, CI/CD | Install dependencies, configure linter, set up test framework |
| `core` | Essential business logic | User authentication, data processing, core algorithms |
| `ui` | User interface and interaction | Forms, navigation, responsive layout, modals |
| `api` | API endpoints and contracts | REST endpoints, GraphQL schema, WebSocket handlers |
| `data` | Data storage and retrieval | Database schema, migrations, caching, queries |
| `integration` | Third-party service connections | OAuth, payment gateway, email service, cloud storage |
| `testing` | Test infrastructure and coverage | E2E tests, integration tests, test fixtures, mocks |
| `polish` | UX refinement and edge cases | Error messages, loading states, accessibility, performance |

### Category Distribution Guidelines

The distribution varies by project type. Read `project_type_guides.md` for type-specific recommendations. As a general rule:
- `setup` features come first (ID 1-3 typically)
- `core` features form the bulk
- `testing` and `polish` come last unless TDD approach requires tests first

## Metadata Section

Aggregate tracking information. Updated automatically as features are completed.

```json
{
  "metadata": {
    "total_features": 25,
    "completed_features": 0,
    "last_updated": "2026-03-03T10:00:00Z"
  }
}
```

| Field | Description |
|-------|-------------|
| `total_features` | Total count of features in the list |
| `completed_features` | Count of features where `passes` is `true` |
| `last_updated` | ISO 8601 timestamp of last modification |

## Immutability Rules

The feature list is append-only with one exception:

| Field | Can Change? | When |
|-------|------------|------|
| `passes` | `false` → `true` | After verification |
| `id` | Never | Immutable once assigned |
| `description` | Never | Add a new feature instead |
| `steps` | Never | Add a new feature with updated steps |
| `category` | Never | Immutable once set |
| `depends_on` | Never | Plan dependencies upfront |
| `priority` | Never | Immutable once set |

**Why?** Changing descriptions after some features are verified creates ambiguity about what was actually tested. If requirements evolve, add new features at the end of the list.

New features can be added with the next sequential ID. `metadata.total_features` should be updated accordingly.

## Adaptive Sizing Guide

Match feature granularity to project complexity:

### Small Projects (10-30 features)
- Simple tools, scripts, single-purpose apps
- Each feature = one clear user-facing behavior
- Example: A CLI tool might have 15 features covering argument parsing, core logic, output formatting, error handling

### Medium Projects (30-80 features)
- Multi-page web apps, REST APIs, typical SaaS products
- Break features along user stories or API endpoints
- Group related behaviors but keep each independently testable
- Example: A blog platform might have 50 features across auth, posts, comments, admin, search

### Large Projects (80-200 features)
- Platforms, complex full-stack apps, enterprise systems
- Fine-grained features for critical paths, coarser for auxiliary features
- Prioritize: high-priority features get more granular breakdown
- Example: An e-commerce platform might have 150 features across catalog, cart, checkout, payments, admin, analytics

When in doubt, start with fewer features. It's always safe to add more later.

---

## Example: Small Project

A URL shortener CLI tool:

```json
{
  "project": {
    "name": "url-shortener",
    "description": "CLI tool to shorten URLs using a local SQLite database",
    "type": "cli",
    "created": "2026-03-03T10:00:00Z",
    "tech_stack": ["python", "click", "sqlite3"],
    "test_command": "pytest",
    "build_command": "pip install -e .",
    "notes": "Self-contained, no external API dependencies"
  },
  "features": [
    {
      "id": 1,
      "category": "setup",
      "description": "Project structure with click CLI framework and pytest configured",
      "steps": [
        "Run `pip install -e .` without errors",
        "Run `pytest` and see 0 tests collected (no errors)",
        "Run `urlshort --help` and see usage information"
      ],
      "depends_on": [],
      "priority": "high",
      "passes": false
    },
    {
      "id": 2,
      "category": "data",
      "description": "SQLite database initializes with urls table on first run",
      "steps": [
        "Run `urlshort init` to create the database",
        "Verify ~/.urlshort/urls.db exists",
        "Verify the urls table has columns: id, short_code, original_url, created_at, click_count"
      ],
      "depends_on": [1],
      "priority": "high",
      "passes": false
    },
    {
      "id": 3,
      "category": "core",
      "description": "Shorten a URL and get a short code back",
      "steps": [
        "Run `urlshort shorten https://example.com` and get a 6-character code",
        "Run it again with the same URL and get the same code",
        "Run with a different URL and get a different code"
      ],
      "depends_on": [2],
      "priority": "high",
      "passes": false
    },
    {
      "id": 4,
      "category": "core",
      "description": "Expand a short code back to the original URL",
      "steps": [
        "Shorten a URL, then run `urlshort expand <code>`",
        "Output shows the original URL",
        "Expanding a non-existent code shows an error message"
      ],
      "depends_on": [3],
      "priority": "high",
      "passes": false
    },
    {
      "id": 5,
      "category": "core",
      "description": "List all shortened URLs with click counts",
      "steps": [
        "Create 3 shortened URLs",
        "Run `urlshort list` and see all 3 with their codes and click counts",
        "Output is formatted as a table"
      ],
      "depends_on": [3],
      "priority": "medium",
      "passes": false
    },
    {
      "id": 6,
      "category": "testing",
      "description": "Unit tests cover all core commands",
      "steps": [
        "Run `pytest` with at least 10 tests passing",
        "Tests cover: init, shorten, expand, list, error cases",
        "Tests use a temporary database (not the real one)"
      ],
      "depends_on": [4, 5],
      "priority": "medium",
      "passes": false
    },
    {
      "id": 7,
      "category": "polish",
      "description": "Helpful error messages for all failure modes",
      "steps": [
        "Invalid URL format shows 'Invalid URL: must start with http:// or https://'",
        "Database not initialized shows 'Run urlshort init first'",
        "Unknown command shows available commands"
      ],
      "depends_on": [4],
      "priority": "low",
      "passes": false
    }
  ],
  "metadata": {
    "total_features": 7,
    "completed_features": 0,
    "last_updated": "2026-03-03T10:00:00Z"
  }
}
```

## Example: Medium Project (excerpt)

First 5 features of a blog API:

```json
{
  "project": {
    "name": "blog-api",
    "description": "REST API for a blog platform with auth, posts, and comments",
    "type": "api",
    "created": "2026-03-03T10:00:00Z",
    "tech_stack": ["python", "fastapi", "sqlalchemy", "postgresql"],
    "test_command": "pytest --tb=short",
    "build_command": "docker compose build",
    "run_command": "uvicorn app.main:app --reload",
    "notes": "Requires PostgreSQL running on localhost:5432"
  },
  "features": [
    {
      "id": 1,
      "category": "setup",
      "description": "FastAPI project structure with health check endpoint",
      "steps": [
        "Run `uvicorn app.main:app` without errors",
        "GET /health returns {\"status\": \"ok\"}",
        "Run `pytest` with 0 errors"
      ],
      "depends_on": [],
      "priority": "high",
      "passes": false
    },
    {
      "id": 2,
      "category": "data",
      "description": "SQLAlchemy models and database connection configured",
      "steps": [
        "Database tables created via alembic migration",
        "User, Post, Comment models defined with proper relationships",
        "Connection pooling configured for production use"
      ],
      "depends_on": [1],
      "priority": "high",
      "passes": false
    },
    {
      "id": 3,
      "category": "core",
      "description": "User registration with email and password",
      "steps": [
        "POST /api/auth/register with valid data returns 201 and user object",
        "Password is hashed (not stored in plaintext)",
        "Duplicate email returns 409",
        "Invalid email format returns 422"
      ],
      "depends_on": [2],
      "priority": "high",
      "passes": false
    },
    {
      "id": 4,
      "category": "core",
      "description": "User login returns JWT access token",
      "steps": [
        "POST /api/auth/login with valid credentials returns 200 and token",
        "Token is valid JWT with user_id claim",
        "Wrong password returns 401",
        "Non-existent email returns 401"
      ],
      "depends_on": [3],
      "priority": "high",
      "passes": false
    },
    {
      "id": 5,
      "category": "api",
      "description": "CRUD operations for blog posts",
      "steps": [
        "POST /api/posts (authenticated) creates a post, returns 201",
        "GET /api/posts returns paginated list of posts",
        "GET /api/posts/:id returns single post with author info",
        "PUT /api/posts/:id (author only) updates post",
        "DELETE /api/posts/:id (author only) deletes post"
      ],
      "depends_on": [4],
      "priority": "high",
      "passes": false
    }
  ],
  "metadata": {
    "total_features": 35,
    "completed_features": 0,
    "last_updated": "2026-03-03T10:00:00Z"
  }
}
```
