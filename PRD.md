# Product Requirements Document
## KelliWorks Training Hub — Patricia Montan Onboarding Site

**Version:** 1.0
**Date:** April 14, 2026
**Client:** Kelli Lewis — KelliWorks Accounting Firm LLC
**Prepared by:** Ola Fakomi (AI Assist Work)
**Repository:** https://github.com/ola-fakomi/kelliwork-trainingpage
**Live Site:** Deployed on Vercel (via GitHub integration)

---

## 1. Project Overview

The KelliWorks Training Hub is a custom, cloud-synced onboarding web application built for Patricia Montan, who joined KelliWorks Accounting Firm LLC as an Accounting Operations Liaison starting April 14, 2026. The site delivers a structured, self-paced training curriculum for her first two weeks on the job, tracks her progress across all training modules, and allows her to save notes and questions for each section — all synced to a cloud database so her progress is never lost across browsers or devices.

The site is intentionally built as a single-file static web application (no build tools, no frameworks) so it is easy to maintain, update, and deploy by non-technical staff. Kelli Lewis or her admin (Ola) can update training content by editing one HTML file and pushing to GitHub.

---

## 2. Goals & Objectives

| Goal | Description |
|------|-------------|
| Structured onboarding | Deliver all Week 1 training content in a clear day-by-day format Patricia can work through independently |
| Progress visibility | Give both Patricia and Kelli a real-time view of how far along Patricia is in her training |
| Note capture | Let Patricia write questions and notes per module that she can copy into Slack or refer back to |
| Cross-device sync | Ensure Patricia's progress is saved whether she works from her laptop, a second browser, or a different device |
| Easy content updates | Allow Kelli or Ola to add new Loom video links and update module content with minimal technical effort |
| Scalability | Support future trainees by changing a single `USER_ID` value in the config file |

---

## 3. Stakeholders

| Role | Person | Responsibilities |
|------|--------|-----------------|
| Firm Owner / Decision Maker | Kelli Lewis | Provides training content, Loom videos, final approval on copy |
| Trainee (End User) | Patricia Montan (Yomaira Patricia Montan) | Primary user of the training site |
| Operations Admin | Tim | Mentioned in training content as day-to-day coordinator |
| Project Manager / Developer | Ola Fakomi | Builds, maintains, and updates the site |

---

## 4. Technical Architecture

### 4.1 Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Frontend | Plain HTML5 + CSS3 + Vanilla JavaScript | No frameworks, no build step |
| Styling | Inline `<style>` block inside index.html | CSS custom properties for the design system |
| Database | Supabase (PostgreSQL) | Stores progress data as JSONB per user |
| Hosting | Vercel | Serves the `src/` directory as a static site |
| Source Control | GitHub (`ola-fakomi/kelliwork-trainingpage`) | Main branch auto-deploys to Vercel |
| Video Platform | Loom | All training videos hosted on Loom, embedded as direct links |

### 4.2 Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                  Patricia's Browser                  │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │           src/index.html                     │   │
│  │  ┌─────────────┐  ┌─────────────────────┐   │   │
│  │  │   HTML/CSS   │  │   Vanilla JS Logic  │   │   │
│  │  │  (UI/Layout) │  │  - Progress tracking│   │   │
│  │  │              │  │  - Notes saving     │   │   │
│  │  └─────────────┘  │  - localStorage sync│   │   │
│  │                   │  - Supabase sync    │   │   │
│  │                   └─────────────────────┘   │   │
│  └──────────────────────────────────────────────┘   │
│            │                    │                    │
│     localStorage          supabase-js@2 SDK          │
│      (offline)              (CDN loaded)             │
└────────────────────────────┼────────────────────────┘
                             │ HTTPS / REST
                   ┌─────────▼──────────┐
                   │      Supabase      │
                   │  ┌──────────────┐  │
                   │  │  progress    │  │
                   │  │  table       │  │
                   │  │  (JSONB)     │  │
                   │  └──────────────┘  │
                   └────────────────────┘

GitHub (main branch)
       │
       │ Auto-deploy on push
       ▼
    Vercel
  (serves src/)
