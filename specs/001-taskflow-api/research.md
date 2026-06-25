# Research: TaskFlow API

## Technology Decisions

### 1. Express + TypeScript (Locked by Constitution)

**Decision**: Express 4.x with TypeScript (strict mode)

**Rationale**: The constitution locks the framework to Express or Fastify.
Express is chosen for its larger ecosystem, simpler middleware model, and
widespread community knowledge. TypeScript strict mode ensures type safety
across the layered architecture.

**Alternatives considered**: Fastify (faster but smaller ecosystem; kept as
future option).

### 2. PostgreSQL + Prisma (Locked by Constitution)

**Decision**: PostgreSQL via Prisma ORM

**Rationale**: Prisma provides type-safe database access that integrates
well with TypeScript. Migrations are versioned via Prisma Migrate. Schema
is defined declaratively in `prisma/schema.prisma`.

**Alternatives considered**: TypeORM (more verbose, less type-safe), Drizzle
(newer, smaller ecosystem), raw SQL (loses type safety).

### 3. JWT Authentication (Locked by Constitution)

**Decision**: Access token (15 min) + Refresh token (7 days) pattern

**Rationale**: Short-lived access tokens reduce the impact of token theft.
Refresh tokens allow seamless session extension without re-authentication.
Tokens are signed with RS256 or HS256. Refresh tokens stored in database
for invalidation capability.

**Implementation pattern**:
- `POST /api/v1/auth/register` — creates user, returns tokens
- `POST /api/v1/auth/login` — validates credentials, returns tokens
- `POST /api/v1/auth/refresh` — accepts valid refresh token, returns new
  access token
- `POST /api/v1/auth/logout` — invalidates refresh token

### 4. Input Validation: Zod

**Decision**: Zod for runtime schema validation

**Rationale**: Zod provides TypeScript-first schema validation with
automatic type inference. Schemas are composable and produce clear error
messages. Controllers validate request bodies against Zod schemas before
passing to services.

**Alternatives considered**: Joi (good but not TypeScript-native), Yup
(less type-safe), class-validator (requires decorators, more boilerplate).

### 5. API Documentation: swagger-jsdoc + swagger-ui-express

**Decision**: Generate OpenAPI 3.x spec from JSDoc annotations on route files

**Rationale**: Keeps the spec co-located with route definitions, reducing
drift. `swagger-ui-express` serves the rendered UI at `/api/v1/docs`.

**Alternatives considered**: Hand-written OpenAPI YAML (drift risk),
tsoa (opinionated, generates routes + spec automatically but less control).

### 6. Structured Logging: Pino

**Decision**: Pino for JSON logging

**Rationale**: Pino is the fastest Node.js logger, outputs structured JSON
by default, and has a rich ecosystem of transport options. Constitution
requires structured JSON logs across all routes.

**Implementation**: A request-logger middleware captures method, URL,
status, duration, and user ID (if authenticated). No sensitive data logged.

### 7. Testing: Jest + Supertest

**Decision**: Jest for unit tests, Supertest for integration tests

**Rationale**: Jest is the standard TypeScript test runner with built-in
coverage reporting. Supertest allows HTTP assertions against the Express
app without binding to a port.

**Test strategy**:
- **Unit tests**: Pure logic in services, validation schemas, helpers
- **Integration tests**: Full request-response cycle through all layers,
  using a test database (or in-memory SQLite via Prisma)
- **Contract tests**: Verify response shapes match OpenAPI spec

### 8. Docker Setup

**Decision**: Docker Compose with API + PostgreSQL services

**Rationale**: Docker Compose provides reproducible local development
environment. API service uses a dev Dockerfile with ts-node-dev for hot
reload. PostgreSQL service uses the official image.

**Production**: Multi-stage Dockerfile: build with full image, run with
distroless Node.js image.

### 9. Project Architecture: Layered Clean Architecture

**Decision**: Four-layer architecture (routes → controllers → services →
repositories)

**Rationale**: Strict dependency direction ensures testability and
maintainability. Controllers depend on services (not repositories).
Services depend on repositories (not controllers). Dependency injection
is manual (constructor parameters) — no DI framework to keep it simple
per the Simplicity principle.

### 10. Environment Configuration

**Decision**: Single `env.ts` module reading from `process.env` with Zod
validation at startup

**Rationale**: Validates all required env vars at app boot, fails fast
with clear messages. Dotenv loads `.env` file in development.

### 11. Error Handling

**Decision**: Centralized error handler middleware with typed error classes

**Rationale**: Services throw typed errors (e.g., `NotFoundError`,
`UnauthorizedError`, `ValidationError`). The error handler middleware
catches them and returns consistent JSON error responses. This prevents
sensitive data leakage and ensures uniform error format.

**Format**: `{ error: { code: string, message: string, details?: any } }`
