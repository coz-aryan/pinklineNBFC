# First Salary

Student site, dashboard and admin panel for First Salary. Static site on Vercel, backend on Supabase.

## What's here

| Path | What it is |
|---|---|
| `index.html` | Public site (React build) + curriculum page (`#curriculum`) + student dashboard (`#student-*`) |
| `admin/index.html` | Admin panel at `/admin/` |
| `assets/supabase.js` | `@supabase/supabase-js` 2.117.2 (UMD, vendored, pinned) |
| `vercel.json` | Security headers, caching, clean URLs |

No build step. Vercel serves the files as they are.

## Backend (Supabase project `first-salary`, ref `gftdncrmvxnrfgizkqsi`, region ap-south-1)

- **Auth:** email one-time code (or magic link). New sign-ups get a `students` row automatically.
- **Admins:** emails listed in `private.admin_emails` get `app_metadata.role = "admin"` on sign-up. Add one with SQL:
  `insert into private.admin_emails values ('someone@example.com');`
  For an account that already exists, also run:
  `update auth.users set raw_app_meta_data = raw_app_meta_data || '{"role":"admin"}' where email = 'someone@example.com';`
- **Tables (public, all with RLS):** `tracks`, `students`, `kyc_documents`, `modules`, `lessons`, `questions`, `lesson_progress`, `quiz_attempts`, `activity_days`, `cohorts`, `talks`, `talk_registrations`, `drives`, `drive_applications`.
- **Rules enforced in the database:**
  - Students read and edit only their own rows. They can't change their KYC status, cohort or email.
  - Quiz answers never reach the browser. `get_quiz` returns questions, and `submit_quiz` scores on the server.
  - Module quizzes require the aptitude check, all lessons in the module, and a pass (≥70%) on the previous module. The final test requires every module.
  - Passing the final test assigns a cohort automatically.
  - Placement drives are visible only to students in a cohort.
- **Storage:** private bucket `kyc` (PDF/JPG/PNG, 5 MB). Each student can only use the folder named after their user id. Admins view files through short-lived signed links.
- **RPCs:** `get_quiz`, `submit_quiz`, `get_leaderboard`, `my_stats`, `admin_students` (admins only).

The full schema is in the project's migration history (Supabase dashboard → Database → Migrations), or run `supabase db pull`.

## Keys

`index.html` and `admin/index.html` use the project URL and the **publishable** key. These are safe in the browser because RLS protects the data. Never put the secret / service-role key in this repo.

## One-time setup in the Supabase dashboard

1. **Authentication → URL Configuration:** set Site URL to the Vercel domain. Add `https://<domain>/` and `https://<domain>/admin/` to Redirect URLs.
2. **Authentication → Email Templates → Magic Link** and **Confirm signup:** add `{{ .Token }}` so emails include the 6-digit code.
3. **Authentication → SMTP:** connect a real email provider before launch. The built-in sender is heavily rate-limited.

## Content

Use the admin panel (`/admin/` → Content) to rename modules, add lesson video links and add quiz questions. Modules without their own questions use the shared sample bank.
