# Supabase Database Reference

This document covers every table, column, and SQL statement needed to set up the database from scratch.

---

## Overview

| Table | Purpose |
|---|---|
| `clients` | One row per client. Stores login info, checklist progress, XP, status. |
| `help_messages` | Every message in every conversation thread — per-step help and general inbox. |
| `pipeline_requests` | Custom pipeline build requests submitted by clients via the simulator form. |
| `workflow_requests` | Custom workflow/automation build requests submitted via the simulator form. |

---

## SQL Block 1 — clients table

Run this first.

```sql
create table public.clients (
  id uuid default gen_random_uuid() primary key,
  email text unique not null,
  name text,
  role text default 'Client',
  progress int8 default 0,
  xp int8 default 0,
  status text default 'Initializing',
  completed_items jsonb default '[]'::jsonb,
  completed boolean default false,
  location_id text,
  last_login timestamptz,
  login_count int8 default 0,
  created_at timestamptz default now()
);

-- Row level security (open policy — anon key has full access)
alter table clients enable row level security;
create policy "Allow all on clients" on clients for all using (true) with check (true);

-- Index for fast email lookups
create index if not exists idx_clients_email on clients (email);
```

---

## SQL Block 2 — help_messages table

Run this second.

```sql
create table public.help_messages (
  id bigint generated always as identity not null,
  client_email text not null,
  item_id text not null,
  sender text not null,
  message text not null,
  created_at timestamp with time zone default now(),
  read_by_team boolean default false,
  read_by_client boolean default false,
  constraint help_messages_pkey primary key (id),
  constraint help_messages_sender_check check (
    sender = any (array['client'::text, 'team'::text])
  )
);

-- Row level security
alter table help_messages enable row level security;
create policy "Allow all on help_messages" on help_messages for all using (true) with check (true);

-- Index for fast client+item lookups
create index if not exists idx_help_messages_lookup
  on help_messages using btree (client_email, item_id);
```

**Important:** `item_id` is either a checklist item ID (e.g. `p1`, `ph3`) for per-step threads, or the reserved value `'general'` for the client's inbox thread. Every client has one `general` thread and potentially one thread per checklist item they've asked about.

---

## SQL Block 3 — pipeline_requests and workflow_requests tables

Run this third.

```sql
-- Pipeline requests
create table if not exists public.pipeline_requests (
  id bigint generated always as identity not null,
  client_email text not null,
  client_name text not null,
  pipeline_name text not null,
  stages text not null,
  description text,
  status text not null default 'pending',
  created_at timestamp with time zone default now(),
  constraint pipeline_requests_pkey primary key (id),
  constraint pipeline_requests_status_check check (
    status = any (array['pending','in_progress','done'])
  )
);

-- Workflow requests
create table if not exists public.workflow_requests (
  id bigint generated always as identity not null,
  client_email text not null,
  client_name text not null,
  workflow_name text not null,
  trigger_event text not null,
  actions text not null,
  description text,
  status text not null default 'pending',
  created_at timestamp with time zone default now(),
  constraint workflow_requests_pkey primary key (id),
  constraint workflow_requests_status_check check (
    status = any (array['pending','in_progress','done'])
  )
);

-- Row level security
alter table pipeline_requests enable row level security;
alter table workflow_requests enable row level security;
create policy "Allow all on pipeline_requests" on pipeline_requests for all using (true) with check (true);
create policy "Allow all on workflow_requests" on workflow_requests for all using (true) with check (true);

-- Indexes
create index if not exists idx_pipeline_requests_email on pipeline_requests (client_email);
create index if not exists idx_workflow_requests_email on workflow_requests (client_email);
```

---

## Column Reference

### clients

| Column | Type | Notes |
|---|---|---|
| `id` | uuid | Auto-generated primary key |
| `email` | text | Unique. Used as the lookup key everywhere. |
| `name` | text | Client's full name |
| `role` | text | e.g. "Client", "CEO @ Acme" — optional, from login form |
| `progress` | int8 | 0–100. Calculated from completed_items / total items |
| `xp` | int8 | Gamification points. Accumulates as items are checked |
| `status` | text | "Initializing" → "Booting Up" → "Calibrating" → "Fully Operational" |
| `completed_items` | jsonb | Array of item IDs the client has checked e.g. `["p1","p2","ph1"]` |
| `completed` | boolean | true when progress = 100 |
| `location_id` | text | GHL sub-account location ID. Passed from menu link URL param. Used to build the pipeline deep link. |
| `last_login` | timestamptz | Updated on every login |
| `login_count` | int8 | How many times this client has logged in |
| `created_at` | timestamptz | Auto-set on insert |

### help_messages

| Column | Type | Notes |
|---|---|---|
| `id` | bigint | Auto-increment primary key |
| `client_email` | text | Foreign key to clients.email (soft reference) |
| `item_id` | text | Checklist item ID or `'general'` for inbox |
| `sender` | text | `'client'` or `'team'` — enforced by check constraint |
| `message` | text | The message content |
| `created_at` | timestamptz | Auto-set on insert |
| `read_by_team` | boolean | Set to true when team opens the message thread |
| `read_by_client` | boolean | Set to true when client opens the message thread |

### pipeline_requests

| Column | Type | Notes |
|---|---|---|
| `id` | bigint | Auto-increment primary key |
| `client_email` | text | Who submitted the request |
| `client_name` | text | Denormalized for easy display without joins |
| `pipeline_name` | text | What the client wants the pipeline called |
| `stages` | text | Comma-separated stage names |
| `description` | text | Optional extra context |
| `status` | text | `pending` → `in_progress` → `done` |
| `created_at` | timestamptz | Auto-set on insert |

### workflow_requests

| Column | Type | Notes |
|---|---|---|
| `id` | bigint | Auto-increment primary key |
| `client_email` | text | Who submitted the request |
| `client_name` | text | Denormalized for easy display |
| `workflow_name` | text | What the client wants the workflow called |
| `trigger_event` | text | What starts the workflow (client's description) |
| `actions` | text | What the workflow should do (client's description) |
| `description` | text | Optional extra context |
| `status` | text | `pending` → `in_progress` → `done` |
| `created_at` | timestamptz | Auto-set on insert |

---

## Checking Your Tables Are Set Up Correctly

Run this in the SQL Editor to verify all four tables exist:

```sql
select table_name
from information_schema.tables
where table_schema = 'public'
  and table_name in ('clients','help_messages','pipeline_requests','workflow_requests')
order by table_name;
```

You should see all four table names returned.

---

## Resetting All Data (for testing)

Run this to wipe all client data without dropping the tables:

```sql
truncate table help_messages, pipeline_requests, workflow_requests, clients restart identity cascade;
```

**Do not run this in production.** Test environments only.
