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

# Database ER Diagram

Database ER diagram can be viewed by this [url](https://lucid.app/lucidchart/8125c771-2596-49a4-a1bc-67ecf6f0c553/edit?viewport_loc=-348%2C414%2C1662%2C881%2C0_0&invitationId=inv_7a7cdd54-1047-40e3-8d80-b68d521c6ecd "Database Url").