```

### 4.3 Data Flow

1. Patricia loads the page → `initCheckboxes()` reads from `localStorage`
2. Checkboxes and notes are restored from local storage immediately (instant, no flicker)
3. `syncFromSupabase()` runs asynchronously in the background
4. If Supabase returns data, it is **merged** into `localData` (localStorage always wins for existing keys — only new keys are pulled from Supabase)
5. On any checkbox change or note save → data is written to `localStorage` **and** upserted to Supabase simultaneously
6. `updateProgress()` recalculates all progress bars, rings, and the top bar on every change

---

## 5. File Structure

```
training-site/
│
├── src/
│   └── index.html              # The entire application — HTML + CSS + JS in one file
│                               # ~1,900+ lines. All training content, styles, and logic here.
│
├── config/
│   └── supabase-config.js      # Supabase credentials and USER_ID
│                               # Loaded by index.html before the main script block
│                               # Change USER_ID here when onboarding a new trainee
│
├── html_Updates/
│   └── tuesday_section_snippet.html   # Content snippet used to update the Tuesday panel
│                                       # Contains both the panel HTML and the Loom video URLs
│                                       # Kept as a reference/working file — not deployed
│
├── Error Image/
│   ├── image.png               # Screenshot: Patricia unable to save notes (bug report)
│   └── image (1).png           # Screenshot: Same bug, second view
│
├── vercel.json                 # Vercel deployment configuration
│                               # Sets src/ as output directory, rewrites all routes to index.html
│
├── README.md                   # Setup guide: Supabase config, table SQL, deployment options
│
└── PRD.md                      # This document
```

---

## 6. Feature Specifications

### 6.1 Navigation & Layout

- **Sticky top bar** — always visible at top of page with: KelliWorks brand badge, site title, trainee name/role/start date, and overall progress bar
- **Sidebar navigation** — left-side nav listing all training days (Mon–Fri Week 1) plus Week 2 preview; each item shows the date badge and a checkmark once the day reaches 100%
- **Day panels** — main content area shows one day at a time; panels are toggled by the `showDay()` JS function without page reloads (SPA-style navigation)

### 6.2 Training Curriculum — Week 1

| Day | Modules | Topics |
|-----|---------|--------|
| Monday (Apr 14) | 1A – Welcome, 1B – Client Roster, 1C – Email Management, 1D – Tech Stack Setup, 1E – Document Sharing | KelliWorks overview, 5 Core Commitments, client roster review, email inbox setup (5 addresses), Gmail/Slack/Asana/QBO/Everhour onboarding, file sharing protocol |
| Tuesday (Apr 15) | 2A – Asana & Daily Workflow, 2B – Keeper: Client Financial Delivery, 2C – Escalation Guide | 4 Asana workflow videos, Keeper financial reports platform, escalation decision framework |
| Wednesday (Apr 16) | 3A – QBO Overview, 3B – Invoicing & Billing, 3C – Bookkeeping & Reconciliation | QuickBooks Online client files, invoicing, A/R tracking, month-end reconciliation |
| Thursday (Apr 17) | 4A – Payroll Processing | Payroll workflow, client coordination, QBO payroll recording, tax corrections |
| Friday (Apr 18) | 5A – State Filings & Sales Tax, 5B – Tax Client Management, 5C – Week 1 Wrap-Up | State/IRS correspondence, tax client management, Jay coordination, debrief call |

### 6.3 Week 2 (Locked)

Week 2 is visible in the sidebar but shows a locked banner. Kelli will unlock it once the KelliWorks Client Intelligence Portal is live. A Loom video walkthrough is planned as a placeholder.

Topics previewed:
- KelliWorks Client Intelligence Portal dashboard walkthrough
- Monthly Close process (BB-Keeper, Travo Client, FTS workflows)
- AR + AP deeper dive and light reconciliation ownership
- Begin taking client communications independently

### 6.4 Loom Video Integration

- Videos are embedded as direct `href` links on play-button elements
- Active videos use `class="loom-slot has-video"` with `class="loom-play"` and status text "▶ Click to watch"
- Pending/upcoming videos use `class="loom-slot"` with `class="loom-play pending"` and status text "📹 Video coming soon"
- Videos open in a new tab (`target="_blank"`)

**Live videos as of v1.0:**

| ID | Title | Day |
|----|-------|-----|
| loom-firmoverview | History of KelliWorks | Monday 1A |
| loom-firmoverview2 | What Do We Do at KelliWorks? | Monday 1A |
| loom-services | Email Mailbox Mgmt — service@kelliworks.com | Monday 1C |
| loom-reports | Email Mailbox Mgmt — reports@kelliworks.com | Monday 1C |
| loom-tue-asana-nav | Asana Project Navigation Guide | Tuesday 2A |
| loom-tue-workflow | Slack, Asana & E-Mail Workflow | Tuesday 2A |
| loom-tue-tasks | Asana — Task KW Emails | Tuesday 2A |
| loom-tue-asana-mgmt | How to Manage Your Workflow in Asana | Tuesday 2A |
| loom-tue-keeper | Reviewing Financials and Keeper (Cash Basis) | Tuesday 2B |
| loom-tue-keeper2 | Understanding New Features in Keeper | Tuesday 2B |

Additional Monday videos (dispatch, inbox clearing, Gmail tips) are referenced in the HTML but their Loom IDs are pending.

### 6.5 Progress Tracking

**Checkbox System:**
- Every training task is a checkbox in a `<ul class="checklist">` list
- Checking/unchecking immediately saves to localStorage and syncs to Supabase
- DAYS object in JS maps each day to its exact list of checkbox IDs

**Progress Indicators (4 types):**
1. **Top bar overall progress** — `#topProgressFill` (width %) + `#topProgressPct` (text) — calculated as total checked / total checkboxes across all 5 days
2. **Week overview cards** — `#wc-{day}` (bar width) + `#wc-{day}-pct` (text) — shown on the Overview panel
3. **Day progress rings** — SVG circle (`#ring-{day}`) with animated `stroke-dashoffset` + `#ring-{day}-pct` text — shown in each day's hero header
4. **Sidebar day-complete checkmark** — `.day-complete` class added to nav item when day reaches 100%

