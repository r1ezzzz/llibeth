# Lola Libeth — deploy notes

The page is static (works on GitHub Pages), and it saves the **wishes** and the
**shared heart count** in a free Supabase database. Do the Supabase step first so
the keys are baked in before you push.

---

## 1. Supabase (shared saving) — ~5 min, free

1. Go to **supabase.com** → sign in → **New project**.
   Pick any name, set a database password (you won't need it again), choose the
   nearest region, and wait for it to finish setting up.

2. In the left sidebar open **SQL Editor** → **New query**, paste **all** of this,
   and click **Run**:

   ```sql
   -- WISHES ------------------------------------------------------
   create table if not exists public.wishes (
     id         uuid primary key default gen_random_uuid(),
     name       text not null default 'A loved one',
     message    text not null,
     created_at timestamptz not null default now()
   );

   -- HEARTS (one row per tap; the count is how many rows) --------
   create table if not exists public.hearts (
     id         uuid primary key default gen_random_uuid(),
     created_at timestamptz not null default now()
   );

   -- Turn on row-level security ---------------------------------
   alter table public.wishes enable row level security;
   alter table public.hearts enable row level security;

   -- Anyone may READ wishes and ADD a wish (but not edit/delete) -
   create policy "read wishes"  on public.wishes for select using (true);
   create policy "add wishes"   on public.wishes for insert with check (true);

   -- Anyone may READ the heart count and ADD a heart -------------
   create policy "read hearts"  on public.hearts for select using (true);
   create policy "add hearts"   on public.hearts for insert with check (true);
   ```

3. In the sidebar open **Project Settings** (gear) → **API**. Copy these two:
   - **Project URL** — looks like `https://abcd1234.supabase.co`
   - **anon public** key — a long string under "Project API keys"

   > The anon key is *meant* to be public and live in the page — the policies
   > above only let visitors read and add, never edit or delete what's there.

4. Open **card.html**, find the `CONFIG` block near the bottom (in the `<script>`),
   and paste your two values:

   ```js
   const CONFIG = {
     url: 'https://abcd1234.supabase.co',
     key: 'eyJhbGci....your-anon-key....'
   };
   ```

   Save the file. That's it — the page is now shared for everyone.
   (Leave them as `PASTE_...` and the page still works, but saves stay on one device.)

---

## 2. Put it on GitHub Pages

The repo is **github.com/r1ezzzz/lola-libeth**.

### If the repo doesn't exist yet
Create it on GitHub first: **New repository** → owner `r1ezzzz`, name
`lola-libeth`, **Public**, and **do not** add a README/.gitignore/license
(keep it empty), then **Create repository**.

### Push from this folder
I've already run `git init`, committed the files, and set the remote for you.
So you only need:

```bash
git push -u origin main
```

Run that in this folder (`C:\Users\krazy\OneDrive\Desktop\site`). If it's your
first push from this machine, Git will pop open a GitHub sign-in — approve it.

### Turn on Pages
On GitHub: repo → **Settings** → **Pages** →
**Source: Deploy from a branch** → Branch **main**, folder **/ (root)** → **Save**.

Wait ~1 minute. Your site is live at:

**https://r1ezzzz.github.io/lola-libeth/**

That opens the envelope (`index.html`), which then leads into the card.

---

## Notes
- `index.html` = the envelope intro. `card.html` = the main card. `aaaaa.jpg` = the photo.
- Wishes/hearts are open to anyone with the link — fine for a family card. If it
  ever gets spammed, you can delete rows in Supabase → Table Editor.
- To change the photo later, replace `aaaaa.jpg` (or click the frame in-page to
  preview a different one locally).
