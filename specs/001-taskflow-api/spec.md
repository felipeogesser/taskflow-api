# Feature Specification: TaskFlow API

**Feature Branch**: `001-taskflow-api`

**Created**: 2026-06-25

**Status**: Draft

**Input**: User description: "Construir uma API REST chamada TaskFlow para gerenciamento de tarefas."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - User Registration and Authentication (Priority: P1)

A person can create an account using their email and a password, then log in
to access the system. All subsequent actions require the user to be
authenticated.

**Why this priority**: P1 — Every other feature depends on knowing who the
user is. Without auth, no task can be assigned, no project can be owned, and
no comment can be attributed.

**Independent Test**: A new user can register with a valid email and
password, receive a confirmation of success, then log in with those
credentials and receive an access token. Invalid credentials are rejected
with a clear error.

**Acceptance Scenarios**:

1. **Given** a person with email "user@example.com" and password "Str0ng!Pass",
   **When** they submit a registration request, **Then** their account is
   created and they receive a success response.

2. **Given** a registered user with email "user@example.com",
   **When** they submit a login request with the correct password,
   **Then** they receive an access token.

3. **Given** a registered user,
   **When** they submit a login request with an incorrect password,
   **Then** they receive an authentication error.

4. **Given** an unauthenticated request,
   **When** they attempt to access any protected resource,
   **Then** they receive an unauthorized error.

---

### User Story 2 - Task Management (Priority: P1)

An authenticated user can create, view, edit, and delete tasks. Each task
contains a title, description, status (pending / in progress / completed),
priority (low / medium / high), due date, and an assignee. Users can filter
and search tasks by status, priority, assignee, and project. All listings
are paginated.

**Why this priority**: P1 — Task CRUD is the core value proposition of the
system. Without it, there is no product.

**Independent Test**: An authenticated user creates a task with all fields
filled, sees it appear when listing tasks, edits the title, verifies the
change, filters by status, and deletes the task — confirming it no longer
appears in listings.

**Acceptance Scenarios**:

1. **Given** an authenticated user,
   **When** they create a task with title, description, status, priority,
   due date, and assignee, **Then** the task is saved and returned with a
   unique identifier.

2. **Given** an existing task,
   **When** the user updates any of its fields, **Then** the changes are
   persisted and reflected on the next read.

3. **Given** an existing task owned or visible to the user,
   **When** the user deletes it, **Then** the task is removed and no longer
   appears in any listing.

4. **Given** multiple tasks with different statuses, priorities, and
   assignees, **When** the user applies a filter (e.g., status = "pending"),
   **Then** only matching tasks are returned.

5. **Given** a list of tasks exceeding the page size,
   **When** the user requests a page, **Then** pagination metadata (total
   count, next page, previous page) is returned.

---

### User Story 3 - Project Organization and Team Collaboration (Priority: P2)

An authenticated user can create projects and invite other users to join
them. Within a project, all members can view, create, and update tasks.
Users can see which tasks belong to which project and filter by project.

**Why this priority**: P2 — Collaboration significantly increases value but
the system is still useful for individual task management without it.

**Independent Test**: User A creates a project, invites User B (via email
or username). User B accepts the invitation and can see and create tasks
within the project. User C (not invited) cannot see the project.

**Acceptance Scenarios**:

1. **Given** an authenticated user,
   **When** they create a project with a name and description,
   **Then** the project is created and they are listed as the owner.

2. **Given** a project owner,
   **When** they invite another registered user to the project,
   **Then** the invited user gains access to the project's tasks.

3. **Given** a project member,
   **When** they create a task within that project,
   **Then** the task is associated with the project and visible to all
   members.

4. **Given** a user who is not a member of a project,
   **When** they attempt to access that project's tasks,
   **Then** they receive a forbidden error.

---

### User Story 4 - Comments on Tasks (Priority: P2)

Project members can add comments to tasks to discuss progress, ask
questions, or provide updates. Each comment records the author and
timestamp. Comments can be read alongside the task details.

**Why this priority**: P2 — Comments enable rich collaboration but
tasks are functional without them.

**Independent Test**: A member of a project adds a comment to a task. The
comment appears when the task details are viewed, showing the author name
and the time it was posted.

**Acceptance Scenarios**:

1. **Given** a task in a project the user has access to,
   **When** the user adds a comment with text content,
   **Then** the comment is saved and associated with the task, author, and
   current timestamp.

2. **Given** a task with multiple comments,
   **When** the user views the task details,
   **Then** all comments are displayed in chronological order.

---

### User Story 5 - Internal Notifications (Priority: P3)

Users receive internal (in-app) notifications when a task is assigned to
them or when a task they are responsible for is approaching its due date.
Notifications are stored in the system and can be viewed as a list and
marked as read.

