# Product Requirements Document (PRD) - Todo App Upgrade

## 1. Overview

We are upgrading the basic Todo app so users can manage deadlines and task importance without adding backend complexity. The current app only supports a task title and completion state. This upgrade introduces optional due dates, simple priority levels, and date-based filters so the app becomes more practical while still remaining a lean, teachable frontend-first implementation with local storage only.

---

## 2. MVP Scope

- Add an optional `dueDate` field for each task.
- Store `dueDate` in ISO `YYYY-MM-DD` format.
- Add a `priority` field with allowed values `P1`, `P2`, and `P3`.
- Default `priority` to `P3` when no value is provided.
- Add task filters for `All`, `Today`, and `Overdue`.
- Keep storage local only, with no backend or external storage changes.
- Require `title` for task creation and editing.
- Treat invalid `dueDate` values as absent instead of failing.
- Show completed tasks in the `All` filter.
- Hide completed tasks in the `Today` and `Overdue` filters.
- Keep the implementation simple enough for an MVP and suitable for teaching/demo purposes.

---

## 3. Post-MVP Scope

- Visually highlight overdue tasks so they stand out in the task list.
- Add sorting rules in this order: overdue tasks first, then priority from `P1` to `P3`, then due date ascending, with undated tasks last.
- Add visual priority badges or color-coding for priority values.

---

## 4. Out of Scope

- Notifications
- Recurring tasks
- Multi-user support
- Keyboard navigation enhancements
- Special accessibility features beyond the current baseline
- Backend changes
- External storage integrations