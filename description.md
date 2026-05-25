# Pulse — API Metrics Monitoring Platform

## Project Overview

**Pulse** is a configurable API metrics monitoring platform that collects values from external APIs, stores historical metric data, evaluates alert conditions, and visualizes information through a customizable dashboard.

The system allows organizations to monitor business and technical metrics in real time by extracting specific values from responses.

---

## Project Goal

The goal of Pulse is to provide a centralized platform where teams can:

- monitor external API values
- track metric changes over time
- detect abnormal behavior
- configure dashboards
- create alert rules
- analyze trends
- manage organization members and roles

---

## Example Use Cases

Pulse can monitor different types of values.

### Financial Data

- EUR/USD exchange rate
- Bitcoin price
- stock prices
- crypto prices

### Development Metrics

- open GitHub pull requests
- open GitHub issues
- repository stars
- repository forks

### Infrastructure Metrics

- queue size
- failed jobs count
- server health value
- API response time
- disk usage
- memory usage

### Business Metrics

- active users
- orders today
- conversion rate
- payments volume
- failed payments count

### Public APIs

- temperature
- humidity
- wind speed
- traffic data
- energy consumption

---

## Core Concept

Each monitor defines:

- where to fetch data from
- what value to extract
- how often to poll the API
- when to trigger alerts
- how to display the data on the dashboard

Example monitor:

```text
Monitor Name: Open Pull Requests
API URL: https://api.github.com/search/issues?q=repo:owner/repo+type:pr+state:open
JSON Path: total_count
Polling Interval: 5 minutes
Alert Rule: value > 20
```

Pulse periodically calls this API, extracts the `total_count` value, stores it, evaluates alerts, and displays it on the dashboard.

---

## Main Workflow

```text
User creates monitor
        ↓
Pulse periodically calls external API
        ↓
System parses JSON response
        ↓
Value is extracted using JSON Path
        ↓
Metric value is stored in database
        ↓
Alert rules are evaluated
        ↓
Dashboard widgets are updated
```

---

# User Roles

Pulse supports organization-based access control.

## User

A `User` has full organization management permissions.

A User can:

- manage monitors (create / update /delete)
- enable and disable monitors
- configure dashboard widgets (add / remove)
- reorder widgets
- resize widgets
- manage alert rules (edit, delete)
- invite organization members
- change member roles
- remove members
- view all metrics and alerts

## Viewer

A `Viewer` has read-only access.

A Viewer can:

- view dashboard
- view monitors
- view monitor details
- view charts
- view metric history
- view alerts

A Viewer cannot modify any data inside an organization.

---

# Functional Requirements

## 1. Authentication

Users must be able to:

- register
- log in
- log out
- view current profile information

Authentication should use JWT tokens.

After login, the user should be able to access the organization dashboard.

---

## 2. Organization Management

Each user belongs to at least one organization.

An organization contains:

- monitors
- dashboard configuration
- members
- alert rules

Features:

- create organization
- view current organization
- update organization name
- switch between organizations

The organization switcher should be placed at the bottom of the sidebar.

---

## 3. Members Management

Users with the `User` role can manage organization members.

Features:

- view members
- invite member by email
- assign role
- update member role
- remove member from organization

Suggested member fields:

```text
Member:
- Id
- UserId
- OrganizationId
- Role
- JoinedAt
```

Roles:

```text
- User
- Viewer
```

---

# Monitor Features

## 4. Value Monitors

A monitor is the main entity of the system.

It defines how Pulse retrieves a metric value from an external API.

### Monitor Fields

General fields:

- Name
- Description

Request configuration:

- API URL
- HTTP Method
- Headers
- Timeout

Extraction configuration:

- JSON Path
- Value Type

Scheduling:

- Polling Interval

Status:

- Is Enabled
- Created At
- Last Checked At

Advanced features:
- POST requests
- request body
- custom headers
- API token support
- authorization headers

---

## 5. JSON Value Extraction

Pulse extracts values from JSON responses using a JSON path.

Examples:

```text
rates.USD
total_count
data.price
stats.queueSize
weather.temperature
```

Example response:

```json
{
  "rates": {
    "USD": 1.0874
  }
}
```

JSON Path:

```text
rates.USD
```

Extracted value:

```text
1.0874
```

If the value cannot be extracted, the check should be saved as failed with an error message.

---

## 6. Metric Values

Each monitor execution creates a metric value record.

Metric value fields:

- Id
- MonitorId
- Value
- NumericValue
- Timestamp
- ResponseTimeMs
- IsSuccessful
- ErrorMessage

Example metric record:

```json
{
  "monitorId": "eur-usd",
  "value": "1.0874",
  "numericValue": 1.0874,
  "timestampt": "2026-05-21T16:30:00Z",
  "responseTimeMs": 215,
  "isSuccessful": true,
  "errorMessage": null
}
```

The raw value can be stored as string, while numeric values can also be stored in a numeric column for charts and alert evaluation.

---

## 7. Monitors marketplace

The Monitors Marketplace is a feature that allows users to quickly create monitors from predefined templates instead of configuring every monitor manually.

It is a library of ready-to-use monitor configurations created by other users.

# Dashboard Features

## 8. Configurable Dashboard

Pulse provides a configurable main dashboard.

Users can:

- add widgets
- remove widgets
- reorder widgets
- resize widgets
- bind widgets to monitors
- choose widget type

There should be one global time range selector for the whole dashboard.

Available ranges:

- Last 1 hour
- Last 24 hours
- Last 7 days
- Last 30 days

Important UI rule:

```text
Widgets should not have their own time window selector.
There should be only one time window selector near the Add Widget button.
```

---

## 9. Dashboard Widgets

### Metric Card

Displays the latest value of one monitor.

Example:

```text
Open Pull Requests
14
+12% vs 24h ago
```

The card should show:

- monitor name
- source or category
- current value
- increase/decrease compared to 24 hours ago

---

### Line Chart

Shows value changes over time.

Used for:

- exchange rates
- prices
- trends
- slowly changing values

Example:

```text
EUR / USD Rate
```

---

### Bar Chart

Shows values as bars over time.

Used for:

- queue size
- orders per hour
- failed jobs
- active users per period

Example:

```text
Queue Size
```

---

# Alerts Features

## 10. Alert Rules

Users can create threshold-based alert rules for monitors.

Examples:

- Queue Size > 100
- Open Pull Requests > 20
- EUR/USD Rate > 1.10
- Bitcoin Price < 60000
- Temperature <= 0

Severity levels:

```text
- Warning
- Critical
```

---

## 11. Triggered Alerts

When a monitor value violates an alert rule, Pulse creates a triggered alert.

Triggered alert fields:

Statuses:

```text
- Active
- Resolved
```

Example:

```text
High Queue Size
Rule: Queue Size > 100
Current value: 132
Status: Active
```

When the value becomes normal again, the alert can be resolved automatically.
