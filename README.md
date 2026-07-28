[README.md](https://github.com/user-attachments/files/30452880/README.md)
# Launchpad Hub

An onboarding hub with topics, in-app slide viewer, quizzes, and progress tracking —
backed by Supabase, with two access levels: **viewer** (read + track own progress)
and **editor** (can also add/edit/delete topics).

This guide assumes you've never used Supabase before. It takes about 20–30 minutes
the first time.

---

## 1. Create your Supabase project

1. Go to [supabase.com](https://supabase.com) and sign up (free tier is enough for this).
2. Click **New project**. Pick an organization, name it (e.g. `launchpad-hub`), set a
   database password (save it somewhere — you likely won't need it again, but keep it
   safe), pick a region close to your team, and click **Create new project**.
3. Wait a minute or two while Supabase provisions the project.

## 2. Set up the database

1. In your project, open **SQL Editor** in the left sidebar.
2. Click **New query**.
3. Open `supabase/schema.sql` from this folder, copy its entire contents, paste it into
   the editor, and click **Run**.
4. You should see "Success. No rows returned." This created four tables
   (`profiles`, `topics`, `progress`, `quiz_scores`) and the security rules that
   separate viewers from editors.

## 3. Get your API keys

1. In the left sidebar, go to **Project Settings → API**.
2. You'll need two values:
   - **Project URL** (looks like `https://xxxxx.supabase.co`)
   - **anon / public** key (a long string — this is the "publishable" key, it's safe
     to use in the browser)
3. Keep this tab open, you'll paste these in a moment.

**Important:** never use the `service_role` key in the app — that one bypasses all
security rules and must stay secret.

## 4. Configure email sign-up (optional but recommended for internal use)

By default, Supabase requires people to click a confirmation link in their email
before they can sign in. For an internal tool this is usually fine to leave on. If you
want people to be able to sign up and use the app immediately without checking email:

1. Go to **Authentication → Providers → Email**.
2. Turn off **Confirm email**.
3. Save.

## 5. Run the app locally

You'll need [Node.js](https://nodejs.org) installed (version 18 or newer).

1. Unzip/open this folder in a terminal.
2. Install dependencies:
   ```
   npm install
   ```
3. Copy the environment template and fill in your values from step 3:
   ```
   cp .env.example .env
   ```
   Then edit `.env`:
   ```
   VITE_SUPABASE_URL=https://xxxxx.supabase.co
   VITE_SUPABASE_ANON_KEY=your-anon-public-key
   ```
4. Start the app:
   ```
   npm run dev
   ```
5. Open the URL it prints (usually `http://localhost:5173`).

## 6. Create your first account and become an editor

1. In the running app, sign up with your work email and a password.
2. Sign in.
3. You'll see the hub, but as a **viewer** — no "Edit content" button yet, and no
   topics (the list starts empty).
4. Go back to the Supabase dashboard → **Table Editor** → `profiles` table.
5. Find your row (matched by email), click into the `role` cell, and change it from
   `viewer` to `editor`. Save.
6. Refresh the app. You'll now see the "Edit content" button and a "Load starter
   content" button (since there are no topics yet) — click it to pre-fill the hub with
   the default topics, which you can then edit freely.

To make someone else an editor later, repeat steps 4–5 for their row once they've
signed up. Everyone else who signs up stays a viewer automatically — that's the
default set by the database trigger.

## 7. Deploy it so your team can use it

The easiest free option is [Vercel](https://vercel.com):

1. Push this project to a GitHub repository (create a new repo, then in this folder:
   `git init`, `git add .`, `git commit -m "Launchpad hub"`, then follow GitHub's
   instructions to push it).
2. On [vercel.com](https://vercel.com), sign in with GitHub, click **Add New → Project**,
   and pick your repo.
3. Vercel will detect it's a Vite project automatically. Before deploying, open
   **Environment Variables** and add the same two values from your `.env`:
   - `VITE_SUPABASE_URL`
   - `VITE_SUPABASE_ANON_KEY`
4. Click **Deploy**. After a minute you'll get a live URL you can share with your team.

(Netlify works the same way, if you prefer it — the steps are equivalent.)

## Notes on the two access levels

- **Viewer**: can sign in, browse topics, view slides, take quizzes, and mark topics
  complete. Cannot add, edit, or delete topics — the "Edit content" button is hidden,
  and the database rules (Row Level Security) block any edit attempt even if someone
  tried to call the API directly.
- **Editor**: everything a viewer can do, plus adding/editing/deleting topics, slides,
  links, tips, and quiz questions.
- There's no in-app way to promote someone to editor — that's intentional, so nobody
  can grant themselves access. It's done by whoever manages the Supabase project, in
  the Table Editor, as described in step 6.

## What's stored where

- `topics` — one row per card, holding the title, description, icon, slides, links,
  tips, and quiz as a single JSON blob. Editable only by editors.
- `progress` — one row per person per topic, just a `completed` flag. Each person only
  sees and edits their own rows.
- `quiz_scores` — the latest quiz result per person per topic, same privacy rule.
- `profiles` — one row per person, holding their role. People can read their own role;
  only the project owner can change it, from the Supabase dashboard.
