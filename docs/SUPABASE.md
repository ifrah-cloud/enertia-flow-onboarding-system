# Supabase Database Reference

The onboarding platform uses **four database tables** to store client progress, conversations, and custom service requests. Each table has a single responsibility, making the system easy to maintain and scale.

---

# Database Overview

| Table | Purpose |
|--------|---------|
| **clients** | Stores each client's profile, onboarding progress, XP, and account information. |
| **help_messages** | Stores all conversations between clients and your team, including both general support and checklist-specific discussions. |
| **pipeline_requests** | Stores custom pipeline requests submitted through the Pipeline Simulator. |
| **workflow_requests** | Stores custom workflow and automation requests submitted through the Workflow Simulator. |

---

# Clients

The **Clients** table serves as the primary data source for the onboarding platform. Every client who accesses the portal has a single record in this table.

## Stored Information

- Client name
- Email address
- Role or company
- Overall onboarding progress
- Experience Points (XP)
- Current onboarding status
- Completed checklist items
- GoHighLevel Location ID
- Login history
- Completion status

## How It's Used

When a client accesses the onboarding portal for the first time, the application automatically creates a client record if one does not already exist.

Whenever a checklist item is completed, the application updates:

- Progress percentage
- XP
- Completed checklist items
- Current onboarding status

The admin dashboard reads this table to display:

- Client progress
- Completion percentage
- XP leaderboard
- Onboarding statistics
- Client activity

---

## Key Fields

### Email

The email address is the application's unique identifier for each client.

All progress, conversations, and request history are associated with the client's email address.

> **Best Practice**
>
> Avoid changing a client's email after onboarding has started unless all related records are updated.

---

### Completed Items

Stores every completed checklist item.

The checklist loads this information whenever a client signs in, allowing them to continue exactly where they left off.

If this information is missing, completed progress cannot be restored.

---

### Progress

Stores the client's onboarding completion percentage.

The application automatically recalculates this value whenever checklist progress changes.

Expected range:

- Minimum: **0**
- Maximum: **100**

---

### Location ID

Stores the client's GoHighLevel Location ID.

This value is automatically captured from the onboarding URL during the client's first visit.

It is later used to generate direct links back into the client's GoHighLevel account.

If no Location ID exists, GoHighLevel deep-link functionality is automatically hidden.

---

# Help Messages

The **Help Messages** table stores every conversation between clients and your internal team.

Instead of creating multiple messaging tables, all conversations are stored here.

---

## Conversation Types

### General Support

A dedicated support inbox for questions unrelated to a specific onboarding task.

---

### Checklist Support

Each checklist item can have its own conversation thread.

Examples include:

- Website Setup
- DNS Configuration
- CRM Configuration
- Workflow Questions
- Funnel Questions

This keeps conversations organized and directly associated with the relevant onboarding step.

---

## Read Status

Every conversation tracks whether it has been viewed by:

- The client
- Your team

These values allow the platform to display unread notifications throughout both the client portal and the admin dashboard.

---

# Pipeline Requests

The **Pipeline Requests** table stores custom CRM pipeline requests submitted through the Pipeline Simulator.

Each request contains:

- Client information
- Pipeline name
- Requested stages
- Description
- Current status
- Submission date

The admin dashboard uses this table to manage implementation progress.

---

## Request Status

Each request moves through one of three stages:

- Pending
- In Progress
- Completed

These statuses allow the dashboard to organize active work and completed requests.

---

# Workflow Requests

The **Workflow Requests** table stores custom workflow and automation requests submitted through the Workflow Simulator.

Each request includes:

- Client information
- Workflow name
- Trigger event
- Requested automation actions
- Additional notes
- Current status
- Submission date

Keeping workflow requests separate from pipeline requests allows each service to be managed independently while maintaining a consistent user experience.

---

# Dashboard Integration

The admin dashboard combines information from all four tables to provide a complete overview of every client.

This includes:

- Onboarding progress
- XP
- Completion status
- Support conversations
- Pipeline requests
- Workflow requests
- Overall onboarding activity

Because each table has a clearly defined responsibility, the dashboard can efficiently retrieve only the information required for each section.

---

# Data Relationships

Although the database is intentionally lightweight, every table is connected through the client's email address.

```text
Client
   │
   ├── Progress
   ├── Help Messages
   ├── Pipeline Requests
   └── Workflow Requests
```

This approach keeps the data model simple while ensuring all client activity remains connected.

---

# Application Behavior

Once these four tables exist, the application automatically manages all database operations.

This includes:

- Creating new client records
- Updating onboarding progress
- Saving checklist completion
- Managing support conversations
- Recording pipeline requests
- Recording workflow requests

No manual database maintenance is required during normal operation.

---

# Summary

The onboarding platform uses a lightweight relational structure that separates client information, conversations, and service requests into dedicated tables.

This architecture keeps the application scalable, easy to maintain, and optimized for future expansion while providing the dashboard with everything it needs to manage the client onboarding experience.
