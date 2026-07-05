# Setup Guide

This takes about 45 minutes if you follow the steps in order. The most common mistake is skipping ahead — each step depends on the one before it. The second most common mistake is running the SQL after deploying the code. Run the SQL first, always.

---

## What you need before starting

- A GoHighLevel agency account with sub-account access
- A Supabase account — free tier at supabase.com
- The codebase loaded in GHL AI Studio
- A custom domain pointed at your AI Studio project (e.g. `onboarding.youragency.com`)

---

## Step 1 — Create your Supabase project

1. Go to **supabase.com** and sign in
2. Click **New Project**
3. Name it (e.g. `client-onboarding`), set a strong password, pick your region
4. Wait ~1 minute for it to initialize

Once inside:
1. Click **Settings** (bottom left sidebar) → **API**
2. Copy and save two things:
   - **Project URL** — `https://xxxxxxxxxx.supabase.co`
   - **anon public key** — long string starting with `eyJ...`

You'll need both in Step 3.

---

## Step 2 — Run the database migrations

**Do this before touching any code files.**

Three blocks to run:
- `clients` table
- `help_messages` table
- `pipeline_requests` and `workflow_requests` tables

> Why first? Because if you deploy code that references a column that doesn't exist yet, Supabase returns a 204 (no content) on writes and the code treats it as success. Your data disappears silently. Run the SQL first.

---

## Step 3 — Update store.ts

Open `src/lib/store.ts`. The only things you need to change are at the very top:

```typescript
const SUPA_URL = 'https://YOUR_PROJECT_ID.supabase.co';
const SUPA_KEY = 'eyJ...YOUR_ANON_KEY...';
const GHL_WEBHOOK_URL = 'https://services.leadconnectorhq.com/hooks/YOUR_LOCATION/webhook-trigger/YOUR_COMPLETION_ID';
const HELP_MESSAGE_WEBHOOK_URL = 'https://services.leadconnectorhq.com/hooks/YOUR_LOCATION/webhook-trigger/YOUR_HELP_ID';
```

You'll get the webhook URLs in Step 5. Come back and fill them in then.

---

## Step 4 — Set up your GHL workflows

Two workflows. Full details with email copy in [GHL.md](./GHL.md).

**Workflow 1 — Completion**
- Trigger: Inbound Webhook
- Actions: notify internal team + send client a congratulations email
- Copy the webhook URL → paste as `GHL_WEBHOOK_URL` in store.ts

**Workflow 2 — Help + Simulators**
- Trigger: Inbound Webhook
- Actions: conditional branch (pipeline request / workflow request / help message) → notify team + confirm to client
- Copy the webhook URL → paste as `HELP_MESSAGE_WEBHOOK_URL` in store.ts

---

## Step 5 — Set up the GHL custom menu link

This is the entry point for every client. When they click it inside their GHL sub-account, they land on the portal with their name and email pre-filled.

1. Go to **Agency Settings → Custom Menu Links**
2. Set the URL to:
```
https://onboarding.youragency.com/?location={{location.id}}&email={{user.email}}&name={{user.fullName}}
```
3. Add this link to your snapshot

The three URL params do specific things:
- `location` → saved to Supabase `clients.location_id`, used to build the live pipeline deep link inside the checklist
- `email` → pre-fills the login form
- `name` → pre-fills the login form

> If you skip the `location` param, the pipeline deep link inside the checklist won't work. Clients will still see the pipeline section but won't get the direct link to their actual pipeline.

---

## Step 6 — Customize the checklist

Open `src/components/GamifiedChecklist.tsx` and update the `CHECKLIST_DATA` array with your actual onboarding steps. Full customization guide in [CUSTOMIZATION.md](./CUSTOMIZATION.md).

One rule that matters: **never reuse an item ID**. Each `id` is the key used to store which items the client has checked in Supabase. Duplicates will cause checklist items to mirror each other's state.

---

## Step 7 — Set the dashboard password

Open `src/pages/DashboardLogin.tsx` and find:
```typescript
if (password === "testabc2024") {
```
Replace `"testabc2024"` with your own password.

---

## Step 8 — Publish and verify

1. Click **Publish** in AI Studio
2. Work through the full verification checklist in [TESTING.md](./TESTING.md)

The TESTING doc has 30+ specific checks organized into phases — database connectivity, login flow, checklist saving, help messaging, simulators, completion, and dashboard. Run them all before giving any real client the link.

---

## What to add to your snapshot

Once everything is tested:
- The custom menu link (Step 5)
- Any GHL workflow triggers that are sub-account specific

From that point on, every new client sub-account created from the snapshot gets the portal menu link automatically. No per-client setup needed.
