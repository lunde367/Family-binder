# Family Card Binder

A one-page website for tracking your family's Pokémon cards. It shows every Scarlet & Violet and Mega Evolution card with its picture (loaded live from [TCGdex](https://tcgdex.dev)), tracks what you have and need, and keeps a chase list for each family member.

Everything here can be done from a phone browser.

## Part 1: Put it online (GitHub Pages, free, about 5 minutes)

1. Sign up or sign in at **github.com**.
2. Tap **+** → **New repository**. Name it `Family-binder`, choose **Public**, and tap **Create repository**.
3. On the new repository page, tap **Add file** → **Upload files**, choose `index.html`, and tap **Commit changes**.
4. Go to **Settings** → **Pages**. Under "Build and deployment", set **Source** to *Deploy from a branch*, **Branch** to `main` and folder `/ (root)`, then tap **Save**.
5. After a minute or two your binder is live at
   `https://YOUR-GITHUB-NAME.github.io/Family-binder/`

At this point it works, but it saves on each device separately. Do Part 2 to sync the whole family.

## Part 2: Sync across devices (Supabase, free, about 10 minutes)

### 2a. Create the database
1. Sign up at **supabase.com** and create a **New project** (any name, any password, the region nearest you). Wait for it to finish setting up.
2. Open **SQL Editor** → **New query**, paste the block below, **replace the emails with your family's**, and tap **Run**.

```sql
create table binder_items (
  kind text not null,
  id text not null,
  data jsonb not null,
  updated_at timestamptz default now(),
  primary key (kind, id)
);

alter table binder_items enable row level security;

-- Only these email addresses can read or change the binder.
create policy "family only" on binder_items
  for all to authenticated
  using ((auth.jwt() ->> 'email') in ('parent@example.com', 'other.parent@example.com'))
  with check ((auth.jwt() ->> 'email') in ('parent@example.com', 'other.parent@example.com'));
```

Kids without their own email can sign in on their device with a parent's email.
To add someone later, run the two `create policy` lines again with the new list, after first running `drop policy "family only" on binder_items;`.

### 2b. Allow sign-in links back to your site
1. Go to **Authentication** → **URL Configuration**.
2. Set **Site URL** to your GitHub Pages address from Part 1.
3. Under **Redirect URLs**, add the same address and tap **Save**.

### 2c. Connect the page to the database
1. In Supabase, go to **Project Settings** → **API** (or **Data API**). Copy the **Project URL** and the **anon public** key.
2. In GitHub, open `index.html` and tap the **pencil** (Edit). Near the top, fill in:

```js
window.BINDER_CONFIG = {
  SUPABASE_URL: "https://your-project.supabase.co",
  SUPABASE_ANON_KEY: "your-long-anon-key"
};
```

3. Tap **Commit changes**. Within a minute or two the site asks for an email, sends a sign-in link, and syncs everything you change.

The anon key is meant to be public. The "family only" rule above is what keeps other people out of your data.

## Notes
- Card lists and pictures come from TCGdex and are cached on each device for a week. New sets appear once TCGdex adds them.
- To track an older era too, edit the line `const SERIES = ["me","sv"];` in `index.html`, e.g. `["me","sv","swsh"]` for Sword & Shield.
- Anything saved before you set up sync stays on that one device; it does not move into the shared binder.
