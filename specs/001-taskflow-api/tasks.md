---

description: "Task list for the TaskFlow API feature implementation"

---

# Tasks: TaskFlow API

**Input**: Design documents from `specs/001-taskflow-api/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Integration tests are included for each user story endpoint (TDD for critical endpoints per Constitution principle IV). Unit tests for services and repositories are included in the Polish phase.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Single project**: `src/`, `tests/`, `prisma/` at repository root
- **Web service**: Layered architecture: `routes/` → `controllers/` → `services/` → `repositories/`

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project directory structure: src/routes, src/controllers, src/services, src/repositories, src/middlewares, src/validators, src/types, src/config, src/utils, prisma, tests/integration, tests/unit, docs
- [ ] T002 [P] Initialize package.json with project metadata and dependencies (express, prisma, @prisma/client, jsonwebtoken, bcrypt, pino, zod, swagger-jsdoc, swagger-ui-express, helmet, cors, dotenv, uuid)
- [ ] T003 [P] Install devDependencies (typescript, ts-node-dev, jest, ts-jest, @types/jest, supertest, @types/supertest, @types/express, @types/jsonwebtoken, @types/bcrypt, @types/uuid, eslint, prettier, @typescript-eslint/parser, @typescript-eslint/eslint-plugin)
- [ ] T004 [P] Configure TypeScript in tsconfig.json (strict mode, ES2022 target, NodeNext module, outDir dist)
- [ ] T005 [P] Configure ESLint in .eslintrc.json with TypeScript rules
- [ ] T006 [P] Configure Prettier in .prettierrc
- [ ] T007 [P] Create Jest config in jest.config.ts (ts-jest preset, roots: tests/, src/)
- [ ] T008 [P] Create .env.example with all required environment variables (DATABASE_URL, JWT_SECRET, JWT_REFRESH_SECRET, PORT, CORS_ORIGIN)
- [ ] T009 [P] Create Dockerfile (multi-stage: build with full image, run with distroless)
- [ ] T010 [P] Create Dockerfile.dev (ts-node-dev for hot reload)
- [ ] T011 Create docker-compose.yml with api and postgres services (PostgreSQL port 5432, API port 3000)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T012 Create Prisma schema in prisma/schema.prisma with all entities (User, Task, Project, ProjectMember, Comment, Notification) and enums (TaskStatus, Priority, ProjectRole, NotificationType)
- [ ] T013 Run initial Prisma migration: `npx prisma migrate dev --name init`
- [ ] T014 [P] Create environment config in src/config/env.ts with Zod schema validation for all env vars
- [ ] T015 [P] Create Prisma client singleton in src/config/database.ts
- [ ] T016 [P] Create error utility classes in src/utils/errors.ts (AppError, NotFoundError, UnauthorizedError, ForbiddenError, ValidationError, ConflictError)
- [ ] T017 [P] Create type definitions in src/types/express.d.ts (extend Express Request with user property)
- [ ] T018 [P] Create auth types in src/types/auth.types.ts (JwtPayload, Tokens)
- [ ] T019 [P] Create pagination types in src/types/pagination.types.ts (PaginationParams, PaginatedResult)
- [ ] T020 [P] Create JWT utility in src/utils/jwt.ts (generateAccessToken, generateRefreshToken, verifyAccessToken, verifyRefreshToken)
- [ ] T021 [P] Create password utility in src/utils/password.ts (hashPassword, comparePassword with bcrypt)
- [ ] T022 [P] Create error handler middleware in src/middlewares/error-handler.middleware.ts (catches AppError subclasses, returns consistent JSON error format)
- [ ] T023 [P] Create request logger middleware in src/middlewares/request-logger.middleware.ts (Pino structured JSON logs with request-id, method, URL, status, duration)
- [ ] T024 [P] Create auth middleware in src/middlewares/auth.middleware.ts (verify Bearer token, attach user to request)
- [ ] T025 [P] Create pagination middleware in src/middlewares/pagination.middleware.ts (parse page/limit query params with defaults)
- [ ] T026 Create Express app setup in src/app.ts (register middleware: helmet, cors, json parser, request-logger, error-handler; register route placeholders)
- [ ] T027 Create server entry point in src/server.ts (import app, listen on PORT, graceful shutdown)

**Checkpoint**: Foundation ready — user story implementation can now begin in parallel

---

## Phase 3: User Story 1 — User Registration & Authentication (Priority: P1) 🎯 MVP

**Goal**: Users can register with email/password and authenticate via JWT tokens

**Independent Test**: A new user registers, receives tokens, and accesses a protected endpoint; invalid credentials are rejected

### Implementation for User Story 1

- [ ] T028 [P] [US1] Create auth validation schemas in src/validators/auth.schema.ts (register: email, password, display_name; login: email, password)
- [ ] T029 [P] [US1] Create user repository in src/repositories/user.repository.ts (findByEmail, create, findById)
- [ ] T030 [US1] Create auth service in src/services/auth.service.ts (register validates email unique, hashes password, creates user, returns tokens; login validates credentials, returns tokens; refresh validates refresh token, returns new access token)
- [ ] T031 [US1] Create auth controller in src/controllers/auth.controller.ts (register, login, refresh, logout handlers with Zod validation)
- [ ] T032 [US1] Create auth routes in src/routes/auth.routes.ts (POST /auth/register, POST /auth/login, POST /auth/refresh, POST /auth/logout — no auth middleware on register/login)
- [ ] T033 [US1] Wire auth routes into app.ts

**Checkpoint**: US1 complete — users can register and authenticate. MVP deliverable.

---

## Phase 4: User Story 2 — Task Management (Priority: P1)

**Goal**: Authenticated users can create, view, edit, delete, filter, and paginate tasks

**Independent Test**: An authenticated user creates a task, lists tasks with filters, edits the title, deletes it — confirming it no longer appears

### Implementation for User Story 2

- [ ] T034 [P] [US2] Create task validation schemas in src/validators/task.schema.ts (create: title, optional description/status/priority/due_date/assignee_id/project_id; update: partial)
- [ ] T035 [P] [US2] Create task repository in src/repositories/task.repository.ts (create, findById, findAll with pagination/filters, update, delete)
- [ ] T036 [US2] Create task service in src/services/task.service.ts (CRUD with authorization — only task creator or project members can update/delete; creates notification on assignee change)
- [ ] T037 [US2] Create task controller in src/controllers/task.controller.ts (list, getById, create, update, delete handlers with Zod validation and pagination)
- [ ] T038 [US2] Create task routes in src/routes/task.routes.ts (GET /tasks, POST /tasks, GET /tasks/:id, PATCH /tasks/:id, DELETE /tasks/:id)
- [ ] T039 [US2] Wire task routes into app.ts

**Checkpoint**: US2 complete — full task CRUD with filters and pagination

---

## Phase 5: User Story 3 — Project Organization & Collaboration (Priority: P2)

**Goal**: Users can create projects, invite members, and collaborate on tasks within projects

**Independent Test**: User A creates a project, invites User B; User B sees and creates tasks in the project; User C (uninvited) cannot access it

### Implementation for User Story 3

- [ ] T040 [P] [US3] Create project validation schemas in src/validators/project.schema.ts (create: name, optional description; update: partial)
- [ ] T041 [P] [US3] Create project repository in src/repositories/project.repository.ts (create, findById, findByUser, update, delete, addMember, removeMember, isMember)
- [ ] T042 [US3] Create project service in src/services/project.service.ts (CRUD with ownership enforcement; invite validates user exists by email; delete checks no orphan tasks or transfers ownership)
- [ ] T043 [US3] Create project controller in src/controllers/project.controller.ts (list, getById with tasks, create, update, delete, inviteMember, removeMember handlers)
- [ ] T044 [US3] Create project routes in src/routes/project.routes.ts (GET /projects, POST /projects, GET /projects/:id, PATCH /projects/:id, DELETE /projects/:id, POST /projects/:id/members, DELETE /projects/:id/members/:userId)
- [ ] T045 [US3] Wire project routes into app.ts

**Checkpoint**: US3 complete — team collaboration with project-based authorization

---

## Phase 6: User Story 4 — Comments on Tasks (Priority: P2)

**Goal**: Project members can add and view comments on tasks

**Independent Test**: A project member adds a comment to a task; the comment appears when task details are viewed with author and timestamp

### Implementation for User Story 4

- [ ] T046 [P] [US4] Create comment validation schemas in src/validators/comment.schema.ts (content: min 1 char, max 5000 chars)
- [ ] T047 [P] [US4] Create comment repository in src/repositories/comment.repository.ts (create, findByTask with chronological order)
- [ ] T048 [US4] Create comment service in src/services/comment.service.ts (create validates task access via project membership; list by task)
- [ ] T049 [US4] Create comment controller in src/controllers/comment.controller.ts (listByTask, create handlers)
- [ ] T050 [US4] Create comment routes in src/routes/comment.routes.ts (GET /tasks/:taskId/comments, POST /tasks/:taskId/comments)
- [ ] T051 [US4] Wire comment routes into app.ts

**Checkpoint**: US4 complete — task discussion via comments

---

## Phase 7: User Story 5 — Internal Notifications (Priority: P3)

**Goal**: Users receive internal notifications when a task is assigned to them or a task's due date is approaching

**Independent Test**: User A assigns a task to User B; User B's notification list shows an unread notification; User B marks it as read

### Implementation for User Story 5

- [ ] T052 [P] [US5] Create notification repository in src/repositories/notification.repository.ts (create, findByUser with pagination, markAsRead, markAllAsRead, countUnread)
- [ ] T053 [US5] Create notification service in src/services/notification.service.ts (createOnAssignment, createDueDateNotification, list, markAsRead; scheduled check for approaching due dates)
- [ ] T054 [US5] Create notification controller in src/controllers/notification.controller.ts (list with unread_only filter and pagination, markAsRead, markAllAsRead handlers)
- [ ] T055 [US5] Create notification routes in src/routes/notification.routes.ts (GET /notifications, POST /notifications/:id/read)
- [ ] T056 [US5] Wire notification routes into app.ts
- [ ] T057 [US5] Integrate notification creation into task service (call notification service when assignee changes)
- [ ] T058 Implement periodic due-date check in src/services/notification.service.ts (cron-like interval that scans tasks with due dates within threshold and creates notifications)

**Checkpoint**: US5 complete — users receive and manage internal notifications

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T059 [P] Create health endpoint in src/routes/health.routes.ts (GET /health returns status + timestamp)
- [ ] T060 [P] Configure swagger-jsdoc in src/config/swagger.ts (generate OpenAPI spec from JSDoc route annotations)
- [ ] T061 [P] Configure swagger-ui-express in src/app.ts (serve docs at /api/v1/docs)
- [ ] T062 [P] Add JSDoc OpenAPI annotations to all route files (auth, task, project, comment, notification routes)
- [ ] T063 [P] Generate Prisma client: `npx prisma generate`
- [ ] T064 [P] Run initial type check: `npx tsc --noEmit` and fix any type errors
- [ ] T065 [P] Write auth integration tests in tests/integration/auth.test.ts (register, login, refresh, logout, unauthenticated rejection)
- [ ] T066 [P] Write task integration tests in tests/integration/task.test.ts (CRUD, pagination, filtering, authorization)
- [ ] T067 [P] Write project integration tests in tests/integration/project.test.ts (CRUD, member invite/remove, task access control)
- [ ] T068 [P] Write comment integration tests in tests/integration/comment.test.ts (add, list, task access control)
- [ ] T069 [P] Write notification integration tests in tests/integration/notification.test.ts (list, mark as read, unread filter)
- [ ] T070 [P] Write unit tests for auth service in tests/unit/services/auth.service.test.ts
- [ ] T071 [P] Write unit tests for task service in tests/unit/services/task.service.test.ts
- [ ] T072 [P] Write unit tests for project service in tests/unit/services/project.service.test.ts
- [ ] T073 [P] Write unit tests for comment service in tests/unit/services/comment.service.test.ts
- [ ] T074 [P] Write unit tests for notification service in tests/unit/services/notification.service.test.ts
- [ ] T075 Run full test suite and verify all tests pass
- [ ] T076 Run coverage report and verify business rule coverage >= 80%
- [ ] T077 Run quickstart.md validation scenarios to confirm end-to-end functionality

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion — BLOCKS all user stories
- **User Stories (Phase 3-7)**: All depend on Foundational phase completion
  - US1 (Phase 3) and US2 (Phase 4) can start immediately after Foundation
  - US3 (Phase 5) has no dependency on US1/US2 but requires US1 auth concepts
  - US4 (Phase 6) depends on US2 tasks existing and US3 projects for access control
  - US5 (Phase 7) depends on US2 (task assignment) and US3 (project members)
- **Polish (Phase 8)**: Depends on all user stories being complete

### User Story Dependencies

- **US1 (P1)**: Can start after Foundation — No dependencies on other stories
- **US2 (P1)**: Can start after Foundation — No dependencies on other stories
- **US3 (P2)**: Can start after Foundation — No dependencies on other stories
- **US4 (P2)**: Depends on US2 (tasks exist) and US3 (project access control)
- **US5 (P3)**: Depends on US2 (task assignment triggers notifications) and US3 (project membership context)

### Within Each User Story

- Models/repositories before services
- Services before controllers
- Controllers before routes
- Routes before wiring into app.ts

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- US1, US2, US3 can all start in parallel once Foundation is complete
- US4 must wait for US2 + US3
- US5 must wait for US2 + US3
- Within each story: all [P] tasks (validators + repositories) can run in parallel
- All Polish tasks marked [P] can run in parallel

---

## Parallel Example: User Story 1

```bash
# Launch all [P] tasks for US1 together:
Task: "Create auth validation schemas in src/validators/auth.schema.ts"
Task: "Create user repository in src/repositories/user.repository.ts"

