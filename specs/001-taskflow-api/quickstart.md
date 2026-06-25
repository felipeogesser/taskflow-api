# Quickstart: TaskFlow API

End-to-end validation guide for the TaskFlow API feature.

## Prerequisites

- Docker and Docker Compose installed
- Node.js v22+ (for local dev without Docker)
- Ports 3000 (API) and 5432 (PostgreSQL) available

## Setup

### Option A: Docker Compose (recommended)

```bash
# Start API + PostgreSQL
docker compose up --build

# API available at http://localhost:3000
```

### Option B: Local development

```bash
# Install dependencies
npm install

# Copy environment file
cp .env.example .env
# Edit .env with your PostgreSQL connection string

# Run database migrations
npx prisma migrate dev

# Start with hot reload
npm run dev
```

## Validation Scenarios

### Scenario 1: Health Check

```bash
curl http://localhost:3000/api/v1/health
```

**Expected**: `{"status":"ok","timestamp":"..."}`

---

### Scenario 2: User Registration & Authentication

```bash
# Register a new user
curl -X POST http://localhost:3000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"Str0ng!Pass","display_name":"Alice"}'

# Expected: 201 with user data + tokens

# Login
curl -X POST http://localhost:3000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@example.com","password":"Str0ng!Pass"}'

# Expected: 200 with user data + tokens (save access_token for next steps)

# Register a second user
curl -X POST http://localhost:3000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"bob@example.com","password":"Str0ng!Pass","display_name":"Bob"}'
```

**Set**: `TOKEN=<access_token_from_login>`

---

### Scenario 3: Task CRUD

```bash
# Create a task
curl -X POST http://localhost:3000/api/v1/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"title":"Buy groceries","description":"Milk, eggs, bread","priority":"high","due_date":"2026-07-01T12:00:00Z"}'

# Expected: 201 with task object (save task_id)

# List tasks
curl http://localhost:3000/api/v1/tasks \
  -H "Authorization: Bearer $TOKEN"

# Expected: 200 with paginated task list

# Filter tasks by status
curl "http://localhost:3000/api/v1/tasks?status=pending" \
  -H "Authorization: Bearer $TOKEN"

# Expected: 200 with filtered list

# Get task details
curl http://localhost:3000/api/v1/tasks/<task_id> \
  -H "Authorization: Bearer $TOKEN"

# Update task status
curl -X PATCH http://localhost:3000/api/v1/tasks/<task_id> \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"status":"in_progress"}'

# Delete task
curl -X DELETE http://localhost:3000/api/v1/tasks/<task_id> \
  -H "Authorization: Bearer $TOKEN"

# Expected: 204 No Content
```

---

### Scenario 4: Project & Collaboration

```bash
# Create a project
curl -X POST http://localhost:3000/api/v1/projects \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"name":"Team Project","description":"Our team project"}'

# Expected: 201 with project (save project_id)

# Invite Bob (using Bob's email)
curl -X POST http://localhost:3000/api/v1/projects/<project_id>/members \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"email":"bob@example.com"}'

# Expected: 200

# Bob creates a task in the project (using Bob's token)
curl -X POST http://localhost:3000/api/v1/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $BOB_TOKEN" \
  -d '{"title":"Project task","project_id":"<project_id>"}'

# Expected: 201

# List project tasks
curl "http://localhost:3000/api/v1/tasks?project_id=<project_id>" \
  -H "Authorization: Bearer $TOKEN"

# Expected: 200 with project tasks
```

---

### Scenario 5: Comments

```bash
# Add a comment to a task (already have a task from scenario 3)
curl -X POST http://localhost:3000/api/v1/tasks/<task_id>/comments \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"content":"Started working on this"}'

# Expected: 201 with comment

# List comments
curl http://localhost:3000/api/v1/tasks/<task_id>/comments \
  -H "Authorization: Bearer $TOKEN"

# Expected: 200 with comments array
```

---

### Scenario 6: Notifications

```bash
# Assign task to Bob → Bob's notification list shows it
curl -X PATCH http://localhost:3000/api/v1/tasks/<task_id> \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"assignee_id":"<bob_user_id>"}'

# Check Bob's notifications (using Bob's token)
curl http://localhost:3000/api/v1/notifications \
  -H "Authorization: Bearer $BOB_TOKEN"

# Expected: 200 with at least one unread notification

# Mark notification as read
curl -X POST http://localhost:3000/api/v1/notifications/<notification_id>/read \
  -H "Authorization: Bearer $BOB_TOKEN"

# Expected: 200
```

## Verification Checklist

- [ ] Health endpoint returns 200
- [ ] User can register
- [ ] User can log in and receive tokens
- [ ] Unauthenticated requests to protected endpoints return 401
- [ ] User can create, read, update, and delete tasks
- [ ] Task listing supports pagination (page, limit, total, total_pages)
- [ ] Task listing supports filtering (status, priority, assignee, project)
- [ ] User can create a project
- [ ] User can invite another user to a project
- [ ] Project members can create tasks in the project
- [ ] Non-members cannot access project tasks
- [ ] User can add and view comments on a task
- [ ] Notification is created when a task is assigned
- [ ] User can list and mark notifications as read

## Cleanup

```bash
# Stop Docker containers
docker compose down -v

# Or if running locally, drop the database
npx prisma migrate reset --force
```

## References

- [Data Model](data-model.md)
- [API Contract](contracts/openapi.yml)
- [Feature Spec](spec.md)
