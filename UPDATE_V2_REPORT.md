# School Connect Generator — **Update v2** Report

**Build:** `update v2` (cumulative successor to `update v1`)
**Date:** 2026-06-26
**Status:** ✅ Verified — **160/160** automated checks pass · all **8 SQL files** run clean, idempotent, and as an upgrade on a simulated old database (PostgreSQL 17) · **0 inline-script errors** across 73 generated pages (enterprise preset).

> **IMPORTANT — a previously-deployed site is an OLD build.** These fixes do not appear automatically. To get them you must **re-generate** a ZIP from this `update v2` builder, **re-upload** the files to GitHub/Vercel/Cloudflare, and **run the new `database/update-v2-schema.sql`** once in Supabase (after the earlier files). Step-by-step deployment is at the end.

This build is **fully additive** — nothing from any previous build was removed (verified by file diff: *0 files removed*; only `database/update-v2-schema.sql` is new at the source level; the new pages are emitted at build time).

---

## 1. How your 17 issues were solved

### Issue 1 — Dedicated AI-prompt page for CSV questions (3 levels)
New **AI Question Prompts** page (`cbt-prompts.html`). It contains three ready-made prompts — **Simple** (MCQ, true/false), **Intermediate** (adds numeric, short-answer, multiple-response), and **Advanced** (adds matching, ordering, essay). Each prompt produces questions in the **exact CSV format the CBT page accepts**. The teacher clicks **Copy prompt**, pastes it into **any free AI chat** (ChatGPT/Gemini/Copilot), fills in `[TOPIC]/[NUMBER]/[CLASS]`, copies the CSV, **edits it**, saves as `.csv`, and uploads on the CBT page. *The platform itself uses no paid AI.*

### Issue 2 — Detailed, first-timer-friendly page descriptions
Every module page now opens with a collapsible **“ℹ️ What is this page?”** panel giving (a) a plain one-line description, (b) **who uses it**, and (c) **step-by-step how to use it**. A detailed guide is defined for the core pages, and the assistant explains every page in depth.

### Issue 3 — Comprehensive, knowledgeable assistant
The built-in assistant’s knowledge base was greatly expanded: **per-page explanations for every page** (including all the new ones), plus many new **FAQ topics** (AI prompts, entrance exams, admission letters, storage, super-admin, digital library, promotion, dashboards, fee/salary tracking, link thumbnails, bulk export, and the activity-log behaviour). It answers “what is this page?” contextually and suggests related topics.

### Issue 4 — Your details & brand embedded; dedicated developer page
New **About the Developer** page (`developer.html`) — the last page of the site — featuring **Adewale Samson Adeagbo** and the full **HMG Concepts** ecosystem (Academy, Technologies, Media, Gospel) with links to all six sites and WhatsApp. The site **footer** on every page also credits the developer and links to this page.

### Issue 5 — Anonymous exams + instant results/certificates/admission letters
New **Entrance & Assessments** page (`entrance.html`). Mark a CBT exam as an **entrance/assessment** (anyone can sit it without an account). Results appear **instantly**; you can generate each candidate’s **result slip, certificate, and admission letter** — **single or in bulk** — branded with your school logo, address and motto. Admission decisions are computed against a pass mark you set and logged in `admission_letters`.

### Issue 6 — Staff DOB privacy (day & month only)
The staff form now captures **Birth day** and **Birth month** only (no year), stored in new `dob_day` / `dob_month` columns.

### Issue 7 — Class-scoped student picker + typeahead
When picking a student anywhere, you can first choose a **class** (only that class’s students then show) **and** type a few letters of the name to filter instantly. Students remain grouped by class for compactness.

### Issue 8 — Easy “pull marks from Digital Library to report card”
The **Results** page has a **“📚 Pull reading scores”** button that scales Digital-Library quiz scores into a CA column in Results, so they flow straight into the report card.

### Issue 9 — Digital ID card includes full school contact details
The ID card footer now shows the school **address, phone, email and motto** (in addition to logo, photo, QR and full bio).

### Issue 10 — Single and bulk download/export everywhere
Every module page exports **CSV and PDF**; ID cards support **Print ALL** (students or staff); the Entrance page supports **bulk** admission letters and certificates. Single-record actions remain available.

