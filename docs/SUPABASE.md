Supabase Database Reference

The onboarding platform uses four database tables to store client progress, conversations, and custom service requests. Each table has a single responsibility, making the system easy to maintain and scale.
| Table                 | Purpose                                                                                                                    |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **clients**           | Stores each client's profile, onboarding progress, XP, and account information.                                            |
| **help_messages**     | Stores all conversations between clients and your team, including both general support and checklist-specific discussions. |
| **pipeline_requests** | Stores custom pipeline requests submitted through the Pipeline Simulator.                                                  |
| **workflow_requests** | Stores custom workflow and automation requests submitted through the Workflow Simulator.                                   |
Clients

The Clients table is the primary data source for the onboarding platform. Every client who accesses the portal has a single record in this table.

Stores
Client name
Email address
Role or company
Overall onboarding progress
Experience points (XP)
Current onboarding status
Completed checklist items
GoHighLevel Location ID
Login history
Completion status
How the platform uses it

When a client opens the onboarding portal for the first time, a record is automatically created if one doesn't already exist.

Each time the client completes a checklist item, the system updates:

Progress percentage
XP
Checklist completion
Current onboarding status

The dashboard reads this table to display client progress, completion percentages, leaderboard data, and onboarding statistics.

Important Fields
Email

The email address uniquely identifies every client throughout the application.

All progress, conversations, and request history are associated with this email address.

Recommendation: Avoid changing a client's email after onboarding has started unless all related records are updated as well.

Completed Items

Stores every completed checklist item.

The onboarding checklist uses this information to restore progress whenever the client logs back in.

If this data is missing, the client will appear to have no completed progress.

Progress

Represents overall onboarding completion as a percentage between 0 and 100.

The application automatically recalculates this value whenever checklist progress changes.

Location ID

Stores the client's GoHighLevel Location ID.

This is captured automatically from the onboarding URL during the client's first visit and is later used to generate direct links back into their GoHighLevel account.

If no Location ID exists, features that rely on deep-linking to GoHighLevel are automatically hidden.

Help Messages

The Help Messages table stores every conversation between clients and your internal team.

Rather than creating separate tables for different conversation types, the system stores everything in one place.

Conversation Types

The platform supports two kinds of conversations:

General Support

A dedicated inbox where clients can ask questions unrelated to a specific onboarding task.

Checklist Support

Each checklist item can have its own conversation thread.

For example:

Domain Setup
DNS Configuration
Website Review
Workflow Questions

This keeps conversations organized and directly attached to the relevant onboarding step.

Read Status

Each conversation tracks whether it has been viewed by:

The client
Your team

This allows the application to display unread indicators throughout both the client portal and the admin dashboard.

Pipeline Requests

The Pipeline Requests table stores custom CRM pipeline requests submitted through the Pipeline Simulator.

Each request includes information such as:

Client details
Requested pipeline
Stage structure
Description
Current request status
Submission date

The internal dashboard uses this table to manage implementation progress.

Request Status

Each request progresses through one of three stages:

Pending
In Progress
Completed

These statuses are used by the dashboard to organize work and monitor outstanding requests.

Workflow Requests

The Workflow Requests table functions similarly to Pipeline Requests but is dedicated to automation requests.

Each submission contains:

Client information
Workflow name
Trigger details
Requested automation actions
Additional notes
Current implementation status

This allows your team to manage automation requests separately from pipeline work while maintaining a consistent workflow.

Dashboard Integration

The dashboard combines information from all four tables to provide a complete overview of each client.

This includes:

Onboarding progress
XP
Completion status
Support conversations
Pipeline requests
Workflow requests
Overall onboarding activity

Because each table has a clearly defined responsibility, the dashboard can efficiently retrieve only the information required for each section.

Data Relationships

Although the database is intentionally lightweight, the tables are connected through the client's email address.
Client
   │
   ├── Progress
   ├── Help Messages
   ├── Pipeline Requests
   └── Workflow Requests
   Development Notes

The application expects these four tables to exist before deployment.

Once they are configured:

Client accounts are created automatically during first login.
Progress is saved in real time.
Support conversations are synchronized automatically.
Pipeline and workflow requests are available immediately from the admin dashboard.

No additional database configuration is required during normal operation. The application manages all ongoing reads and updates automatically.
