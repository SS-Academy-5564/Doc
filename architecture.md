# Background Worker

Pulse should have a separate Worker application.

The Worker is responsible for executing monitors on schedule.

Worker flow:

```text
1. Find enabled monitors that are due for checking
2. Send HTTP request
3. Measure response time
4. Parse JSON response
5. Extract value using JSON path
6. Save metric value
7. Evaluate alert rules
8. Create or resolve triggered alerts
9. Update monitor LastCheckedAt
```

The Worker should not contain business logic directly.

It should call BL services:

```text
Worker → BL → DAL
```

---

# Technical Architecture

## Solution Structure

```text
Pulse.sln
│
├── Pulse.API
├── Pulse.BL
├── Pulse.DAL
├── Pulse.Entities
└── Pulse.Worker
```

---

## Pulse.API

Responsible for HTTP endpoints.

Contains:

- Controllers
- Middleware
- Filters
- Extensions
- Program.cs
- appsettings.json

API should call BL services, not DAL directly.

```text
Controller → BL Service → DAL
```

---

## Pulse.BL

Responsible for business logic, DTOs, validation and shared behavior.

Contains:

```text
Services
- Auth
- Organizations
- Members
- Monitors
...

Models
- Requests
- Responses
- DTOs

Common
- Exceptions
- Validators
- Helpers
```

BL responsibilities:

- validate monitor configuration
- parse extracted values
- evaluate alert rules
- map entities to DTOs
- check user permissions
- prepare dashboard data

---

## Pulse.DAL

Responsible for database access using Dapper.

Contains:

```text
Connection
- IDbConnectionFactory
- DbConnectionFactory

Queries
- MonitorQueries
- DashboardQueries
...

Commands
- MonitorCommands
- DashboardCommands
...

Scripts
- Tables
- Indexes
- Seed
```

DAL responsibilities:

- SQL queries
- Dapper mapping
- database reads
- database writes
- transactions

DAL should not contain business logic.

---

## Pulse.Entities

Contains database entities and enums.

---

## Pulse.Worker

Contains background polling worker.

Worker responsibilities:

- execute scheduled monitor checks
- call BL monitoring service
- run continuously in the background

---

# Dependency Direction

```text
Pulse.API     → Pulse.BL
Pulse.Worker  → Pulse.BL
Pulse.BL      → Pulse.DAL, Pulse.Entities
Pulse.DAL     → Pulse.Entities
```