### Issue 11 — Render links as image/video thumbnails
When a Google-Drive/YouTube/image/video **link** is stored in a record, the list automatically renders it as a **clickable image or video thumbnail** (`Super.media.thumb`) — no upload, saving Supabase space.

### Issue 12 — Storage manager (when Supabase nears full)
New **Storage Manager** page (`storage.html`). It shows **each table’s size** (via the `table_sizes()` RPC) and lets an admin **safely purge old, low-value rows** (audit logs, old results, read notifications, old check-ins) via the guarded `purge_old()` RPC — after exporting on Admin Data.

### Issue 13 — Activity Log “no editable form” error fixed
The **Activity Log** is now a proper **read-only** module: it shows the audit trail and **does not display an “Add new” button**, so the error never appears.

### Issue 14 — Birthdays grouped by birth month
The **Birthdays** page has a **“📅 Group by month”** view listing students under each birth month, each with **name and class**.

### Issue 15 — Full student/parent dashboard (admin can view any)
New **Student Dashboard** page (`student-profile.html`) showing a 360° view: **name, admission no, DOB, class, fees & full payment history, attendance summary, awards/behaviour, and results/report card**. A student/parent sees their own; **admins/staff open any student’s dashboard** from a class-grouped picker or the **Dashboard** button on the Students list.

### Issue 16 — Track fees, salaries and overview every dashboard
A `student_overview` view aggregates each student’s fees-paid, results count, attendance and awards; a `staff_salary_overview` view exposes payroll. Admins track fee/payment history per student on the dashboard, salary on HR/Payroll, and a school-wide overview on Analytics.

### Issue 17 — Super-admin (proprietor)
Added a **`super_admin`** role and a `proprietor`/`super_admin` recognised as **super-admin** by `is_super_admin()`. The **proprietor/proprietress is automatically the super-admin** (full access, manage admins, see all dashboards, control storage). The Approvals page can set the `super_admin` role; `set_super_admin()` lets an existing super-admin promote others.

---

## 2. Extra expert / enterprise features added
- **Admission-letter register** (`admission_letters`) with auto reference numbers and decision tracking.
- **Entrance flag & pass mark** on CBT exams (`is_entrance`, `pass_mark`).
- **Brand settings** in `school_settings` (developer name/brand/url) for white-label flexibility.
- **Media thumbnail engine** reused across lists and dashboards.
- **Guarded, safe-list purge** so storage cleanup can never touch core tables.
- **Class-grouped, searchable pickers** reused on the dashboard picker too.
- All **free-tool, no-AI, deterministic** — as required.

---

## 3. What changed, file by file (cumulative — nothing removed)

| File | Change |
|---|---|
| `assets/js/generator.js` | New pages: **`pageCBTPrompts`, `pageEntrance`, `pageStorage`, `pageDeveloper`, `pageStudentProfile`**; ID-card footer with address/phone/email/motto; `super_admin` role in Approvals; `primary`/`accent` added to `window.SCHOOL`; new pages registered (build list, dedicated list, dispatch, pageIds, emits `update-v2-schema.sql`). |
| `assets/js/crud.js` | Staff DOB → **day & month only**; **searchable + class-filter** student pickers (`filterRefByClass`, `filterRefBySearch`); **`renderBirthdaysByMonth`**; **`pullReadingScoresToResults`**; **link thumbnails** in lists; **read-only `activity_log`** schema; **Dashboard** button on student rows. |
| `assets/js/super.js` | New **`media`** thumbnail engine (Drive/YouTube/image/video); ID-card footer (address/phone/email/motto); **assistant KB + PAGE_HELP greatly expanded** (all new pages + FAQs). |
| `assets/js/templates.js` | **`PAGE_GUIDE` + `guideHTML`** (detailed first-timer panels); read-only-aware module toolbar; birthdays “Group by month” + results “Pull reading scores” buttons; icons for new modules; footer credits the developer + links to `developer.html`. |
| `assets/js/catalog.js` | New modules (cbt-prompts, entrance, storage, developer) with clear descriptions; added to all presets; `developer` appended to every preset. |
| `database/update-v2-schema.sql` | **NEW** — staff DOB columns; super-admin role + helpers + `set_super_admin`; `table_sizes()` & guarded `purge_old()`; `admission_letters` + entrance flags; `student_overview` & `staff_salary_overview` views; brand settings; RLS — all idempotent. |
| `verify.sh` | Added section **[20]** with 17 new checks (160 total). |

