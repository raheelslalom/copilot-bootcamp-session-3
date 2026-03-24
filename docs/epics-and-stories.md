# Todo App Upgrade Epics and Stories

## MVP

### Epic: Task Metadata Management

#### Story: Add due date field to task creation and editing

##### Acceptance Criteria

- Users can optionally provide a due date when creating a task.
- Users can update or clear the due date when editing a task.
- Due dates are stored in `YYYY-MM-DD` format.
- Tasks without a due date continue to save successfully.

##### Technical Requirements

- Update the task form in `packages/frontend/src/TaskForm.js` to keep using the date input for due date entry during create and edit flows.
- Normalize edit-mode due date values to `YYYY-MM-DD` before binding them to the form state.
- Preserve `due_date` in the request payloads sent from `packages/frontend/src/App.js` to the existing `POST /api/tasks` and `PUT /api/tasks/:id` endpoints.
- Keep backend request handling in `packages/backend/src/app.js` compatible with nullable `due_date` values in create and update operations.

#### Story: Add priority selection with default P3

##### Acceptance Criteria

- Users can assign a priority of `P1`, `P2`, or `P3` when creating a task.
- If a user does not choose a priority, the task defaults to `P3`.
- Users can update a task priority when editing a task.
- Only `P1`, `P2`, and `P3` are accepted as valid priority values.

##### Technical Requirements

- Add priority form state and a constrained UI control in `packages/frontend/src/TaskForm.js` for `P1`, `P2`, and `P3`.
- Include `priority` in task payloads created in `packages/frontend/src/App.js` for create and edit requests.
- Extend the tasks table in `packages/backend/src/app.js` to store `priority` with a default value of `P3`.
- Update backend create, read, and update handlers in `packages/backend/src/app.js` so task responses always include `priority`.
- Expand backend tests in `packages/backend/__tests__/tasks.test.js` and frontend mocks in `packages/frontend/src/__tests__/App.test.js` to cover priority defaults and allowed values.

### Epic: Task Validation and Persistence

#### Story: Enforce required title and tolerant due date handling

##### Acceptance Criteria

- A task cannot be created or updated without a non-empty title.
- Invalid due date values are ignored and treated as absent.
- Valid due date values continue to save successfully.
- Validation behavior is consistent between task creation and task editing.

##### Technical Requirements

- Keep title validation in `packages/frontend/src/TaskForm.js` and ensure it runs for both create and edit submissions.
- Add due date validation before submission so malformed values are omitted from the payload rather than blocking the user.
- Add defensive date validation in `packages/backend/src/app.js` so invalid `due_date` inputs are stored as `NULL`.
- Add or update automated coverage in `packages/backend/__tests__/tasks.test.js` and `packages/frontend/src/__tests__/App.test.js` for empty-title rejection and invalid-date handling.

#### Story: Persist task metadata through the existing task API

##### Acceptance Criteria

- Task data returned by the system includes title, description, due date, priority, and completion status.
- New task metadata persists across create, read, update, and completion flows.
- No external storage or new backend services are introduced.
- The implementation remains compatible with the current app architecture.

##### Technical Requirements

- Continue using the existing React-to-Express API flow implemented in `packages/frontend/src/App.js` and `packages/backend/src/app.js`.
- Extend backend SQL statements in `packages/backend/src/app.js` so `priority` is included in inserts, selects, and updates.
- Preserve the current in-repo storage model by keeping data within the existing application stack and avoiding third-party storage integrations.
- Ensure task objects consumed by `packages/frontend/src/TaskList.js` and `packages/frontend/src/TaskForm.js` include the new metadata fields without breaking current completion and delete actions.

### Epic: Task Filtering Experience

#### Story: Add All, Today, and Overdue task filters

##### Acceptance Criteria

- Users can switch between `All`, `Today`, and `Overdue` task views.
- The `All` view shows every task.
- The `Today` view shows only incomplete tasks due today.
- The `Overdue` view shows only incomplete tasks with due dates earlier than today.

