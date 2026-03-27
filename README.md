# 📋 MEAuditSync — Team Deployment Guide

This guide gets your whole team on the same live app in about 15 minutes.

---

## What you'll set up

| Part | Service | Cost |
|------|---------|------|
| App hosting (URL) | GitHub Pages | Free |
| Real-time database | Supabase | Free (up to 50,000 rows) |

---

## STEP 1 — Create your Supabase database

1. Go to **https://supabase.com** and click **Start your project**
2. Sign in with GitHub (or create a free account)
3. Click **New project**, give it a name like `auditsync`, choose a region close to you, set a database password, click **Create new project**
4. Wait ~2 minutes for it to provision

### Create the tables

5. In your Supabase project, click **SQL Editor** in the left sidebar
6. Click **New query**, paste the SQL below, and click **Run**:

```sql
-- Counts table (one row per audit session)
create table if not exists auditsync_counts (
  session_id   text primary key,
  counts       jsonb default '{}',
  act_expiry   jsonb default '{}',
  act_desc     jsonb default '{}',
  counted_by   jsonb default '{}',
  counted_at   jsonb default '{}',
  updated_at   timestamptz default now(),
  updated_by   text default ''
);

-- Audit log table
create table if not exists auditsync_audit (
  id          bigserial primary key,
  session_id  text not null,
  ts          text,
  action      text,
  product     text,
  sku         text,
  old_val     integer,
  new_val     integer,
  by          text,
  notes       text,
  created_at  timestamptz default now()
);

-- Team members table
create table if not exists auditsync_team (
  session_id  text primary key,
  members     jsonb default '[]',
  updated_at  timestamptz default now()
);

-- Enable real-time for all three tables
alter publication supabase_realtime add table auditsync_counts;
alter publication supabase_realtime add table auditsync_audit;
alter publication supabase_realtime add table auditsync_team;

-- Row-level security: allow all operations with the anon key
alter table auditsync_counts enable row level security;
alter table auditsync_audit  enable row level security;
alter table auditsync_team   enable row level security;

create policy "Allow all for anon" on auditsync_counts for all using (true) with check (true);
create policy "Allow all for anon" on auditsync_audit  for all using (true) with check (true);
create policy "Allow all for anon" on auditsync_team   for all using (true) with check (true);
```

### Get your credentials

7. Click **Project Settings** (gear icon) → **API**
8. Copy two values:
   - **Project URL** — looks like `https://abcdefgh.supabase.co`
   - **anon / public key** — the long `eyJ...` string under "Project API keys"

---

## STEP 2 — Add your credentials to the app

1. Open `index.html` in a text editor (Notepad, VS Code, etc.)
2. Find this section near the top (around line 15):

```javascript
window.SUPABASE_URL = '';
window.SUPABASE_ANON_KEY = '';
```

3. Replace with your actual values:

```javascript
window.SUPABASE_URL = 'https://abcdefgh.supabase.co';
window.SUPABASE_ANON_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
```

4. Save the file.

---

## STEP 3 — Host on GitHub Pages

1. Go to **https://github.com** and sign in (or create a free account)
2. Click the **+** icon → **New repository**
3. Name it `auditsync` (or anything you like)
4. Set it to **Public**, leave everything else as default, click **Create repository**
5. On the next page, click **uploading an existing file**
6. Drag and drop **both files** from this folder:
   - `index.html`
   - `_config.yml`
   - `README.md`
7. Click **Commit changes**

### Enable GitHub Pages

8. Click **Settings** tab in your repository
9. Click **Pages** in the left sidebar
10. Under **Source**, select **Deploy from a branch**
11. Under **Branch**, select `main` and `/ (root)`, click **Save**
12. Wait ~2 minutes, then refresh the page
13. You'll see: **"Your site is live at https://YOUR-USERNAME.github.io/auditsync"** 🎉

---

## STEP 4 — Share with your team

Send your team the URL: `https://YOUR-USERNAME.github.io/auditsync`

That's it! Everyone opens the same URL and:
- Counts sync live across all devices
- The **● Live** indicator appears in the top bar when connected
- Anyone's count update appears on everyone's screen within 1–2 seconds
- Install it as an app: tap **📲 Install app** button (Android/PC) or Share → Add to Home Screen (iPhone)

---

## How sync works

| Action | What happens |
|--------|-------------|
| Any team member enters a count | All other devices update within ~1 second |
| Internet drops | App keeps working offline, syncs when reconnected |
| Two people count the same item simultaneously | Last write wins (the most recent timestamp) |
| Open on a new device | Pulls the latest counts from the cloud automatically |

---

## Troubleshooting

**"Sync error" shows in the header**
- Check that your `SUPABASE_URL` and `SUPABASE_ANON_KEY` are correct (no extra spaces)
- Make sure the SQL tables were created successfully

**Changes not appearing on other devices**
- Make sure real-time was enabled in the SQL (the `alter publication` lines)
- Check Supabase → Realtime → Inspector to verify events are firing

**Want a different session/audit per location?**
- Change `SESSION_ID` in `index.html` (search for `const SESSION_ID =`) to a unique string per location, e.g. `'branch_manila_2026'`

---

## Updating the app

When a new version of the app is available:
1. Replace `index.html` in your GitHub repository with the new file
2. Make sure your `SUPABASE_URL` and `SUPABASE_ANON_KEY` are still in place
3. GitHub Pages will redeploy automatically within 2 minutes
