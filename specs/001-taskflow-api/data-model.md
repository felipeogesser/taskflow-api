# Data Model: TaskFlow API

## Entity-Relationship Diagram (Text)

```
User ──< Task ──< Comment
  │        │
  │        └──< ProjectMember >── Project
  │
  └──< Notification
```

## Entities

### User

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID (PK) | Auto-generated |
| email | String | Unique, not null |
| password_hash | String | Not null |
| display_name | String | Not null |
| created_at | DateTime | Auto, not null |
| updated_at | DateTime | Auto, not null |

**Relationships**:
- Has many `Task` (as assignee)
- Has many `ProjectMember` (memberships)
- Has many `Notification`

### Project

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID (PK) | Auto-generated |
| name | String | Not null |
| description | String? | Optional |
| owner_id | UUID (FK → User) | Not null |
| created_at | DateTime | Auto, not null |
| updated_at | DateTime | Auto, not null |

**Relationships**:
- Belongs to `User` (owner)
- Has many `Task`
- Has many `ProjectMember`

### ProjectMember

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID (PK) | Auto-generated |
| project_id | UUID (FK → Project) | Not null, unique with user |
| user_id | UUID (FK → User) | Not null, unique with project |
| role | Enum (owner, member) | Default: member |
| joined_at | DateTime | Auto, not null |

**Uniques**: `[project_id, user_id]`

### Task

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID (PK) | Auto-generated |
| title | String | Not null |
| description | Text? | Optional |
| status | Enum (pending, in_progress, completed) | Default: pending |
| priority | Enum (low, medium, high) | Default: medium |
| due_date | DateTime? | Optional |
| assignee_id | UUID (FK → User)? | Optional |
| project_id | UUID (FK → Project)? | Optional (personal task) |
| created_by_id | UUID (FK → User) | Not null |
| created_at | DateTime | Auto, not null |
| updated_at | DateTime | Auto, not null |

**Indexes**: `status`, `priority`, `assignee_id`, `project_id`

**Relationships**:
- Belongs to `User` (assignee, optional)
- Belongs to `Project` (optional)
- Belongs to `User` (creator)
- Has many `Comment`

### Comment

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID (PK) | Auto-generated |
| content | Text | Not null |
| author_id | UUID (FK → User) | Not null |
| task_id | UUID (FK → Task) | Not null |
| created_at | DateTime | Auto, not null |
| updated_at | DateTime | Auto, not null |

**Relationships**:
- Belongs to `User` (author)
- Belongs to `Task`

### Notification

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID (PK) | Auto-generated |
| type | Enum (task_assigned, due_date_approaching) | Not null |
| message | String | Not null |
| read | Boolean | Default: false |
| user_id | UUID (FK → User) | Not null |
| task_id | UUID (FK → Task)? | Optional (reference) |
| created_at | DateTime | Auto, not null |

**Indexes**: `user_id`, `read`

## Enums

### TaskStatus

- `pending`
- `in_progress`
- `completed`

### Priority

- `low`
- `medium`
- `high`

### NotificationType

- `task_assigned`
- `due_date_approaching`

### ProjectRole

- `owner`
- `member`

## Validation Rules

- **User email**: Valid email format, max 255 chars, unique
- **User password**: Min 8 chars, at least 1 uppercase, 1 lowercase, 1 digit
- **Task title**: Min 1 char, max 255 chars
- **Task description**: Max 5000 chars
- **Project name**: Min 1 char, max 255 chars
- **Comment content**: Min 1 char, max 5000 chars
- **Due date**: Must be a valid ISO 8601 datetime; can be in the past
  (allowed but flagged as overdue)

## State Transitions

### Task Status

```
pending ──→ in_progress ──→ completed
  ↑              │               │
  └──────────────┘───────────────┘
```
All transitions are allowed in both directions (free-form). No workflow
restrictions in v1.
