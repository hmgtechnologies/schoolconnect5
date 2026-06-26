# School Connect Generator — **Update v1** Report

**Build:** `update v1` (cumulative successor to `connect repair v3`)
**Date:** 2026-06-26
**Status:** ✅ Verified — 143/143 automated checks pass · all 7 SQL files run clean, idempotent, and as an upgrade on a simulated old database · 0 inline-script errors across 68 generated pages (enterprise preset).

> **IMPORTANT — your live `gosa` site is an OLD build.** None of these fixes appear on a previously-deployed site automatically. To get them you must **re-generate** a ZIP from this `update v1` builder, **re-upload** the files to your GitHub repo (or Vercel/Cloudflare), and **run the new `database/update-v1-schema.sql`** once in Supabase. Step-by-step instructions are at the end of this document.

This build is **fully additive** — nothing from any previous build was removed. The only brand-new file is `database/update-v1-schema.sql`; everything else is an enhancement of existing files. Verified by file diff: *0 files removed*.

---

## 1. How your 15 issues were solved

### Issue 1 — Register subjects, map to teacher, term, session
- **Subjects** is now its **own page**. Each subject is registered once with *code, department (dropdown), level, and a Subject Teacher chosen from the staff list* (so a subject is mapped to a teacher without typing).
- **Term** and **Session** are global lists kept in the `school_settings` / `lookups` tables and appear as **dropdowns** everywhere they are needed (results, scheme of work, timetable, fees, promotion). Set them once under **Settings**; every page then offers them as a choice.

### Issue 2 — Classes and Subjects as separate pages; class teacher from dropdown
- The old combined "Classes & Subjects" module was **split into two pages: Classes and Subjects**.
- On the **Classes** page, **Class teacher is a dropdown of teaching staff** (no typing). It also offers Level (dropdown) and Capacity.

### Issue 3 — Never re-type anything already on the platform
- Every reference field is now a **dropdown/list sourced from the database**: students, staff, classes, subjects, terms, sessions, departments, audiences.
- Picking a record **auto-fills related fields** (e.g. choosing a student fills their class and DOB).

### Issue 4 — Richer staff details + auto-extract approved teachers
- The **Staff** form now captures: **Staff type (teaching / non-teaching), Subject(s) taught (dropdown), Highest qualification (dropdown), Religion, Marital status, Gender, Date of birth, Address, Photo (Drive link), Role/Designation, Department (dropdown)** plus the existing fields.
- When a teacher **signs up and is approved**, a database trigger (`extract_staff_from_profile`) **automatically creates a matching Staff record** linked to their account. Admin can then enrich it.

### Issue 5 — Auto admission number actually enabled (and not typed)
- The **Admission No** field is now **read-only and labelled "(auto)"** — you cannot type it. On save, the `gen_admission_no` trigger produces `STD/2026/0001`, `STD/2026/0002`, …
- The same is now true for staff: a **Staff No** is auto-generated (`STF/2026/0001`) by the new `gen_staff_no` trigger. *Verified live on PostgreSQL 17.*

### Issue 6 — Attendance from QR check-in (no one-by-one)
- The **Attendance** page has a **"📲 Pull from QR Check-in"** button. It reads today's QR self-check-ins, marks every scanned student **PRESENT** in one click, and avoids duplicates. Hand-entry is still available for exceptions.

### Issue 7 — Scheme of work per teacher/subject + weekly confirmation
- The **Scheme of Work** page lets each teacher enter the term's topics per subject/class/week (teacher, subject, class chosen from dropdowns). Each row has a **"Taught this week (confirm)"** checkbox; the date of confirmation is stored so admin can track coverage.

### Issue 8 — Clearer, unambiguous sections
- Module names and descriptions were rewritten to be plain and explicit — e.g. *"Results / Scores"*, *"Rewards & Badges (PBIS)"*, and the **Promotion** page now carries an inline explainer banner describing exactly what each button does.
- The built-in **ℹ️ Help assistant** explains every page in context.

### Issue 9 — Digital Library
- New **Digital Library** page. A teacher posts a book/resource by **pasting its online link** (Google Drive or web — *no upload*), optionally attaches a few **comprehension questions**, and sets a **max score**.
- Students click **Read**, take the **auto-marked quiz**, and their score is saved to `reading_scores`. That score **can count toward their grade** (it is stored with subject/class so it can be added as continuous assessment).

