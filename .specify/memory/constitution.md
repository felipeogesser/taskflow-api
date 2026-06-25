<!--
  Sync Impact Report

  Version change: N/A (initial) → 1.0.0
  Bump rationale: Initial constitution creation.

  Modified principles: None (all new)
  Added sections:
    - I. Simplicity
    - II. API-First
    - III. Security by Default
    - IV. Mandatory Testing
    - V. Observability
    - VI. Compatibility & Versioning
    - VII. Technology Stack
    - Additional Constraints (Technology Stack & Standards)
    - Development Workflow & Quality Gates

  Removed sections: None
  Templates requiring updates:
    - .specify/templates/plan-template.md ✅ (Constitution Check gate is already generic)
    - .specify/templates/spec-template.md ✅ (No principle-specific references)
    - .specify/templates/tasks-template.md ✅ (Generic categories cover all principle areas)
    - .specify/templates/checklist-template.md ✅ (Generic template, no changes needed)

  Deferred TODOs: None
-->

# TaskFlow API Constitution

## Core Principles

### I. Simplicity

Prioritize direct solutions over abstract indirection. Every feature MUST
justify its own complexity — YAGNI (You Ain't Gonna Need It) is strictly
enforced. Favor straightforward implementations (procedural or flat modules)
over patterns (repositories, factories, DTOs) unless the complexity is
demonstrably required by changing requirements or performance constraints.

### II. API-First

The OpenAPI specification is the single source of truth for the API contract.
All endpoint changes MUST be reflected in the contract BEFORE implementation
begins. The contract is versioned under `docs/` following semantic versioning
(v1, v2...). No breaking changes are permitted within the same API version;
breaking changes require a new major version. Consumer-driven contract tests
MUST validate every endpoint against the spec.

### III. Security by Default

Authentication via JWT is MANDATORY for every endpoint unless explicitly and
documentedly opted out. Sensitive data (passwords, tokens, PII) MUST NEVER
appear in logs, error messages, metrics labels, or response bodies. Input
validation MUST be applied at every API boundary. Follow OWASP API Security
Top 10 guidelines. All secrets MUST be managed via environment variables or a
secrets manager — never committed to the repository.

### IV. Mandatory Testing

Business rule code MUST maintain minimum 80% coverage. Critical endpoints
(authentication, task CRUD, team management) MUST follow TDD: tests written
first → tests reviewed → tests fail → then implement. Red-Green-Refactor cycle
is strictly enforced. Test categories:
- **Unit tests**: Business logic and validation rules.
- **Integration tests**: Endpoint behavior, database interactions, auth flows.
- **Contract tests**: API request/response conformance to OpenAPI spec.
Tests MUST run in CI and block merges on failure.

### V. Observability

Every request MUST produce structured JSON logs containing: request-id,
endpoint, HTTP method, duration (ms), status code, and authenticated user ID
(if applicable). Metrics MUST be collected for all routes: request count,
latency (p50/p95/p99), error rate by status, and throughput. No sensitive data
in log fields or metric labels. Health and readiness endpoints are required.

### VI. Compatibility & Versioning

The API MUST follow semantic versioning (MAJOR.MINOR.PATCH) based on contract
changes:
- **MAJOR**: Breaking endpoint or schema change.
- **MINOR**: Backward-compatible addition (new endpoint, optional field).
- **PATCH**: Bug fix, clarification, non-breaking refinement.
API versions are URL-prefixed (`/api/v1/`, `/api/v2/`). Deprecated versions
MUST be supported for at least one minor release cycle before removal.

### VII. Technology Stack

The stack is locked to:
- **Runtime**: Node.js (latest LTS)
- **Language**: TypeScript (strict mode enabled)
- **Framework**: Express or Fastify
- **Database**: PostgreSQL
- **ORM**: Prisma
- **API Spec**: OpenAPI 3.x
- **Auth**: JWT (jsonwebtoken)
- **Testing**: Jest or Vitest + Supertest
- **Logging**: Pino or Winston (structured JSON)
- **Linting**: ESLint + Prettier
Deviation from any stack component requires documented justification and team
approval.

## Additional Constraints

### Technology Stack & Standards

- **Database migrations**: Managed exclusively via Prisma Migrate.
  Rollbacks MUST be tested before deployment.
- **Environment configuration**: Managed via `.env` files + validation schema.
  Never commit `.env` files — use `.env.example` as template.
- **Containerization**: Docker Compose for local development; production
  images MUST be distroless or alpine-based.
- **CI/CD**: All pushes MUST pass lint, type check, and test suite before
  merge. Coverage reports MUST be generated and verified.

### Security Requirements

- Rate limiting on auth endpoints (configurable via env).
- CORS MUST be restricted to known origins in production.
- All passwords hashed with bcrypt (cost factor >= 10).
- JWT tokens MUST have short expiry (access: 15 min, refresh: 7 days max).
- Helmet middleware MUST be applied to set secure HTTP headers.

## Development Workflow & Quality Gates

### Branch Strategy

- `main` — production-ready, protected, requires passing CI + review.
- `feat/<name>` — new features, branched from `main`, merged via PR.
- `fix/<name>` — bug fixes.
- `chore/<name>` — maintenance, dependencies, tooling.

### Pull Request Gates

1. **Lint & Format** — ESLint + Prettier pass.
2. **Type Check** — TypeScript `tsc --noEmit` passes.
3. **Test Suite** — All tests pass (unit + integration + contract).
4. **Coverage Gate** — Business rule coverage >= 80%.
5. **API Contract Check** — Contract tests against OpenAPI spec pass.
6. **Review** — At least one approval; author may not approve own PR.
7. **No Secrets** — Automated scan confirms no credentials committed.

### Definition of Done

- Feature matches OpenAPI contract.
- All acceptance scenarios pass.
- Tests written and passing (TDD for critical endpoints).
- Structured logging added for new endpoints.
- Documentation updated if contract changed.
- No sensitive data exposed in logs or responses.

## Governance

This Constitution supersedes all other project practices and conventions.
Amendments require:
1. A documented proposal explaining the rationale and impact.
2. Team review and consensus (or majority approval).
3. A migration plan for any breaking changes.
4. Update of this document with new version and date.

Compliance is verified during every PR review. Any deviation from principles
MUST be explicitly justified in the PR description.

Complexity introduced without justification is grounds for PR rejection.

**Version**: 1.0.0 | **Ratified**: 2026-06-25 | **Last Amended**: 2026-06-25