**Why this priority**: P3 — Notifications improve user experience but the
core task management and collaboration features work without them.

**Independent Test**: User A assigns a task to User B. User B's notification
list shows an unread notification about the assignment. User B marks it as
read and it moves to a read state.

**Acceptance Scenarios**:

1. **Given** a task assigned to a user,
   **When** the assignment is saved,
   **Then** an unread notification is created for the assignee.

2. **Given** a task with a due date approaching (within a configurable
   threshold, e.g., 24 hours),
   **When** the system checks periodic conditions,
   **Then** a notification is created for the assignee.

3. **Given** a user with unread notifications,
   **When** they list their notifications,
   **Then** they see all notifications with their status (read/unread) and
   timestamps.

4. **Given** a notification,
   **When** the user marks it as read,
   **Then** its status is updated and it no longer appears as unread.

---

### Edge Cases

- What happens when a user tries to register with an already-used email?
  The system must reject with a clear message.
- What happens when a task's due date is already past when created? The
  system should accept it but may indicate it's overdue.
- What happens when the last member leaves a project? The project should
  have at least one owner; transfer ownership or prevent the action.
- What happens when a task is deleted that has comments? Comments should
  be cascade-deleted or orphaned with a reference to the deleted task.
- What happens when pagination parameters are invalid (negative page,
  excessive page size)? The system should clamp to reasonable defaults
  and return an error for truly invalid values.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow users to register with email and password.
- **FR-002**: System MUST authenticate users and return an access token on
  successful login.
- **FR-003**: System MUST reject unauthenticated requests to protected
  resources with a 401 status.
- **FR-004**: System MUST allow authenticated users to create tasks with
  title, description, status, priority, due date, and assignee.
- **FR-005**: System MUST allow authenticated users to view individual task
  details.
- **FR-006**: System MUST allow authenticated users to list tasks with
  pagination and filtering by status, priority, assignee, and project.
- **FR-007**: System MUST allow authenticated users to update any field of
  a task they have access to.
- **FR-008**: System MUST allow authenticated users to delete tasks they
  have access to.
- **FR-009**: System MUST allow authenticated users to create projects with
  name and description.
- **FR-010**: System MUST allow project owners to invite other registered
  users to join the project.
- **FR-011**: System MUST restrict project task access to project members
  only.
- **FR-012**: System MUST allow project members to add comments to tasks.
- **FR-013**: System MUST return comments in chronological order when
  viewing task details.
- **FR-014**: System MUST create an internal notification when a task is
  assigned to a user.
- **FR-015**: System MUST create an internal notification when a task's due
  date is approaching (within a configurable threshold).
- **FR-016**: System MUST allow users to list their notifications with
  read/unread status.
- **FR-017**: System MUST allow users to mark individual notifications as
  read.
- **FR-018**: System MUST return paginated results for all list endpoints,
  including total count and navigation metadata.
- **FR-019**: System MUST validate input data and return clear error
  messages for invalid fields.

### Key Entities

- **User**: Represents a registered person. Has email (unique), password
  (hashed), display name, and account creation timestamp.
- **Task**: A unit of work. Has title, description, status (enum: pending,
  in_progress, completed), priority (enum: low, medium, high), due date,
  assignee reference, and project reference.
- **Project**: A container for tasks and members. Has name, description,
  owner reference, creation timestamp, and a list of members.
- **Comment**: A message attached to a task. Has content, author reference,
  task reference, and creation timestamp.
- **Notification**: An internal alert for a user. Has type (assignment,
  due_date_approaching), message text, read status, user reference, and
  creation timestamp.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A new user can complete registration and login in under
  30 seconds from a standard internet connection.
- **SC-002**: A user can create a task with all fields, verify it appears
  in listings, edit it, and delete it within 2 minutes of first use.
- **SC-003**: A project owner can invite a user and that user can view
  project tasks within 30 seconds of the invitation being sent.
- **SC-004**: Task list pages with up to 100 items load within 1 second
  under normal conditions.
- **SC-005**: Users can find a specific task using filters and search
  within 3 attempts or fewer.
- **SC-006**: Notifications appear within 1 minute of the triggering event
  (assignment or approaching due date).

## Assumptions

- Password strength requirements follow industry standards (minimum 8
  characters, mix of character types) — specific rules may be refined
  during implementation.
- Invitations to projects are sent by searching for existing registered
  users by email — no external email sending is required (out of scope).
- The due date proximity threshold for notifications defaults to 24 hours
  but is configurable.
- Pagination default page size is 20 items, maximum is 100.
- Task status transitions are free-form (users can move from any status to
  any other) — no workflow restrictions in this version.
- A task may exist without being assigned to a project (personal task).
