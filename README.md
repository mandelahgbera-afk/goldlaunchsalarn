# Salarn — Crypto Copy-Trading Platform

> React 18 · Vite 5 · Supabase · Tailwind CSS v4 · Framer Motion · Vercel

---

## What's Included

| Page | Users | Admins |
|---|---|---|
| Dashboard | Live market + portfolio overview | ✓ |
| Portfolio | Holdings + P&L | ✓ |
| Trade | Buy / sell crypto | ✓ |
| Copy Trading | Follow top traders | ✓ |
| Transactions | Deposit / withdraw (with email OTP) | ✓ |
| Withdrawal Processing | Real-time animated status page | ✓ |
| Admin Dashboard | Platform overview | ✓ |
| Manage Users | Balance, roles, wallet | ✓ |
| Manage Cryptos | Prices, add/remove coins | ✓ |
| Manage Traders | Approve copy traders | ✓ |
| Admin Transactions | Approve / reject all requests | ✓ |
| Platform Settings | Global config | ✓ |

---

## STEP 1 — Create Your Supabase Project

1. Go to [supabase.com](https://supabase.com) → **New Project**
2. Choose a region close to your users
3. Set a strong database password (save it — you'll need it later)
4. Wait ~2 minutes for provisioning

---

## STEP 2 — Run the Database Schema

1. In Supabase: **SQL Editor** → **New Query**
2. Paste the entire contents of `SUPABASE_SCHEMA.sql`
3. Click **Run** (green button, top right)
4. You should see `Success. No rows returned`

The schema creates all tables, triggers, RLS policies, indexes, and Realtime config.
It is **fully idempotent** — safe to re-run at any time without data loss.

---

## STEP 3 — Configure Auth URLs  ←  SIGNUP BREAKS WITHOUT THIS

Go to **Supabase → Authentication → URL Configuration**

### 3a. Site URL
Set this to your production domain:
```
https://your-app.vercel.app
```

### 3b. Additional Redirect URLs
Click **Add URL** and add ALL of these (one per line):
```
https://your-app.vercel.app/auth/callback
http://localhost:3000/auth/callback
```

> **Why `/auth/callback` must be here**: Supabase's PKCE flow sends a `code` parameter
> to this URL. Your `emailRedirectTo` in the signup call points to this path.
> Supabase will reject the signup with "Redirect URL not allowed" if this exact path
> is not listed — the root domain alone is not enough.

---

## STEP 4 — Configure Email Sending

Go to **Supabase → Authentication → Providers → Email**

### 4a. Basic settings

| Toggle | Value |
|---|---|
| Enable Email Provider | ON |
| Confirm email | ON (recommended for production) |
| Secure email change | ON |
| Double confirm email changes | ON |

> **For fast local testing only**: turn "Confirm email" OFF — users sign in immediately
> after signup with no email required. Turn it back ON before going live.

### 4b. SMTP (required for production — Supabase free tier limited to ~4 emails/hour)

Go to **Supabase → Project Settings → Authentication → SMTP Settings**

| Field | Example (using Resend) |
|---|---|
| Enable Custom SMTP | ON |
| Sender Name | Salarn |
| Sender Email | no-reply@your-domain.com |
| SMTP Host | smtp.resend.com |
| SMTP Port | 465 |
| SMTP User | resend |
| SMTP Password | re_xxxxxxxxxxxx |

After saving: click **Send Test Email** and confirm delivery before continuing.

**Recommended SMTP providers:**
- [Resend](https://resend.com) — free 3,000 emails/month, easiest setup
- [Mailgun](https://mailgun.com) — free 5,000 emails/month for 3 months
- [SendGrid](https://sendgrid.com) — free 100 emails/day

---

## STEP 5 — Set Up Email Templates

Go to **Supabase → Authentication → Email Templates**

You will see 4 templates. Configure each one below.

---

### Template 1: Confirm signup

**Subject line:**
```
Confirm your Salarn account
```

**Body (HTML):**
→ Copy the full contents of `email-templates/confirm-signup.html` and paste here.

The key variable Supabase injects:
- `{{ .ConfirmationURL }}` — the verification link (already in the template)

---

### Template 2: Magic Link  ← used for withdrawal OTP

**Subject line:**
```
Your Salarn withdrawal verification code
```

**Body (HTML):**
→ Copy the full contents of `email-templates/magic-link-otp.html` and paste here.

Key variables:
- `{{ .Token }}` — the 6-digit code shown in the large box
- `{{ .ConfirmationURL }}` — fallback link (already in the template)

---

### Template 3: Change Email Address

**Subject line:**
```
Confirm your new Salarn email address
```

**Body (HTML):**
→ You can reuse `email-templates/confirm-signup.html` — just change the heading text
  from "Welcome to Salarn" to "Confirm your new email address" if you want.
  The `{{ .ConfirmationURL }}` variable works the same way.

---

### Template 4: Reset Password

**Subject line:**
```
Reset your Salarn password
```

**Body (HTML):**
→ Copy the full contents of `email-templates/reset-password.html` and paste here.

Key variable:
- `{{ .ConfirmationURL }}` — the reset link (already in the template)

---

### After saving templates

1. Click **Save** on each template
2. Do a **fresh signup test** with a real email address
3. Confirm you receive the styled Salarn email
4. Click the link and verify the callback redirects to your dashboard

> If you receive a plain-text email after saving: double-check you pasted into the
> **HTML Body** field, not the plain-text field, and that the template starts with `<!DOCTYPE html>`.

---

## STEP 6 — Get Your API Keys

Go to **Supabase → Project Settings → API**

| Key | Where to find it |
|---|---|
| `VITE_SUPABASE_URL` | "Project URL" — looks like `https://abcdefgh.supabase.co` |
| `VITE_SUPABASE_ANON_KEY` | Under "Project API keys" → `anon` `public` |

Keep the `service_role` key private — it is never used in this app.

---

## STEP 7 — Deploy to Vercel

### 7a. Push to GitHub

Include these files (do NOT put them in .gitignore):
```
src/
public/
email-templates/
index.html
package.json
package-lock.json
vite.config.ts
tsconfig.json
components.json
vercel.json
SUPABASE_SCHEMA.sql
.env.example          ← rename from .env (never commit .env itself)
```

### 7b. Import on Vercel

1. [vercel.com](https://vercel.com) → **Add New → Project** → import your GitHub repo
2. Framework preset: **Vite** (auto-detected from `vite.config.ts`)
3. Build Command: `npm run build` (pre-filled from `vercel.json`)
4. Output Directory: `dist` (pre-filled)
5. **DO NOT deploy yet** — set env vars first

### 7c. Add Environment Variables

In Vercel → **Project Settings → Environment Variables** → add:

| Name | Value | Environments |
|---|---|---|
| `VITE_SUPABASE_URL` | `https://xxxx.supabase.co` | Production, Preview, Development |
| `VITE_SUPABASE_ANON_KEY` | `eyJ...` (anon key) | Production, Preview, Development |
| `VITE_APP_URL` | `https://your-app.vercel.app` | Production |

Then click **Deploy** (or **Redeploy** if you deployed before adding vars).

### 7d. Verify deployment

After deploy completes:
1. Open `https://your-app.vercel.app`
2. Try signing up with a real email address
3. Check your inbox for the styled Salarn email
4. Click the confirmation link → you should land on `/dashboard`

---

## STEP 8 — Set Your Admin Account

1. Sign up with the email you want as admin
2. Confirm the email
3. Go to **Supabase → Table Editor → users**
4. Find your row → click to edit → change `role` from `user` → `admin`
5. Click **Save**
6. Sign out of Salarn → sign back in
7. You will be redirected to `/admin`

---

## Troubleshooting

| What you see | Why it happens | Exact fix |
|---|---|---|
| "Redirect URL not allowed" on signup | `/auth/callback` not in Supabase Additional Redirect URLs | **Auth → URL Configuration → Additional Redirect URLs** → add `https://your-app.vercel.app/auth/callback` |
| `error {}` / blank error on signup | Supabase cannot deliver the email | **Auth → SMTP Settings → Send Test Email** — fix SMTP or disable email confirmation temporarily |
| "Email rate limit hit" | Supabase free email: ~4/hour | Wait 1 hour, or set up custom SMTP |
| Plain white page after deploy | Missing env vars | Check all 3 vars are in Vercel Project Settings and redeploy |
| Signup says "already registered" | Email was used before | Sign in instead; or go to Supabase Auth → Users → delete the old unconfirmed user |
| "Email not confirmed" on sign-in | User signed up but never clicked link | Supabase → Auth → Users → find user → click three-dots menu → Send confirmation email |
| Admin route redirects to `/dashboard` | Role not set to admin | Set `role = 'admin'` in Supabase Table Editor → users |
| 404 on page refresh | SPA rewrite missing | `vercel.json` already has it — make sure the file is committed to git |
| Withdrawal OTP email never arrives | Magic Link template has `shouldCreateUser: false` | Ensure email is already registered. Check SMTP. Check spam folder. |
| Realtime not updating withdrawal status | REPLICA IDENTITY not set | Re-run `SUPABASE_SCHEMA.sql` — the `ALTER TABLE transactions REPLICA IDENTITY FULL` line fixes this |
| Auth callback loop | Site URL mismatch | Site URL in Supabase must be exactly `https://your-app.vercel.app` (no trailing slash) |

---

## Architecture

```
salarn/
├── SUPABASE_SCHEMA.sql           — Run once in Supabase SQL Editor
├── vercel.json                   — Build config + SPA rewrite rule
├── email-templates/
│   ├── confirm-signup.html       — Paste into: Auth → Email Templates → Confirm signup
│   ├── reset-password.html       — Paste into: Auth → Email Templates → Reset Password
│   └── magic-link-otp.html      — Paste into: Auth → Email Templates → Magic Link
└── src/
    ├── lib/
    │   ├── auth.tsx              — AuthContext (race-condition-safe signUp/signIn/signOut)
    │   ├── supabase.ts           — Supabase client
    │   ├── api.ts                — Typed DB operations (balances, trades, portfolio)
    │   └── marketData.ts         — CoinGecko live prices with TTL cache
    ├── pages/
    │   ├── Auth.tsx              — Sign in / Sign up / Forgot password
    │   ├── AuthCallback.tsx      — PKCE code exchange → role-based redirect
    │   ├── Dashboard.tsx         — Live market data + portfolio summary
    │   ├── Portfolio.tsx         — Holdings breakdown
    │   ├── Trade.tsx             — Buy / sell
    │   ├── CopyTrading.tsx       — Follow traders
    │   ├── Transactions.tsx      — History + withdrawal with email OTP
    │   ├── WithdrawalProcessing.tsx — Animated real-time status (Supabase Realtime)
    │   ├── Settings.tsx          — Profile
    │   └── admin/
    │       ├── AdminDashboard.tsx
    │       ├── AdminTransactions.tsx — Approve/reject deposits & withdrawals
    │       ├── ManageUsers.tsx
    │       ├── ManageCryptos.tsx
    │       ├── ManageTraders.tsx
    │       └── PlatformSettings.tsx
    └── components/
        ├── ProtectedRoute.tsx    — Auth guard
        ├── AdminRoute.tsx        — Admin role guard
        ├── ErrorBoundary.tsx     — Catches rendering crashes
        └── layout/               — AppLayout, Sidebar, BottomNav
```

---

## Auth & Security Notes

**PKCE flow** — no tokens in URLs, no implicit grant. Every signup and password reset
uses `emailRedirectTo` → `/auth/callback` → `exchangeCodeForSession`. The code verifier
is stored in `sessionStorage` by supabase-js and consumed once.

**Race-condition safety** — `signInFetchInProgress` is set **before** calling any
`supabase.auth.*` method. The Supabase client fires `onAuthStateChange` _during_ the
`await`, not after. Setting the guard beforehand prevents the listener from double-fetching
the user profile while `signIn` / `signUp` is still running.

**Withdrawal OTP** — uses `supabase.auth.signInWithOtp({ shouldCreateUser: false })` to
re-verify the user's email address before creating a withdrawal transaction. The
same-user guard in `onAuthStateChange` ignores the resulting `SIGNED_IN` event so it
does not interrupt the active session.

**RLS** — all tables have Row Level Security enabled. Users can only read/write their
own rows. Admin operations (approve withdrawals, edit balances) are gated by the
`service_role` key via Supabase server-side policies, not exposed to the frontend.
