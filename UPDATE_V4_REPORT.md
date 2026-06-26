# School Connect Generator — **Update v4** Report

**Build:** `update v4` (cumulative successor to `update v2`)
**Date:** 2026-06-26
**Status:** ✅ Verified — **168/168** automated checks pass · all **9 SQL files** run clean, idempotent, and as an upgrade on a simulated old database (PostgreSQL 17) · **0 inline-script errors** across 81 generated pages (enterprise preset).

> **IMPORTANT — a previously-deployed site is an OLD build.** These fixes do not appear automatically. To get them you must **re-generate** a ZIP from this `update v4` builder, **re-upload** the files to GitHub/Vercel/Cloudflare, and **run the new `database/update-v4-schema.sql`** once in Supabase (after the earlier files). Full deployment steps are at the end.

This build is **fully additive and progressive** — nothing from any previous build was removed (verified by file diff: *0 files removed*; only `database/update-v4-schema.sql` is new at the source level; the new pages/features are emitted at build time and bundled into the school ZIP).

---

## 1. How your 5 issues were solved

### Issue 1 — Assistant explains each page (what / does / who / advantages / benefits)
The in-app assistant now carries a structured **`PAGE_INFO`** knowledge base. For every page it answers, in full sentences, **what the page is, exactly what it does, who uses it, its advantages, and the benefit to the school** — so a brand-new user understands everything at a glance. The **ℹ️ Help** top-bar button and the 💬 assistant both render this rich, multi-section explanation, and the in-page **“What is this page?”** panel now also shows advantages and benefits.

### Issue 2 — Professional, international-standard flyer tools
The Flyer designer gained a professional toolkit:
- **Two new premium templates** — *Poster* (gradient, ribbon, frosted-glass content card, pill CTA) and *Elegant* (serif, award-style double border) — alongside the existing Gradient/Banner/Sidebar/Minimal.
- **One-click colour palettes** (Royal blue & gold, Emerald, Crimson, Violet, Teal, Slate, Sunset).
- **Paper/size presets** — A4, A5 handbill, Instagram square, WhatsApp/IG story, landscape banner.
- **Decorations** — corner ribbon (with session text), badge/pill, decorative background pattern, optional contact bar.
- Full text control + live preview + print/Save-as-PDF.

### Issue 3 — Professional, international-standard digital ID cards
The ID-card generator now offers **three professional templates** — *Horizontal* (enhanced classic), *Vertical* (portrait/lanyard with large photo + centred QR), and *Corporate* (dark premium with accent edge) — plus **primary/accent colour pickers**. Cards include the full school contact block (address, phone, email, motto), QR, signature line, session and a security footer. Works for **students and staff**, single or **Print ALL**.

### Issue 4 — Parent picked from a dropdown; parent↔child linking
On the **Parent–Child** page, the parent is now chosen from a **searchable dropdown of registered parent accounts** (filtered to `role = parent`), and the student from a **class-grouped, searchable** list. The link is **bidirectional** (a new `parent_child_view` lets you find a parent from a child and vice-versa), so during registration you link parent→student or student→parent without typing any IDs.

### Issue 5 — Track staff salary, bonus, loans, appraisal
A complete **Staff HR suite** was added (PowerSchool/Fedena/HR-module parity):
- **Salary & Payslips** — basic, allowances, bonus, overtime minus tax/pension/loan/other → **AUTO net pay** (also enforced by a DB trigger), with a **printable, professional payslip** for every staff member.
- **Staff Loans & Advances** — principal, monthly EMI, months, amount repaid, status (active/completed/defaulted).
- **Staff Bonuses** — performance, 13th-month, holiday, long-service, with citations and pay status.
- **Staff Appraisals** — weighted 1–10 criteria (punctuality, teaching quality, results, teamwork, conduct) with **auto average & grade band** and a recommendation (promote/retain/train/warn/commend).

---

## 2. Researched competitor features added (PowerSchool, Infinite Campus, Fedena, Edsby, HR modules)
Deep web research identified standout features on leading platforms. Those not already present were added:
- **Standards-based grading Rubrics** (Edsby/PowerSchool parity) — define skills/criteria and a scale.
- **Academic Transcripts** — cumulative per-student records (`transcript_view`) for transfers/applications.
- **Transfer / Leaving Certificates** (Infinite Campus “National Records Exchange” parity) with conduct & reason.
- **Counselling & Wellbeing** session log (Infinite Campus behaviour/health parity).
- **Professional payslip generation, loans with EMI, weighted appraisals** (from dedicated school-HR modules).
- **Touch-friendly, fully mobile-responsive** UI (44px tap targets, 16px inputs to stop iOS zoom, scalable flyer/ID/cert previews, stacking grids, scrollable tables, collapsing sidebar).
- (Already present from earlier builds and retained: per-class QR attendance, auto-grading gradebook, customisable report cards, parent/student portals, analytics, LMS/digital library, admissions, audit log, super-admin, storage manager.)

---

## 3. What changed, file by file (cumulative — nothing removed)

