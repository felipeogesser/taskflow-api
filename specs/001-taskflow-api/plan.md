# Implementation Plan: TaskFlow API

**Branch**: `` | **Date**: 2026-06-25 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/001-taskflow-api/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

Build a REST API for task management with user registration and
authentication (JWT), individual and team task CRUD, project organization,
collaborative comments, and internal notifications. The API follows a
layered architecture (routes → controllers → services → repositories) with
Express + TypeScript + Prisma + PostgreSQL, containerized via Docker.

## Technical Context

**Language/Version**: TypeScript with Node.js (LTS — currently v22)

**Primary Dependencies**: Express, Prisma ORM, PostgreSQL driver (`pg`),
jsonwebtoken + `@types/jsonwebtoken`, bcrypt + `@types/bcrypt`, Pino (structured
logging), Zod (input validation), Swagger/OpenAPI (`swagger-jsdoc` +
`swagger-ui-express`), Jest + Supertest (testing), ESLint + Prettier (lint/format)

**Storage**: PostgreSQL via Prisma ORM, migrations managed by Prisma Migrate

**Testing**: Jest + Supertest for integration tests; Jest for unit tests;
contract tests via OpenAPI spec validation

**Target Platform**: Linux (Docker container), Docker Compose for local dev

**Project Type**: web-service (REST API)

**Performance Goals**: <200ms p95 response time for authenticated endpoints
under normal load; <500ms for list/filter endpoints with up to 100 items

**Constraints**: JWT auth required on all endpoints (except register/login);
layered architecture with strict layer boundaries (routes → controllers →
services → repositories); no business logic in controllers or repositories

**Scale/Scope**: Small-to-medium team (up to 50 users per project), up to
10k registered users, up to 100k tasks

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Justification |
|-----------|--------|---------------|
| I. Simplicity | ✅ PASS | Layered architecture is justified for separation of concerns in a multi-resource REST API. Each layer has a single responsibility. No over-engineering (no abstract factories, no DTO classes beyond TypeScript interfaces). |
| II. API-First | ✅ PASS | OpenAPI spec is generated via `swagger-jsdoc` from JSDoc annotations on route files, serving as living documentation. Contract tests will validate endpoints against the spec. |
| III. Security by Default | ✅ PASS | JWT (access + refresh tokens) on all endpoints except `/auth/register` and `/auth/login`. bcrypt for passwords. Helmet middleware. Zod validation at controller boundary. |
| IV. Mandatory Testing | ✅ PASS | Jest + Supertest for integration tests targeting all endpoints. Jest for unit tests on services and repositories. Minimum 80% coverage on business rules. |
| V. Observability | ✅ PASS | Pino for structured JSON logging with request-id, duration, and status per endpoint. Health endpoint at `/api/v1/health`. |
| VI. Compatibility & Versioning | ✅ PASS | URL-prefixed versioning `/api/v1/`. OpenAPI spec versioned independently. No breaking changes without new version. |
| VII. Technology Stack | ✅ PASS | Express + TypeScript + PostgreSQL + Prisma + Jest + Pino — all within constitution-defined stack. |

**No violations.** All principles are met by the proposed architecture.

## Project Structure

### Documentation (this feature)

```text
specs/001-taskflow-api/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0 research
├── data-model.md        # Phase 1 data model
├── quickstart.md        # Phase 1 validation guide
├── contracts/           # Phase 1 API contracts
│   └── openapi.yml
├── checklists/
│   └── requirements.md  # Spec quality checklist
└── tasks.md             # Phase 2 tasks
```

### Source Code (repository root)

```text
src/
├── routes/              # Express route definitions
│   ├── auth.routes.ts
│   ├── task.routes.ts
│   ├── project.routes.ts
│   ├── comment.routes.ts
│   └── notification.routes.ts
├── controllers/         # Request handling, validation, response formatting
│   ├── auth.controller.ts
│   ├── task.controller.ts
│   ├── project.controller.ts
│   ├── comment.controller.ts
│   └── notification.controller.ts
├── services/            # Business logic (no HTTP awareness)
│   ├── auth.service.ts
│   ├── task.service.ts
│   ├── project.service.ts
│   ├── comment.service.ts
│   └── notification.service.ts
├── repositories/        # Data access layer wrapping Prisma calls
│   ├── user.repository.ts
│   ├── task.repository.ts
│   ├── project.repository.ts
│   ├── comment.repository.ts
│   └── notification.repository.ts
├── middlewares/         # Express middleware
│   ├── auth.middleware.ts
│   ├── error-handler.middleware.ts
│   ├── request-logger.middleware.ts
│   └── pagination.middleware.ts
├── validators/          # Zod schemas for input validation
│   ├── auth.schema.ts
│   ├── task.schema.ts
│   ├── project.schema.ts
│   └── comment.schema.ts
├── types/               # TypeScript types and interfaces
│   ├── express.d.ts
│   ├── auth.types.ts
│   └── pagination.types.ts
├── config/              # Environment configuration
│   ├── env.ts
│   └── database.ts
├── utils/               # Helpers
│   ├── jwt.ts
│   ├── password.ts
│   └── errors.ts
├── app.ts               # Express app setup
└── server.ts            # Entry point

tests/
├── integration/
│   ├── auth.test.ts
│   ├── task.test.ts
│   ├── project.test.ts
│   ├── comment.test.ts
│   └── notification.test.ts
└── unit/
    ├── services/
    │   ├── auth.service.test.ts
    │   ├── task.service.test.ts
    │   ├── project.service.test.ts
    │   ├── comment.service.test.ts
    │   └── notification.service.test.ts
    └── repositories/
        └── *.test.ts

prisma/
└── schema.prisma         # Database schema

docs/
└── openapi.yaml          # Generated OpenAPI spec

docker-compose.yml        # API + PostgreSQL
Dockerfile                # Production image
Dockerfile.dev            # Dev image (hot reload)
.env.example
```

**Structure Decision**: Single project (backend-only REST API).
Clean Architecture simplified: strict one-way dependency flow
routes → controllers → services → repositories. Controllers handle
HTTP request parsing and response formatting. Services contain all
business logic. Repositories wrap Prisma client calls.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No violations found. Complexity tracking section is intentionally left empty.
