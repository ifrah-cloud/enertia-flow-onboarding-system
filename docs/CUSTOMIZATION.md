# Customization Guide

This project is designed to be customized without modifying the core application logic. In most implementations, you'll only need to update credentials, branding, copy, and checklist content.

Unless you're extending the application, avoid renaming database tables, column names, storage keys, or internal identifiers. These values are referenced throughout the application and changing them may require corresponding code updates.

---

# File 1 — `src/lib/store.ts`

This file contains the application's external service configuration.

Update the following values before deployment:

```typescript
const SUPA_URL = "https://YOUR_PROJECT_ID.supabase.co";
const SUPA_KEY = "YOUR_SUPABASE_ANON_KEY";
const GHL_WEBHOOK_URL = "YOUR_COMPLETION_WEBHOOK";
const HELP_MESSAGE_WEBHOOK_URL = "YOUR_HELP_WEBHOOK";
```

## What these values do

| Setting | Purpose |
|----------|---------|
| `SUPA_URL` | Connects the application to your Supabase project. |
| `SUPA_KEY` | Allows the application to communicate with your Supabase database. |
| `GHL_WEBHOOK_URL` | Triggered when a client completes onboarding. |
| `HELP_MESSAGE_WEBHOOK_URL` | Receives help requests and simulator submissions. |

For most deployments, these are the only values that need to be changed in this file.

---

# File 2 — `src/components/GamifiedChecklist.tsx`

This file controls the onboarding experience, including the checklist, XP system, completion messages, and simulator placement.

---

## Customize the Checklist

The onboarding checklist is defined inside the `CHECKLIST_DATA` object.

Each section contains one or more checklist items.

Example:

```typescript
export const CHECKLIST_DATA = [
  {
    id: "profile",
    title: "Set Up Your Profile",
    items: [
      {
        id: "p1",
        text: "Complete your profile",
        points: 10,
      },
    ],
  },
];
```

Each checklist item requires:

- A unique ID
- Display text
- XP value

---

### Item ID Guidelines

Every checklist item must have a unique ID.

Recommended naming:

- `p1`
- `p2`
- `ph1`
- `ph2`
- `c1`
- `o1`

> **Important**
>
> Once your application is in use, avoid changing existing item IDs.
>
> Client progress is stored using these identifiers. Renaming or reusing IDs may cause previously completed tasks to appear incomplete.

---

## Pipeline Simulator

The Pipeline Simulator is rendered inside the checklist section with the ID:

```text
pipeline
```

If you'd like the simulator to appear elsewhere, update the section identifier used by the component.

---

## Workflow Simulator

The Workflow Simulator is rendered inside the checklist section with the ID:

```text
contacts
```

You may change this if you want the simulator to appear in another section.

---

## Progress Status Labels

These labels appear throughout the application as the client's onboarding progresses.

Example:

```typescript
Initializing
Booting Up
Calibrating
Fully Operational
```

Feel free to replace these with terminology that matches your brand.

---

## Encouragement Messages

The checklist displays motivational messages as clients make progress.

Example:

```typescript
System booting up.
Good start.
Halfway there.
Almost finished.
All systems go.
```

These messages can be customized to match your preferred tone of voice.

---

## Completion Screen

The completion modal includes:

- Title
- Description
- Call-to-action text

Update these values to reflect your own branding and onboarding experience.

---

# File 3 — `src/pages/ClientLogin.tsx`

This file controls the client login page.

---

## Logo

Replace the logo with your own.

Example:

```tsx
<img src="YOUR_LOGO_URL" />
```

---

## Branding

Update the following text throughout the page:

- Portal name
- Welcome message
- Subtitle
- Button label
- Loading message
- Team login link

These values control the client's first impression of your onboarding portal.

---

# File 4 — `src/pages/Dashboard.tsx`

This file controls the internal dashboard.

---

## Logo

Replace the default logo with your own branding.

---

## Dashboard Branding

Customize:

- Dashboard name
- Page heading
- Page description

These changes affect the appearance of the admin dashboard only.

---

# File 5 — `src/pages/DashboardLogin.tsx`

This file controls access to the internal dashboard.

---

## Authentication

Replace the default password before deploying the application.

For production environments, using a proper authentication provider is strongly recommended.

---

## Branding

You can also customize:

- Dashboard name
- Login subtitle
- Input label
- Login button text

---

# GoHighLevel Configuration

To automatically identify the logged-in client, configure your GoHighLevel Client Portal menu link to include dynamic client information.

Example:

```text
https://onboarding.yourdomain.com/?location={{location.id}}&email={{user.email}}&name={{user.fullName}}
```

When a client opens the portal, the application automatically reads these values to:

- Create the client record
- Associate progress with the correct user
- Save the GoHighLevel Location ID
- Generate direct links back into the client's GoHighLevel account

---

## Email Branding

If you're using GoHighLevel workflows, remember to update:

- Internal notification email addresses
- Agency name
- Email signatures
- Company references
- Any default branding

---

# Internal Components

The following values are referenced throughout the application.

Unless you're extending the platform, they should remain unchanged.

| Component | Reason |
|-----------|--------|
| Database table names | Referenced throughout the application. |
| Database column names | Used by the application's data layer. |
| `currentUser` localStorage key | Used to maintain client sessions. |
| `store_updated` custom event | Used to refresh dashboard data. |
| Existing checklist item IDs | Used to restore saved client progress. |
| Reserved thread identifier (`general`) | Used to identify the general support conversation. |

---

# Designing Your XP System

XP values are completely customizable.

A useful approach is to assign points based on the effort required to complete each task.

| Task Difficulty | Suggested XP |
|-----------------|-------------:|
| Quick task | 5–10 |
| Standard setup | 15–25 |
| Major milestone | 30–50 |

Rather than assigning equal values to every task, consider rewarding items that require more effort or are commonly delayed. This creates a stronger sense of progression throughout the onboarding experience.

---

# Deployment Checklist

Before launching your onboarding portal, verify the following:

- Supabase credentials have been updated.
-  Webhook URLs have been configured.
-  Logos and branding have been replaced.
-  Dashboard password has been changed.
-  Checklist content has been customized.
-  GoHighLevel menu link has been configured.
-  Database tables have been created.
-  A complete onboarding flow has been tested successfully.

Once these steps are complete, your onboarding portal is ready for production.