---

## 4. DEPLOYMENT — clear, unambiguous, step by step

You will (A) generate the ZIP, (B) set up the database in Supabase, (C) put files on GitHub, and (D) host on Vercel or Cloudflare Pages. All on **free** tiers.

### A. Generate your school’s ZIP
1. Open **`builder.html`** from this `update v2` folder in a browser.
2. Fill in the wizard (school name, logo, colours, contact details, modules). Ensure the new modules are ticked: **AI Question Prompts, Entrance & Assessments, Storage Manager, About the Developer**, plus Students, Staff, Classes, Subjects, Digital Library, Promotion, Birthdays.
3. Click **Generate**, then **Download** the ZIP and **unzip** it. The unzipped folder is your school’s website.

### B. Set up the database (Supabase — free)
1. Create a project at **https://supabase.com → New project** (pick a region near you; save the DB password). Wait for provisioning.
2. Open **SQL Editor → New query** and run each file from the unzipped `database/` folder **in this exact order**, one at a time (paste → Run → wait for success → next):
   1. `schema.sql`
   2. `voting-schema.sql`
   3. `cbt-schema.sql`
   4. `reportcard-schema.sql`
   5. `enterprise-schema.sql`
   6. `enhancements-schema.sql`
   7. `update-v1-schema.sql`
   8. **`update-v2-schema.sql`**  ← the new one for this build
   (All are safe to re-run.)
3. Open **Project Settings → API**; copy the **Project URL** and the **anon public** key.
4. Edit **`assets/js/config.js`** (or root `config.js`) in the unzipped site and paste your URL + anon key. Save.
5. **Make the proprietor the super-admin:** in Supabase → **Authentication**, enable **Email**; sign up on your site; then in **Table editor → profiles**, set that account’s `role` to **`super_admin`** (or `proprietor`) and `status` to `approved`.

### C. Put the files on GitHub
1. At **https://github.com**, create a **New repository** (e.g. `my-school`), Public, no README.
2. Choose **“uploading an existing file”** and drag in **ALL files** from the unzipped site folder (including `.nojekyll`, `assets/`, `database/`). Commit.

### D. Host it (pick ONE — both free)
**Vercel:** sign in at **https://vercel.com** with GitHub → **Add New… → Project** → import your repo → Framework **Other**, output **`./`** → **Deploy**.
**Cloudflare Pages:** **https://dash.cloudflare.com → Workers & Pages → Create → Pages → Connect to Git** → pick the repo → build command **blank**, output **`/`** → **Save and Deploy**.
**GitHub Pages (optional):** repo **Settings → Pages** → branch **main**, folder **/ (root)** (the `.nojekyll` file is included).

### E. First-run checklist
1. Sign in; confirm the proprietor account is `super_admin` and `approved`.
2. In **Settings**, set the current session and term.
3. Add **Classes**, then **Subjects** (map each to a teacher), then **Staff**.
4. Add students (or **Import CSV**), print **ID cards**, set **Digital Library** readings.
5. For entrance exams: create a CBT exam (tick *entrance*), share the code, then use **Entrance & Assessments** to generate results & letters.
6. Show new staff the **AI Question Prompts** page to build CBT questions fast.

---

## 5. Verification evidence
- `bash verify.sh` → **Passed: 160   Failed: 0**.
- All **eight** SQL files: **clean install**, **idempotent re-run**, and **upgrade on a simulated old DB** — all error-free on PostgreSQL 17 with the Supabase auth shim.
- v2 SQL confirmed live: staff `dob_day/dob_month`; `table_sizes()` returns table sizes; `purge_old()` correctly rejects non-admins/non-purgeable tables; `admission_letters`, `student_overview`, `staff_salary_overview`, `cbt_exams.is_entrance/pass_mark` present; a **proprietor profile returns `is_super_admin = true`**.
- Full generation (enterprise preset): **110 files / 73 HTML pages, 0 inline-script errors**; new pages `cbt-prompts.html`, `entrance.html`, `storage.html`, `developer.html`, `student-profile.html` all produced with correct content.
- Cumulative diff vs `update v1`: **0 files removed.**