| File | Change |
|---|---|
| `assets/js/super.js` | **`PAGE_INFO`** rich per-page knowledge + **`renderPageInfo`**; `explainPage` now renders the full explanation. **Flyer engine**: `PALETTES`, `SIZES`, *poster* & *elegant* templates, ribbon/badge/pattern/contact-bar options. **ID-card engine**: *vertical* & *corporate* templates + primary/accent colour support. |
| `assets/js/crud.js` | Richer **payroll** schema + **`staff_loans`, `staff_bonus`, `appraisals`** schemas; **parent dropdown** (`role:parent` filter) + bidirectional link; **`printPayslip`**; auto net-pay & auto appraisal-band compute in `save()`; new generic modules **rubrics, transcripts, transfer_cert, counselling**; Payslip action button. |
| `assets/js/generator.js` | Flyer page: palette/size/ribbon/badge/template UI + `applyPalette`. ID-card page: template + colour pickers, passed through `person()`/`printAll()`. New modules added to `pageIds`; emits `update-v4-schema.sql`. |
| `assets/js/templates.js` | `PAGE_GUIDE` enriched with **advantages & benefit**; new HR/parent/idcard/flyer guides; icons for new modules; `guideHTML` renders advantages/benefit. |
| `assets/js/catalog.js` | New modules (payroll, staff_loans, staff_bonus, appraisals, rubrics, transcripts, transfer_cert, counselling) with clear descriptions; added to presets. |
| `assets/css/style.css` | Mobile: 44px tap targets, 16px inputs, full-width modals, scalable card previews, smaller phone padding. |
| `database/update-v4-schema.sql` | **NEW** — payroll columns + auto net-pay trigger; `staff_loans`, `staff_bonus`, `staff_appraisals`; `parent_child_view`; `transcript_view`; RLS — all idempotent & upgrade-safe. |
| `verify.sh` | Added section **[21]** (8 new checks → 168 total). |

---

## 4. DEPLOYMENT — clear, unambiguous, step by step

You will (A) generate the ZIP, (B) set up the database in Supabase, (C) put files on GitHub, and (D) host on Vercel or Cloudflare Pages. All on **free** tiers. Everything is **mobile-friendly** out of the box.

### A. Generate your school’s ZIP
1. Open **`builder.html`** from this `update v4` folder in a browser.
2. Fill in the wizard (school name, logo, colours, contact details, modules). Tick the new modules you want: **Salary & Payslips, Staff Loans, Staff Bonuses, Staff Appraisals, Grading Rubrics, Transcripts, Transfer Certificates, Counselling** (plus all earlier modules).
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
   8. `update-v2-schema.sql`
   9. **`update-v4-schema.sql`**  ← the new one for this build
   (All are safe to re-run.)
3. Open **Project Settings → API**; copy the **Project URL** and the **anon public** key.
4. Edit **`assets/js/config.js`** (or root `config.js`) in the unzipped site and paste your URL + anon key. Save.
5. (Recommended) In Supabase → **Authentication**, enable **Email**; sign up on your site; then in **Table editor → profiles** set the proprietor’s `role` to **`super_admin`** and `status` to `approved`.

### C. Put the files on GitHub
1. At **https://github.com**, create a **New repository** (e.g. `my-school`), Public, no README.
2. Choose **“uploading an existing file”** and drag in **ALL files** from the unzipped site folder (including `.nojekyll`, `assets/`, `database/`). Commit.

### D. Host it (pick ONE — both free)
- **Vercel:** sign in at **https://vercel.com** with GitHub → **Add New… → Project** → import your repo → Framework **Other**, output **`./`** → **Deploy**.
- **Cloudflare Pages:** **https://dash.cloudflare.com → Workers & Pages → Create → Pages → Connect to Git** → pick the repo → build command **blank**, output **`/`** → **Save and Deploy**.
- **GitHub Pages (optional):** repo **Settings → Pages** → branch **main**, folder **/ (root)** (the `.nojekyll` file is included).

### E. First-run checklist
1. Sign in on your phone or laptop (the site is fully responsive).
2. Add **Staff**, then run **Salary & Payslips** (net pay auto-computes) and print a payslip.
3. Record **Loans**, **Bonuses** and an **Appraisal** for a staff member.
4. On **Parent–Child**, link a registered parent to a student from the dropdowns.
5. Design a flyer (try the **Poster** template + a palette) and an ID card (try **Vertical** / **Corporate**).
6. Click the **ℹ️ Help** button on any page — the assistant explains what it is, who uses it, and its benefits.

---

## 5. Verification evidence
- `bash verify.sh` → **Passed: 168   Failed: 0**.
- All **nine** SQL files: **clean install**, **idempotent re-run**, and **upgrade on a simulated old DB** — all error-free on PostgreSQL 17 with the Supabase auth shim. Payroll **net pay auto-computed to 120000** from sample inputs; legacy payroll preserved and back-filled on upgrade.
- Full generation (enterprise preset): **119 files / 81 HTML pages, 0 inline-script errors**; new pages `payroll.html`, `staff_loans.html`, `staff_bonus.html`, `appraisals.html`, `rubrics.html`, `transcripts.html`, `transfer_cert.html`, `counselling.html` and the upgraded `flyer.html` / `idcards.html` all produced with the new tools.
- Cumulative diff vs `update v2`: **0 files removed.**
