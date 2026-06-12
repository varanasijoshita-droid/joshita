# ARIVA website — setup guide (plain language)

This guide lists **every step you need to do yourself** in your own accounts.
No coding required — you'll mostly copy, paste, and click.

You have **four files**. Keep them together in the same folder when you upload them:

| File | What it is |
|---|---|
| `ariva-website.html` | Your main website (the cleaned-up version). |
| `account.html` | The login / register / account page. |
| `supabase-config.js` | A tiny file that holds your keys — **the only place you paste them.** |
| `README.md` | This guide. |

> ⚠️ **First, a tiny rename.** The main site file is delivered as
> `ariva-website-updated.html`. Rename it to **`ariva-website.html`** (replacing your
> old file). The other files expect that exact name.

---

## STEP 1 — Put in your WhatsApp number

Your WhatsApp number currently appears as a placeholder: **`91XXXXXXXXXX`**.

1. Open `ariva-website.html` in any plain text editor (Notepad, TextEdit, VS Code).
2. Use **Find & Replace** (Ctrl/Cmd + H).
3. Find `91XXXXXXXXXX` and replace **all** with your real number.
   - Format: country code + number, **no “+”, no spaces**. Example: `919876543210`.
4. Save the file.

That single replace covers all three spots (the floating green button, the
“Chat on WhatsApp” link, and the booking form). Done.

---

## STEP 2 — Create your free Supabase project

Supabase is the free service that stores logins and booking enquiries.

1. Go to **https://supabase.com** and click **Start your project** → sign in with GitHub or email.
2. Click **New project**.
3. Give it a name (e.g. `ariva`), choose a **database password** (save it somewhere safe),
   pick the region closest to India (e.g. **Mumbai / South Asia**), and click **Create new project**.
4. Wait ~2 minutes while it sets up.

---

## STEP 3 — Copy your keys into `supabase-config.js`

1. In Supabase, open your project → click the **gear icon (Project Settings)** in the left sidebar → **API**.
2. You'll see two things you need:
   - **Project URL** (looks like `https://abcdxyz.supabase.co`)
   - **Project API keys → `anon` `public`** (a long string)
3. Open `supabase-config.js` and paste them in, replacing the placeholders:

   ```js
   window.SUPABASE_URL      = 'https://abcdxyz.supabase.co';   // ← your Project URL
   window.SUPABASE_ANON_KEY = 'paste-your-anon-public-key';    // ← your anon public key
   ```
4. Save the file.

> 🔒 **Safety:** only ever use the **`anon` `public`** key here.
> **Never** paste the **`service_role`** key into any website file — it is a secret admin key.

---

## STEP 4 — Create the database table (copy-paste SQL)

This makes a place to store booking enquiries, and **locks it down so each person can
only ever see their own** (this is “Row Level Security”).

1. In Supabase, click **SQL Editor** in the left sidebar → **New query**.
2. Paste the block below **exactly** and click **Run**.

   ```sql
   -- Table to store booking enquiries
   create table public.enquiries (
     id          uuid primary key default gen_random_uuid(),
     user_id     uuid not null references auth.users(id) on delete cascade,
     name        text,
     phone       text,
     event_date  date,
     guests      int,
     package     text,
     pincode     text,
     notes       text,
     created_at  timestamptz not null default now()
   );

   -- Turn ON Row Level Security
   alter table public.enquiries enable row level security;

   -- A logged-in person may save enquiries only as themselves
   create policy "Users can insert their own enquiries"
     on public.enquiries for insert
     with check (auth.uid() = user_id);

   -- A logged-in person may read only their own enquiries
   create policy "Users can view their own enquiries"
     on public.enquiries for select
     using (auth.uid() = user_id);
   ```
3. You should see “Success. No rows returned.” That's correct.

---

## STEP 5 — Turn on email + password login

1. In Supabase, go to **Authentication** (left sidebar) → **Providers** → **Email**.
2. Make sure **Email** is **enabled**.
3. **Optional but recommended for now:** turn **“Confirm email” OFF**.
   - With it OFF, people are logged in the instant they register (simplest).
   - With it ON, Supabase emails them a confirmation link before they can log in.
   - The page handles both — if it's ON, the user simply sees “check your email”.

---

## STEP 6 — Turn on “Sign in with Google”

This has two sides: Google's side, then Supabase's side.

### 6a — Get the Supabase “callback” address
1. In Supabase: **Authentication → Providers → Google**.
2. Near the top you'll see a **Callback URL (for OAuth)** that looks like:
   `https://abcdxyz.supabase.co/auth/v1/callback`
3. **Copy it** — you'll paste it into Google in a moment.

### 6b — Create Google credentials
1. Go to **https://console.cloud.google.com** and sign in.
2. Top bar → **Select a project** → **New Project** → name it `ariva` → **Create**, then select it.
3. Left menu → **APIs & Services → OAuth consent screen**:
   - Choose **External** → **Create**.
   - Fill **App name** (ARIVA), your **support email**, and a **developer contact email**.
   - Save and continue through the steps (you can leave scopes empty). Publish/“Back to dashboard” when done.
4. Left menu → **APIs & Services → Credentials → Create Credentials → OAuth client ID**:
   - **Application type:** Web application.
   - **Authorized JavaScript origins:** add your website address (e.g. `https://www.arivacart.com`).
   - **Authorized redirect URIs:** paste the **Supabase Callback URL** from step 6a.
   - Click **Create**.
5. Google shows a **Client ID** and **Client secret** — keep this little box open.

### 6c — Give them to Supabase
1. Back in Supabase: **Authentication → Providers → Google**.
2. Toggle Google **ON**, paste the **Client ID** and **Client secret**, and **Save**.

> 📌 Google sign-in only works on a **hosted** website (a real `http`/`https` address),
> not when you open the file directly from your computer. Email + password works either way.

---

## STEP 7 — Tell Supabase your website address

1. In Supabase: **Authentication → URL Configuration**.
2. Set **Site URL** to your live website address (e.g. `https://www.arivacart.com`).
3. Under **Redirect URLs**, add your account page, e.g. `https://www.arivacart.com/account.html`.
4. Save.

(If you're just testing before you have a domain, you can use the address your host gives you.)

---

## STEP 8 — Upload the files

Upload all four files **into the same folder** on whatever host you use
(e.g. Netlify, Vercel, Hostinger, GoDaddy, GitHub Pages):

- `ariva-website.html`  (renamed from the “-updated” file)
- `account.html`
- `supabase-config.js`
- `README.md` (optional to upload — it's just for you)

Then open `ariva-website.html` in a browser and test:
1. Click **Login** in the menu → create an account or use Google.
2. Go to **Book Now**, fill the form, and send — it saves to your database **and** opens WhatsApp.
3. Return to the **account page** to see the enquiry listed.

---

## Where do I read the booking enquiries?

Two easy places:
- **The account page** shows each customer their own enquiries.
- **You (the owner)** can see *all* of them in Supabase: **Table Editor → `enquiries`**.
  You also still get the instant **WhatsApp** message for every booking.

---

## Quick recap of what changed on the site itself
- WhatsApp number is now a single placeholder you replace once (Step 1).
- The menu sections (Act I–V) and blog cards now work with the **keyboard** (Tab, then Enter/Space) and are screen-reader friendly.
- The booking form's guest count now matches your copy: **15 to 50**.
- Booking now **requires login**, saves each enquiry to your database, **and** still opens WhatsApp.
- A new **Login / account page** (`account.html`) with Google and email sign-in.

Nothing about your design, colours, fonts, pricing, or wording was changed.