# Then sequential:
Task: "Create auth service in src/services/auth.service.ts"
Task: "Create auth controller in src/controllers/auth.controller.ts"
Task: "Create auth routes in src/routes/auth.routes.ts"
Task: "Wire auth routes into app.ts"
```

## Parallel Example: User Story 2

```bash
# Launch all [P] tasks for US2 together:
Task: "Create task validation schemas in src/validators/task.schema.ts"
Task: "Create task repository in src/repositories/task.repository.ts"

# Then sequential:
Task: "Create task service in src/services/task.service.ts"
Task: "Create task controller in src/controllers/task.controller.ts"
Task: "Create task routes in src/routes/task.routes.ts"
Task: "Wire task routes into app.ts"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL — blocks all stories)
3. Complete Phase 3: User Story 1 (Auth)
4. **STOP and VALIDATE**: Test US1 independently
5. Deploy Demo: Register and login working

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 (Auth) → Test independently → Deploy
3. Add User Story 2 (Task CRUD) → Test independently → Deploy (Core MVP!)
4. Add User Story 3 (Projects) → Test independently → Deploy
5. Add User Story 4 (Comments) → Test independently → Deploy
6. Add User Story 5 (Notifications) → Test independently → Deploy
7. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:
1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1 (Auth)
   - Developer B: User Story 2 (Task CRUD)
   - Developer C: User Story 3 (Projects)
3. Developer A then picks up User Story 4 (Comments) — depends on US2+US3
4. Developer B or C picks up User Story 5 (Notifications) — depends on US2+US3
5. Polish phase done together

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Verify tests fail before implementing
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Ensure no sensitive data in logs (passwords, tokens) per Constitution principle III
- Use structured Pino logging in all services and controllers per Constitution principle V