### Issue 10 — Automated promotion / graduation
- The **Promotion** page has **"⚙ Auto-promote (by exam)"**. You set a **pass benchmark (%)**, session/term, and the graduating class. The system computes each active student's **term average** and drafts **promote / repeat / graduate / pending** decisions.
- These are **drafts only** — the admin can **edit any row**, then click **"✅ Apply promotions"** to move students up (graduates become `graduated`). Nothing changes until you apply.

### Issue 11 — Grouped student dropdowns + bulk CSV import
- Student dropdowns are now **grouped by class** (`<optgroup>`) for compact, fast navigation.
- **"⬆ Import CSV"** lets you bulk-register students (or any module) from a spreadsheet. A ready-made **`students_import_template.csv`** is downloadable from the page. The CSV is **parsed in the browser and the file is never stored** on Supabase — only the extracted records are saved.

### Issue 12 — Export records as PDF / CSV
- Every module page now has **"⬇ Export CSV"** *and* **"📄 Export PDF"** (a clean, branded printable table via the browser's Save-as-PDF). Restricted to staff/admin.

### Issue 13 — Certificate signature via Drive link; no direct uploads
- Across the platform, photos and signatures are taken via **Google Drive / web links**, with explicit *"no upload (saves storage)"* guidance, to protect the Supabase free-tier storage limit.

### Issue 14 — Professional digital ID card, for staff too
- The ID card was redesigned to be **professional and complete**: header with logo + motto, coloured card-type band (**STUDENT / STAFF IDENTITY CARD**), photo, full detail table (ID no, class/arm or designation/department/type, gender, phone, blood group), session, signature line, QR code, "if found, return to office" footer.
- A **Card type selector (Student / Staff)** lets you generate **staff ID cards** too, including **Print ALL**.

### Issue 15 — Flyer designer (colours, fonts, layouts, UI/UX, text)
- The **Flyer** page is now a full designer: choose **layout (Gradient / Banner / Sidebar / Minimal), font, primary/accent/text colours**, and **edit every text element** (title, tagline, headline, bullet list, call-to-action). Live preview + print/Save-as-PDF.

---

## 2. Extra expert / enterprise features added

Because you asked me to think ahead, this build also adds:

1. **Auto staff numbering** (`STF/<year>/####`) — parity with student admission numbers.
2. **Sensible promotion ladder** (Nursery → Primary → JSS → SSS) used by auto-promotion, fully overridable by admin.
3. **Reading-score pipeline** (`reading_scores`) that links digital-library quizzes to grades.
4. **Reference filters** — e.g. only *teaching* staff appear in the "class teacher" / "subject teacher" dropdowns, preventing data-entry mistakes.
5. **Chunked, safe bulk import** (200 rows per request) to stay within free-tier request limits.
6. **Branded PDF exports** for compliance/record-keeping with zero paid libraries.
7. **Idempotent, upgrade-safe migration** — the new schema can be applied to an existing live database without data loss (verified on a simulated old DB).
8. **CSV template generator** so non-technical staff can import students correctly first time.

All of this stays **100% free-tool, no-AI, deterministic/rules-based**, exactly as required.

---

## 3. What changed, file by file (cumulative — nothing removed)

| File | Change |
|---|---|
| `assets/js/crud.js` | Split classes/subjects schemas; class & subject teacher = staff dropdowns; expanded staff fields; read-only auto admission/staff no; `groupBy` (optgroup) student dropdowns; `refFilter`; **`importCSV` + `_parseCSV`**; **`exportPDF`**; **`autoPromote` + `applyPromotions`**; **`importAttendanceFromCheckin`**; **`pushReadingScore`**; **`PromoUI`** modal; digital-library & promotion schemas. |
| `assets/js/generator.js` | New **`pageDigitalLibrary`**; staff/student tabs + richer fields in **ID-card page**; full **flyer designer**; emits `update-v1-schema.sql`, `students_import_template.csv` (root + db); `subjects` added to page list & `digital_library` to dedicated pages; `studentsImportTemplate()`. |
| `assets/js/super.js` | Professional, complete **ID card** (student + staff); fully **customisable flyer** (4 layouts, colours, fonts, editable text). |
| `assets/js/templates.js` | Module toolbar now has **Export PDF, Import CSV, CSV template, Pull-from-QR, Auto-promote, Apply-promotions** buttons + a promotion explainer banner. |
| `assets/js/catalog.js` | Classes/Subjects/Digital-Library split into separate modules; clearer descriptions; presets updated. |
| `database/update-v1-schema.sql` | **NEW** — staff fields + auto staff no; teacher→staff extraction trigger; promotion `average`; `digital_library` + `reading_scores`; RLS; all idempotent. |
| `verify.sh` | Added section **[19]** with 17 new checks (143 total). |

---

## 4. DEPLOYMENT — clear, unambiguous, step by step

You will (A) generate the ZIP, (B) set up the database in Supabase, (C) put the files on GitHub, and (D) host on Vercel or Cloudflare Pages. Everything below uses **free** tiers.

### A. Generate your school's ZIP
1. Open **`builder.html`** from this `update v1` folder in a browser (double-click it, or host the folder).
2. Fill in the wizard (school name, logo, colours, modules). Make sure **Subjects**, **Digital Library**, **Promotion**, **Classes**, **Staff**, **Attendance** are ticked.
3. Click **Generate** and **Download** the ZIP. **Unzip** it — this unzipped folder is your school's website.

### B. Set up the database (Supabase — free)
1. Go to **https://supabase.com** → **New project**. Choose a name, a strong DB password, a region near you. Wait until it finishes provisioning.
2. In the project, open **SQL Editor → New query**. Open each file from the unzipped `database/` folder and **run them in this exact order**, one at a time (paste, click **Run**, wait for success, then next):
   1. `schema.sql`
   2. `voting-schema.sql`
   3. `cbt-schema.sql`
   4. `reportcard-schema.sql`
   5. `enterprise-schema.sql`
   6. `enhancements-schema.sql`
   7. **`update-v1-schema.sql`**  ← the new one for this build
   Each should end with a green "Success"/status row. (They are safe to re-run if needed.)
3. Open **Project Settings → API**. Copy your **Project URL** and **anon public** key.
4. In the unzipped site, open **`assets/js/config.js`** (or `config.js` at the root) in a text editor and paste your URL and anon key into the two placeholders. Save.
5. (Recommended) In Supabase → **Authentication → Providers**, enable **Email**. Create your first admin: sign up on your site, then in **Table editor → profiles**, set that user's `role` to `admin` and `status` to `approved`.

### C. Put the files on GitHub
1. Create a free account at **https://github.com** and click **New repository** (e.g. `my-school`). Keep it **Public**, do **not** add a README.
2. On the new repo page choose **"uploading an existing file"**, then **drag in ALL files from the unzipped site folder** (including `.nojekyll`, the `assets/` and `database/` folders). Commit.
   - *Tip:* keep the `database/` SQL in the repo for your records; it is only run in Supabase, never served.

### D. Host it (pick ONE — both free)

**Option 1 — Vercel**
1. Sign in at **https://vercel.com** with GitHub → **Add New… → Project** → import your repo.
2. Framework preset: **Other**. Leave build command empty, output directory **`./`** (root). Click **Deploy**.
3. Your live URL appears (e.g. `my-school.vercel.app`). The included `vercel.json` handles routing.

**Option 2 — Cloudflare Pages**
1. Sign in at **https://dash.cloudflare.com** → **Workers & Pages → Create → Pages → Connect to Git** → pick your repo.
2. Build command: **(leave blank)**. Build output directory: **`/`**. **Save and Deploy**.
3. Your live URL appears (e.g. `my-school.pages.dev`). The included `_headers` file is applied automatically.

**Option 3 — GitHub Pages** (also free)
1. Repo → **Settings → Pages** → Source: **Deploy from a branch** → Branch **main**, folder **/ (root)** → Save. The `.nojekyll` file is already included so assets load correctly.

### E. First-run checklist
1. Visit your live URL, sign up, and confirm your admin account is `approved`.
2. Under **Settings**, set the **current session and term**.
3. Add **Classes**, then **Subjects** (map each to a teacher), then **Staff**.
4. Add students manually or **Import CSV** (download the template first).
5. Print **ID cards**, set **Digital Library** readings, and at term end use **Auto-promote**.

---

## 5. Verification evidence
- `bash verify.sh` → **Passed: 143   Failed: 0**.
- All seven SQL files: **clean install**, **idempotent re-run**, and **upgrade on a simulated old DB** — all error-free on PostgreSQL 17 with the Supabase auth shim.
- Triggers confirmed live: student `STD/2026/0001`, staff `STF/2026/0001`, and **approved teacher profile auto-created a Staff row** (`staff_type: teaching, status: active`).
- Full generation (enterprise preset): **104 files / 68 HTML pages, 0 inline-script errors**; new pages `digital_library.html`, `subjects.html`, `classes.html`, redesigned `idcards.html` and `flyer.html`, and `students_import_template.csv` all produced.
- Cumulative diff vs `connect repair v3`: **0 files removed.**
