# Cloud Architecture Overview

This document provides a simple system context view of the monorepo and a request flow for creating a TODO item. The architecture is intentionally minimal and avoids cloud-provider-specific components.

## System Context

```mermaid
flowchart LR
    user[User]
    frontend[React Frontend\npackages/frontend]
    api[Express API\npackages/backend]
    store[(In-Memory SQLite Store)]

    user -->|Uses via browser| frontend
    frontend -->|HTTP requests| api
    api -->|Reads and writes tasks| store
```

## Sequence: User Creating a TODO

```mermaid
sequenceDiagram
    actor User
    participant Frontend as React Frontend
    participant API as Express API
    participant Store as In-Memory SQLite Store

    User->>Frontend: Enter title, description, and optional due date
    User->>Frontend: Click Add Task
    Frontend->>Frontend: Validate required title
    Frontend->>API: POST /api/tasks
    Note right of Frontend: JSON body includes title, description,<br/>and optional due_date
    API->>API: Validate request payload
    API->>Store: Insert task record
    Store-->>API: Return created task id and fields
    API-->>Frontend: 201 Created with task JSON
    Frontend->>API: GET /api/tasks
    API->>Store: Select current task list
    Store-->>API: Return tasks
    API-->>Frontend: 200 OK with tasks array
    Frontend-->>User: Render updated TODO list
```