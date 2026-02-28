# Court Booking System — Client Setup Guide

A complete pickleball / sports court booking system with:
- Online booking form for customers
- Admin dashboard for managing bookings, courts, and schedules
- Email confirmations + reschedule notifications
- Telegram notifications to the owner
- QR-code payment support (GCash / GoTyme)
- Double-booking prevention

---

## Step 1 — Create a Supabase Project

1. Go to [https://supabase.com](https://supabase.com) and sign up (free)
2. Click **New Project**, choose a name and region
3. Go to **Project Settings → API**
4. Copy your **Project URL** and **anon public key**

---

## Step 2 — Configure `supabase-config.js`

Open `supabase-config.js` and replace the two placeholders:

```js
const SUPABASE_URL  = 'https://YOUR_PROJECT_REF.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR_ANON_PUBLIC_KEY';
```

---

## Step 3 — Create the Database Tables

In **Supabase Dashboard → SQL Editor**, run this SQL to create all required tables:

```sql
-- Courts
create table if not exists public.courts (
  id text primary key,
  name text not null,
  description text,
  rate numeric not null default 0,
  blocked boolean not null default false,
  feats jsonb default '[]',
  photo text
);

-- Bookings
create table if not exists public.bookings (
  ref text primary key,
  full_name text not null,
  contact_number text not null,
  email text,
  court_id text not null,
  court_name text not null,
  date date not null,
  slots jsonb not null default '[]',
  start_time text,
  end_time text,
  duration numeric,
  rate numeric,
  total numeric,
  payment_method text,
  payment_flow text,
  payment_status text default 'unpaid',
  payment_provider text,
  payment_session_id text,
  payment_checkout_url text,
  paid_at timestamptz,
  gcash_ref text,
  downpayment numeric,
  status text not null default 'pending',
  created_at timestamptz not null default now()
);

-- Settings (key-value store)
create table if not exists public.settings (
  key text primary key,
  value text
);

-- Blocked dates
create table if not exists public.blocked_dates (
  date date primary key,
  created_at timestamptz not null default now()
);

-- Accounts (admin users metadata)
create table if not exists public.accounts (
  id text primary key,
  username text,
  password text,
  role text,
  full_name text,
  email text,
  created_at timestamptz
);
```

---

## Step 4 — Run the Security Migrations

Still in SQL Editor, run each file in order:

1. `supabase/migrations/001_prevent_double_booking.sql` — prevents double-bookings at DB level
2. `supabase/migrations/002_enable_rls.sql` — enables Row Level Security
3. `supabase/migrations/20260227_payment_security.sql` — payment sessions table

---

## Step 5 — Create an Admin Account

In **Supabase Dashboard → Authentication → Users**, click **Add User**:
- Enter the admin's **email** and **password**
- This is what they'll use to log in at `login.html`

Then in SQL Editor, insert a matching record in the accounts table:

```sql
insert into public.accounts (id, username, role, full_name, email, created_at)
values (
  gen_random_uuid()::text,
  'admin',
  'admin',
  'Your Name Here',
  'admin@yourdomain.com',   -- same email as the Auth user above
  now()
);
```

---

## Step 6 — Customize the Branding

Search and replace the following across `index.html`, `admin.html`, and `login.html`:

| Placeholder | Replace with |
|-------------|-------------|
| `Your Court Name` | e.g. `Smash Grove` |
| `Your Location` | e.g. `Bambulo Pickleyard` |
| `YOUR COURT NAME` | uppercase version |
| `YOUR COURT NAME DASHBOARD` | uppercase version |

Also update the logo image `f6b5eb3c-a6b6-49ce-981e-d9b127b67ba3.jpg` with the client's actual logo.

---

## Step 7 — Set Up Email Notifications (Optional but Recommended)

Uses [Resend](https://resend.com) — free up to 3,000 emails/month.

1. Sign up at [https://resend.com](https://resend.com) → get API key
2. Install Supabase CLI: `npm install -g supabase`
3. Deploy Edge Functions:
   ```bash
   npx supabase functions deploy send-confirmation-email --project-ref YOUR_PROJECT_REF --no-verify-jwt
   npx supabase functions deploy send-reschedule-email   --project-ref YOUR_PROJECT_REF --no-verify-jwt
   ```
4. Add secrets in Supabase Dashboard → Edge Functions → Manage Secrets:
   - `RESEND_API_KEY` = your Resend API key
   - `EMAIL_FROM` = `Your Court Name <onboarding@resend.dev>` (or verified domain)

---

## Step 8 — Set Up Telegram Notifications (Optional)

Sends a Telegram message to the owner whenever a customer books.

1. Message [@BotFather](https://t.me/BotFather) on Telegram → `/newbot` → get your **Bot Token**
2. Message your new bot once, then open: `https://api.telegram.org/botYOUR_TOKEN/getUpdates`
3. Copy the `id` from the `chat` object — that's your **Chat ID**
4. Deploy the Edge Function:
   ```bash
   npx supabase functions deploy send-telegram-notification --project-ref YOUR_PROJECT_REF --no-verify-jwt
   ```
5. Add secrets:
   - `TELEGRAM_BOT_TOKEN` = your bot token
   - `TELEGRAM_CHAT_ID` = your chat ID

---

## Step 9 — Deploy the Website

Options:
- **GitHub Pages** (free) — push this folder to a GitHub repo, enable Pages in Settings
- **Netlify** (free) — drag and drop this folder at [netlify.com](https://netlify.com)
- **Vercel** (free) — connect GitHub repo at [vercel.com](https://vercel.com)
- **cPanel / shared hosting** — upload via File Manager

> After deploying, share the URL with customers for bookings and `/admin.html` for the owner.

---

## File Overview

| File | Purpose |
|------|---------|
| `index.html` | Customer-facing booking form |
| `admin.html` | Admin dashboard (bookings, courts, settings) |
| `login.html` | Admin login page |
| `supabase-config.js` | Database connection + all data operations |
| `style.css` | Shared styles |
| `supabase/migrations/` | SQL files to run in Supabase |
| `supabase/functions/` | Edge Functions (email + Telegram) |
