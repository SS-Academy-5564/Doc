---
name: Pulse
colors:
  primary: "#6C63FF"
  secondary: "#64748B"
  surface: "#FFFFFF"
  surface-variant: "#F8FAFC"
  border: "#E2E8F0"
  success: "#22C55E"
  warning: "#F59E0B"
  error: "#EF4444"
  on-surface: "#0F172A"
  on-surface-muted: "#64748B"

typography:
  headline-lg:
    fontFamily: Inter
    fontSize: 36px
    fontWeight: 700

  headline-md:
    fontFamily: Inter
    fontSize: 28px
    fontWeight: 600

  title-md:
    fontFamily: Inter
    fontSize: 20px
    fontWeight: 600

  body-md:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: 400

  body-sm:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: 400

  label-sm:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: 500

rounded:
  sm: 8px
  md: 12px
  lg: 16px

spacing:
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 32px
---

# Design System

## Overview

Pulse is a modern SaaS application for monitoring business metrics retrieved from external APIs.

The interface should feel:

- clean
- lightweight
- developer-focused
- highly readable
- data-oriented
- minimalistic

Avoid enterprise-style visual noise.

Use large whitespace, subtle borders, and simple layouts.

No gradients.
No glassmorphism.
No excessive shadows.

The application should feel similar to:

- Linear
- Vercel
- Stripe Dashboard
- Retool
- Supabase

## Application Layout

### Sidebar

Fixed width: 240px

Contains:

- Pulse logo
- Overview
- Monitors
- Alerts
- Members
- Settings

At the bottom:

- Organization switcher

There is no Logout button.

### Top Actions

Contains:

- Add Widget button
- Global Time Range selector
- Theme toggle
- User menu

The Time Range selector exists only once per page.

Widgets never contain their own time selectors.

Supported ranges:

- Last 24 Hours
- Last 7 Days
- Last 30 Days

## Cards

Cards are the primary UI surface.

Properties:

- white background
- 1px border
- 12px radius
- no shadow

Hover:

- slightly darker border

## Metric Cards

Displayed in the first row of the Overview page.

Each card contains:

- metric title
- current value
- percentage change compared to selected time range
- actions menu (...)

Do not display:

- timestamps
- last updated labels
- mini charts

Example:

Open Pull Requests

14

▲ 12% vs 24h ago

## Charts

### Line Chart

Used for:

- Currency rates
- Bitcoin price
- Pull request count
- Active users

Contains:

- title
- chart
- actions menu (...)

No time range selector.

### Bar Chart

Used for:

- Queue size
- Orders per hour
- Failed jobs

Contains:

- title
- chart
- actions menu (...)

No time range selector.

# Overview Page

## First Row

Metric cards:

- Open Pull Requests
- EUR/USD Rate
- Bitcoin Price
- Server Uptime
- Queue Size

All cards display current value and delta compared to selected time range.

## Second Row

Large Line Chart widget.

Example:

EUR/USD Rate

Contains:

- chart title
- chart
- actions menu (...)

## Third Row

Large Bar Chart widget.

Example:

Queue Size

Contains:

- chart title
- chart
- actions menu (...)

# Monitors Page

Purpose:

Manage configured monitors.

Top area:

- Page title
- Search box
- New Monitor button

Filter chips:

- All
- Enabled
- Disabled
- Error

Main content:

Table

Columns:

- Name
- URL
- Current Value
- Last Check
- Status
- Interval

No logos.
No avatars.
No icons next to monitor names.

Rows are text-focused and highly scannable.

# Monitor Details Page

Header:

- Monitor Name
- Status Badge
- Actions Menu

Sections:

### Configuration

Displays:

- URL
- HTTP Method
- JSON Path
- Interval
- Timeout

### Current Value

Large metric display.

### History

Line chart of metric values over time.

### Alert Rules

Configured alert rules.

# Alerts Page

Purpose:

Display triggered alerts.

Filters:

- All
- Critical
- Warning
- Resolved

Columns:

- Status
- Alert
- Monitor
- Triggered At
- Current Value

Colors:

- Critical → Red
- Warning → Orange
- Resolved → Green

# Members Page

Columns:

- Name
- Email
- Role

Actions:

- Change Role
- Remove Member

Primary action:

- Invite Member

# Buttons

Primary:

- Filled
- Purple (#6C63FF)

Secondary:

- White background
- Border only

Danger:

- Red text

# Icons

Use Lucide icons.

Use icons only for:

- navigation
- actions
- status indicators

Avoid decorative icons.

# Do's

- Use whitespace generously
- Prefer borders over shadows
- Keep charts simple
- Keep data highly scannable
- Use one primary accent color
- Maintain consistent spacing

# Don'ts

- No gradients
- No glass effects
- No neumorphism
- No dashboard clutter
- No mini charts inside metric cards
- No multiple time selectors
- No monitor logos
- No avatars inside tables
- No excessive animations