**Checkpoint counts per day:**

| Day | Checkboxes |
|-----|-----------|
| Monday | 22 |
| Tuesday | 13 |
| Wednesday | 9 |
| Thursday | 6 |
| Friday | 9 |
| **Total** | **59** |

### 6.6 Notes & Questions System

- Every module has a `<div class="question-box">` with a textarea and "Save Note" button
- Clicking "Save Note" stores the text in `localStorage` (and syncs to Supabase) under key `q_q-{module-key}`
- A "✓ Saved!" confirmation flash appears for 2.5 seconds
- Notes are restored from storage on every page load
- Notes are included in the JSON backup export

**Question boxes by module:**

| Key | Module |
|-----|--------|
| q-mon-welcome | Monday — Welcome & Orientation |
| q-mon-roster | Monday — Client Roster |
| q-mon-email | Monday — Email & Communication |
| q-mon-tech | Monday — Tech Stack Setup |
| q-mon-docs | Monday — Document Sharing |
| q-tue-asana | Tuesday — Asana & Daily Workflow |
| q-tue-keeper | Tuesday — Keeper |
| q-tue-escalation | Tuesday — Escalation Guide |
| q-wed-qbo | Wednesday — QBO Bookkeeping |
| q-thu-payroll | Thursday — Payroll |
| q-fri-state | Friday — State Filings / Tax |
| q-fri-wrapup | Friday — Week 1 Wrap-Up Reflection |

### 6.7 Progress Backup (Export / Import)

- **Export:** Downloads a `.json` file named `KW_Patricia_Training_Progress_{date}.json` containing all checkbox states, notes, and timestamps
- **Import:** Reads a previously exported `.json` file and restores all checkbox and note state
- **Reset:** Confirms then wipes all `localStorage` data and unchecks all boxes

### 6.8 Cloud Sync (Supabase)

- On page load: reads from Supabase `progress` table, merges any server-only keys into local data
- On every save: upserts to `progress` table with `user_id = USER_ID` and `data = localData`
- Merge strategy: localStorage wins for any key that already exists locally; Supabase fills in missing keys only
- `supabaseLoaded` flag ensures writes never fire before the initial read completes

---

## 7. Data Model

### Supabase Table: `progress`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Auto-generated primary key |
| `user_id` | TEXT NOT NULL | Identifies the trainee (e.g., `patricia-montan`) |
| `data` | JSONB | Full progress state — checkbox booleans + note strings |
| `updated_at` | TIMESTAMPTZ | Auto-set to NOW() on creation |

### `data` JSONB Schema (example)

```json
{
  "mon-1": true,
  "mon-2": true,
  "mon-3": false,
  "q_q-mon-welcome": "This is my question about the welcome module...",
  "q_q-mon-roster": "Notes about clients from the roster call...",
  "__exported": "2026-04-14T18:30:00.000Z",
  "__trainee": "Yomaira Patricia Montan"
}
```

Checkbox keys: `{day}-{number}` (e.g., `mon-1`, `tue-7`, `fri-9`)
Note keys: `q_q-{module-key}` (e.g., `q_q-mon-welcome`, `q_q-tue-asana`)

### Row Level Security (RLS)

RLS is enabled on the `progress` table. Current policies allow any client with the anon key to SELECT, INSERT, and UPDATE — meaning all operations are open to the frontend. This is appropriate for a single-trainee internal tool; tighter policies (e.g., per-user auth) should be considered if the tool expands to multiple trainees in production.

---

## 8. Design System

All styles are defined as CSS custom properties at the `:root` level within `src/index.html`.