##### Technical Requirements

- Add filter state and filter controls to the main task experience in `packages/frontend/src/App.js` or `packages/frontend/src/TaskList.js`.
- Derive Today and Overdue filtering against `due_date` values using consistent local-date comparisons in the frontend.
- If filtering is implemented server-side, extend `GET /api/tasks` in `packages/backend/src/app.js` with explicit filter parameters while preserving existing behavior for unfiltered requests.
- Update frontend tests in `packages/frontend/src/__tests__/App.test.js` to cover switching between filter views and verifying visible tasks.

#### Story: Apply completion rules to filtered task views

##### Acceptance Criteria

- Completed tasks remain visible in the `All` view.
- Completed tasks are hidden from the `Today` view.
- Completed tasks are hidden from the `Overdue` view.
- Toggling a task to completed updates its visibility correctly in the active filter.

##### Technical Requirements

- Ensure filter evaluation in `packages/frontend/src/TaskList.js` or `packages/frontend/src/App.js` checks both `completed` and `due_date`.
- Keep the existing completion toggle flow in `packages/frontend/src/TaskList.js` and `PATCH /api/tasks/:id` in `packages/backend/src/app.js` compatible with active filters.
- Refresh or recompute the visible task list after completion changes so filtered views update immediately.
- Add test coverage in `packages/frontend/src/__tests__/App.test.js` for completion visibility rules under each filter.

## Post-MVP

### Epic: Task Status Visibility

#### Story: Highlight overdue tasks in the task list

##### Acceptance Criteria

- Incomplete overdue tasks are visually distinguished from other tasks.
- Tasks that are not overdue do not use the overdue highlight treatment.
- Completed tasks do not appear as overdue in filtered views.
- Overdue highlighting updates automatically as task data changes.

##### Technical Requirements

- Add overdue state styling in `packages/frontend/src/TaskList.js` using the existing Material UI-based task row presentation.
- Base overdue determination on `due_date`, `completed`, and the current local date.
- Keep the styling implementation isolated to presentation logic so it does not alter task persistence behavior.
- Add frontend test coverage for overdue presentation rules in `packages/frontend/src/__tests__/App.test.js`.

#### Story: Add visual priority badges to task items

##### Acceptance Criteria

- Each task displays its priority using a visible badge or label.
- `P1`, `P2`, and `P3` are visually distinguishable from one another.
- Tasks without an explicit stored priority render as `P3`.
- Priority display remains visible in both normal and filtered task views.

##### Technical Requirements

- Extend task row rendering in `packages/frontend/src/TaskList.js` to display `priority` alongside the existing due date chip and action buttons.
- Reuse the current Material UI `Chip`-based presentation pattern for consistency with existing due date metadata.
- Ensure task API responses from `packages/backend/src/app.js` always provide a usable priority value for rendering.
- Add frontend tests in `packages/frontend/src/__tests__/App.test.js` to verify badge rendering for each priority level.

### Epic: Task Ordering

#### Story: Sort tasks by urgency and due date rules

##### Acceptance Criteria

- Overdue tasks appear before non-overdue tasks.
- Within the same overdue status, tasks are ordered by priority from `P1` to `P3`.
- Tasks with the same overdue status and priority are ordered by ascending due date.
- Tasks without due dates appear after tasks with due dates.

##### Technical Requirements

- Implement a deterministic sort routine that evaluates overdue status, priority rank, due date, and undated fallback in that order.
- Apply the sort either in `packages/frontend/src/TaskList.js` before rendering or in `GET /api/tasks` within `packages/backend/src/app.js`, but keep one source of truth for the ordering rules.
- If backend sorting is used, update the current SQL ordering in `packages/backend/src/app.js` to incorporate priority and overdue precedence.
- Add backend and frontend test coverage for ordering behavior in `packages/backend/__tests__/tasks.test.js` and `packages/frontend/src/__tests__/App.test.js`.