| Token | Value | Usage |
|-------|-------|-------|
| `--navy` | `#1B2A4A` | Primary brand color — top bar, sidebar, headers |
| `--teal` | `#2E86AB` | Accent color — links, active states, progress bars |
| `--teal-lt` | `#D5E8F0` | Light teal — info alert backgrounds |
| `--gold` | `#F4A261` | Highlight color — today marker, progress rings, CTA buttons |
| `--gold-lt` | `#FFF3CD` | Light gold — question box backgrounds |
| `--green` | `#28A745` | Success — completed states |
| `--green-lt` | `#D4EDDA` | Success backgrounds |
| `--red` | `#DC3545` | Warning/error — escalation items |
| `--red-lt` | `#F8D7DA` | Warning backgrounds |
| `--gray` | `#F5F5F5` | Card backgrounds |
| `--dark` | `#2C2C2C` | Body text |
| `--mid` | `#666666` | Secondary text |
| `--shadow` | `0 2px 12px rgba(0,0,0,0.10)` | Card shadows |
| `--radius` | `10px` | Border radius |

**Typography:** `'Segoe UI', Arial, sans-serif`
**Base background:** `#EEF3F8`

---

## 9. Infrastructure & Deployment

### 9.1 Hosting

- **Platform:** Vercel
- **Configuration:** `vercel.json` sets `outputDirectory: "src"` and rewrites all routes to `index.html` (SPA routing support)
- **Build:** No build command — static files served directly
- **Deployment trigger:** Any push to the `main` branch on GitHub auto-deploys to Vercel

### 9.2 GitHub Repository

- **URL:** https://github.com/ola-fakomi/kelliwork-trainingpage
- **Branch:** `main` (single branch, no branching strategy currently)
- **Default branch:** main

### 9.3 External Dependencies (CDN)

| Library | Version | Source | Purpose |
|---------|---------|--------|---------|
| `@supabase/supabase-js` | `@2` (latest v2) | jsDelivr CDN | Database client for progress sync |

No npm packages, no build pipeline, no Node.js required.

### 9.4 Deployment Options

As documented in README.md, the site can alternatively be deployed via:
- Netlify (drag-and-drop `src/index.html`)
- GitHub Pages
- KelliWorks cPanel (upload `src/index.html` to `public_html/`)

---

## 10. Configuration

### `config/supabase-config.js`

```javascript
const SUPABASE_URL = 'https://vrxytzeaeyzmrftuazak.supabase.co';
const SUPABASE_ANON_KEY = 'sb_publishable_...';
const USER_ID = 'patricia-montan';
```

- `SUPABASE_URL` and `SUPABASE_ANON_KEY` are the project credentials from the Supabase dashboard
- `USER_ID` uniquely identifies Patricia's progress row in the database
- To onboard a new trainee: change `USER_ID` to a new slug (e.g., `jane-smith`) and update the trainee's name throughout `index.html`
- All three variables are declared as `const` (not attached to `window`), so they must be referenced by name, not as `window.USER_ID`

---

## 11. Content Update Guide

### Adding a Loom Video

1. Get the Loom share URL (e.g., `https://www.loom.com/share/abc123`)
2. In `src/index.html`, find the relevant `loom-slot` block (by its `id`)
3. Change `class="loom-slot"` → `class="loom-slot has-video"`
4. Change `<a class="loom-play pending" href="#">` → `<a class="loom-play" href="[LOOM_URL]">`
5. Change `<div class="loom-status pending">📹 Video coming soon</div>` → `<div class="loom-status">▶ Click to watch</div>`
6. Commit and push — Vercel auto-deploys

### Adding a New Training Day or Module

1. Add a new `<div class="day-panel" id="panel-{day}">` block in `src/index.html`
2. Add the day's checkbox IDs to the `DAYS` object in the `<script>` block
3. Add a nav item in the sidebar
4. Commit and push

### Onboarding a New Trainee

1. Create a new HTML file (e.g., copy `index.html` and rename it)
2. Update the name, role, and dates throughout the file
3. Update `USER_ID` in `config/supabase-config.js` to a new slug
4. Update `__trainee` in `exportProgress()` function
5. Deploy

---

## 12. Known Limitations & Future Considerations

| Item | Description | Priority |
|------|-------------|----------|
| Single trainee per config | `USER_ID` is hardcoded; switching trainees requires editing the config file | Medium |
| No authentication | Any user with the URL can view Patricia's training site | Low (internal tool) |
| No admin view | Kelli cannot log in to view Patricia's progress from a separate dashboard | Medium |
| RLS is fully open | Supabase policies allow all operations with the anon key | Low (single trainee, internal) |
| `updated_at` not auto-updated | The `updated_at` column is set on insert but not updated on `upsert` | Low |
| No notification system | Kelli has no automated alert when Patricia saves a note | Medium |
| Supabase key format | The site uses the new `sb_publishable_` key format; verify SDK v2 compatibility | Low |
| Mobile layout | Sidebar is hidden on mobile; content layout not fully optimized for small screens | Low |
| Week 2 content | All Week 2 modules are placeholders pending portal and video availability | High (upcoming) |
| No offline mode indicator | If Supabase sync fails, the user has no visual feedback | Low |
