# Changelog
## KelliWorks Training Hub — Patricia Montan Onboarding Site

All changes to this project are documented here in reverse chronological order.

---

## [1.5.0] — 2026-04-14

**Commit:** — *Add Slack DM notifications when Patricia saves a note*

### Feature

**Notes now delivered to Kelli's Slack DM instantly on save**

- Every time Patricia clicks "Save Note" on any module, a formatted Slack message is sent to Kelli via Incoming Webhook
- Message includes the module name, full note text, and timestamp
- Uses a no-backend `fetch()` POST — no server required, runs entirely in the browser
- Webhook URL stored in `src/config/supabase-config.js` as `SLACK_WEBHOOK_URL`
- Gracefully skipped if note is empty or webhook is not configured
- Does not block or delay the local save — fire-and-forget

### Files Changed
- `src/config/supabase-config.js` — 3 insertions (added `SLACK_WEBHOOK_URL`)
- `src/index.html` — 25 insertions, 0 deletions (modified `saveQuestion()`)

---

## [1.4.0] — 2026-04-14

**Commit:** — *Fix supabase-config.js path — move file inside src/ for Vercel*

### Bug Fix

**Critical production crash — entire JS dead on load**

- `config/supabase-config.js` was located outside the `src/` directory
- Vercel only serves files inside `src/` (per `outputDirectory: "src"` in `vercel.json`)
- The `../config/supabase-config.js` script tag returned a 404 HTML error page
- Browser tried to parse HTML as JavaScript, hit `<` at character 1, threw `Uncaught SyntaxError: Unexpected token '<'`
- This crashed the entire `<script>` block — all event listeners, progress bars, and saves were dead
- **Fix:** Moved file to `src/config/supabase-config.js`; updated `<script src>` from `../config/supabase-config.js` to `config/supabase-config.js`; removed old file via `git rm`

### Files Changed
- `src/config/supabase-config.js` — new location (moved from `config/supabase-config.js`)
- `src/index.html` — 1 insertion, 1 deletion (updated script src path)

---

## [1.3.0] — 2026-04-14

**Commit:** `0f3a6d4` — *Fix progress bars not updating on any page*

### Bug Fixes

**SVG progress rings never updated visually**
- The daily progress ring for each day is an SVG circle element with a `stroke-dashoffset` attribute that animates from full (invisible) to zero (full circle) as checkboxes are completed
- The code was using `ring.style.strokeDashoffset = value` (CSS property assignment) to update the ring
- For SVG elements, CSS property assignment via `.style` is unreliable across browsers — the HTML presentation attribute `stroke-dashoffset="188.5"` was not being overridden correctly
- **Fix:** Changed to `ring.setAttribute('stroke-dashoffset', String(value))` which is the universally correct method for updating SVG presentation attributes

**Duplicate event listeners on every checkbox**
- The boot sequence called `initCheckboxes()` twice: once on initial page load, and again inside the `.then()` callback after Supabase sync completed
- Each call to `initCheckboxes()` attached a new `change` event listener to every checkbox — after two calls, each of the 59 checkboxes had two listeners firing on every click, causing double saves and double Supabase upserts
- **Fix:** The `.then()` callback no longer calls `initCheckboxes()`. Instead it directly restores checkbox checked states and textarea values from the already-merged `localData`, then calls `updateProgress()`. Event listeners are now attached exactly once.

**Unhandled promise rejection**
- If `syncFromSupabase()` rejected (e.g., network error, Supabase auth failure), the `.then()` would silently fail with no error handling
- **Fix:** Added `.catch(() => {})` to the Supabase sync promise chain to prevent uncaught promise rejection errors in the browser console

### Files Changed
- `src/index.html` — 16 insertions, 2 deletions

---

## [1.2.0] — 2026-04-14

**Commit:** `0f64174` — *Fix Supabase sync — replace window.USER_ID with USER_ID*

### Bug Fixes

**Notes not saving to Supabase — all progress lost on new browser/device session**

This was the root cause of Patricia being unable to persistently save her notes.

- `config/supabase-config.js` declares `const USER_ID = 'patricia-montan'`
- In JavaScript, variables declared with `const` (or `let`) at the top level of a `<script>` tag are in the global scope but are **NOT** properties of the `window` object — only `var` declarations attach to `window`
- Both Supabase operations in `index.html` referenced `window.USER_ID`, which evaluated to `undefined`
- The Supabase SELECT query ran as `.eq('user_id', undefined)` — returning no data, so no previously saved progress was ever loaded from the server
- The Supabase UPSERT ran as `{ user_id: undefined, data: localData }` — data was being upserted under a `null` user_id, not under `patricia-montan`
- **Symptom:** Clicking "Save Note" showed the "✓ Saved!" confirmation (localStorage was working correctly), but on any new browser session, fresh device, or after clearing the browser cache, all notes and progress were permanently lost
- **Fix:** Both `window.USER_ID` references replaced with `USER_ID` — two changes in `src/index.html`

### Impact
- All of Patricia's data previously written to Supabase under `null` user_id will not automatically migrate; she should re-enter notes from her current session
- Going forward, all saves correctly write to `user_id = 'patricia-montan'`

### Files Changed
- `src/index.html` — 2 insertions, 2 deletions

---

## [1.1.0] — 2026-04-14

**Commit:** `572cefc` — *Update Tuesday training section with Asana, Keeper, and Escalation modules*

### Content Update

**Full Tuesday panel replacement**

The original Tuesday panel (committed in the initial build) contained placeholder outreach/communication content that did not reflect the actual training curriculum Kelli provided. The entire panel was replaced with the correct Week 1 Tuesday content.

**Before (removed):**
- Module 2A: Outreach & Client Check-Ins — 3 placeholder Loom video slots, 3 checkboxes
- Module 2B: Escalation Guide — escalation grid, 3 checkboxes
- Total: 6 checkboxes

**After (new content):**
- Module 2A: Asana & Daily Workflow — 4 live Loom videos, 6 checkboxes, notes box
- Module 2B: Keeper: Client Financial Delivery — 2 live Loom videos, 4 checkboxes, Keeper site link callout, notes box
- Module 2C: Escalation Guide — (renumbered from 2B) expanded description, escalation grid, 3 checkboxes, notes box
- Total: 13 checkboxes

**6 Loom videos activated for Tuesday:**

| Slot ID | Video Title | Loom URL |
|---------|-------------|----------|
| loom-tue-asana-nav | Asana Project Navigation Guide | `/share/3f3d44e2da2448aba96b82802c70c335` |
| loom-tue-workflow | Slack, Asana & E-Mail Workflow | `/share/52e9aa32928a4f25838a0b17e046e672` |
| loom-tue-tasks | Asana — Task KW Emails | `/share/bcc48f8f430f4713afb6edc9b14d7db0` |
| loom-tue-asana-mgmt | How to Manage Your Workflow in Asana | `/share/a9f5a88380124da288788842a1ba98c9` |
| loom-tue-keeper | Reviewing Financials and Keeper (Cash Basis) | `/share/a2067a7d6159454896e5e2abe25a0602` |
| loom-tue-keeper2 | Understanding New Features in Keeper | `/share/32994f05805f4019a4eddcdcc484101a` |

**DAYS object updated:**
- `tue` checkbox IDs updated from `['tue-1'...'tue-6']` (6 items) to `['tue-1'...'tue-13']` (13 items)
- Progress calculation for Tuesday now correctly tracks all 13 checkboxes

**Source file used:** `html_Updates/tuesday_section_snippet.html` — a reference snippet containing the full panel HTML and script call block. Since `setLoomVideo()` JS was removed in a previous commit (fc9db08), Loom URLs were embedded directly as `href` attributes rather than using the script calls listed in the snippet.

### Files Changed
- `src/index.html` — 135 insertions, 33 deletions

---

## [1.0.2] — 2026-04-13

**Commit:** `fc9db08` — *Simplify Loom links - direct hrefs instead of JavaScript*

### Refactor

**Removed `setLoomVideo()` JavaScript function in favour of direct href attributes**

The original implementation used a `setLoomVideo(id, url, title)` JavaScript function that would:
1. Find the loom slot element by ID
2. Set the `href` on the play button anchor
3. Update the status text
4. Add `has-video` class to the container

This pattern required calling `setLoomVideo(...)` for each video in a separate script block, and required matching HTML IDs to be maintained in sync with JS calls.

**Change:** All active Loom video links are now set directly as `href` attributes on the `<a class="loom-play">` elements in the HTML. The `has-video` class and status text are set directly in the markup. This eliminates the JavaScript indirection and makes videos easier to add and maintain — the HTML is the single source of truth for each video's URL.

Pending videos continue to use `href="#"` with `class="loom-play pending"` and `📹 Video coming soon` status text.

### Files Changed
- `src/index.html` — 36 insertions, 102 deletions (net removal of ~66 lines of JS boilerplate)

---

## [1.0.1] — 2026-04-13

**Commit:** `5e08acf` — *Add vercel.json for SPA routing*

### Infrastructure

**Added Vercel deployment configuration**

Without this file, Vercel would attempt to serve files from the project root and direct URL access to any path other than `/` would return a 404.

`vercel.json` configures:
- `"buildCommand": null` — no build step required (static site)
- `"outputDirectory": "src"` — serves files from the `src/` directory
- `"rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]` — rewrites all routes to `index.html` to support SPA-style navigation where `showDay()` switches panels without changing the URL

### Files Added
- `vercel.json` — 5 lines

---

## [1.0.0] — 2026-04-13

**Commit:** `38587da` — *Initial commit: Training site with Supabase sync*

### Initial Build — ~2,000 lines across 3 files

**`src/index.html`** — Full training application

*UI & Layout:*
- Sticky top bar with KelliWorks branding, trainee identity, and overall progress bar
- Left sidebar with day navigation, date badges, and completion checkmarks
- Save bar with Export/Import/Reset progress controls
- Responsive layout (sidebar hidden on mobile)

*Design system:*
- Full CSS custom property design system (`--navy`, `--teal`, `--gold`, `--green`, `--red`, etc.)
- Card-based module layout with colour-coded headers
- SVG progress rings per day (animated stroke-dashoffset)
- Progress bars on week overview cards
- Alert components (info, success, warning) in teal/green/gold
- Escalation grid (two-column green/red layout)
- Loom video slot components
- Checklist items with strikethrough-on-check styling
- Question box components (gold background)

*Training content — Week 1:*
- **Overview panel** — Week overview grid, Meet the Team section (Kelli, Tim, Rangan, Sagar), 5 Core Commitments
- **Monday panel** — 5 modules (1A–1E), 22 checkboxes, 5 notes boxes
- **Tuesday panel** — placeholder outreach/escalation content (later replaced in v1.1.0)
- **Wednesday panel** — QBO overview, invoicing, reconciliation; 9 checkboxes
- **Thursday panel** — Payroll processing; 6 checkboxes
- **Friday panel** — State filings, tax client management, Week 1 wrap-up; 9 checkboxes
- **Week 2 panel** — Locked banner, 4 preview checklist items (disabled)

*JavaScript logic:*
- Supabase client initialisation
- `syncFromSupabase()` — reads progress from database, merges with localStorage
- `syncToSupabase()` — upserts progress to database (guarded by `supabaseLoaded` flag)
- `DAYS` object — maps each day name to its array of checkbox IDs
- `load()` / `save()` — localStorage read/write wrappers
- `initCheckboxes()` — restores checkbox states from storage, attaches change listeners
- `updateProgress()` — recalculates and updates all 4 types of progress indicators
- `showDay(day)` — SPA-style panel switching with nav highlight
- `saveQuestion(key)` — saves note textarea to storage, shows confirmation flash
- `exportProgress()` — downloads JSON backup file
- `importProgress(event)` — restores state from uploaded JSON file
- `resetProgress()` — clears all stored data with confirmation dialog
- `showStatus(msg)` — shows timed status message in save bar

**`config/supabase-config.js`** — Supabase project URL, anon key, and `USER_ID = 'patricia-montan'`

**`README.md`** — Full setup guide including:
- Supabase project configuration steps
- SQL to create and configure the `progress` table with RLS
- 4 deployment options (Netlify, Vercel, GitHub Pages, cPanel)
- Project structure overview
- How progress sync works
- Multi-trainee management instructions
- Troubleshooting guide

### Files Added
- `src/index.html` — 1,887 lines
- `config/supabase-config.js` — 9 lines
- `README.md` — 104 lines

---

## Version Summary

| Version | Date | Commits | Type | Summary |
|---------|------|---------|------|---------|
| 1.0.0 | 2026-04-13 | `38587da` | Feature | Initial full build |
| 1.0.1 | 2026-04-13 | `5e08acf` | Infra | Vercel deployment config |
| 1.0.2 | 2026-04-13 | `fc9db08` | Refactor | Simplified Loom video links |
| 1.1.0 | 2026-04-14 | `572cefc` | Content | Tuesday panel rebuilt with correct curriculum |
| 1.2.0 | 2026-04-14 | `0f64174` | Bug Fix | Supabase sync broken (`window.USER_ID` → `USER_ID`) |
| 1.3.0 | 2026-04-14 | `0f3a6d4` | Bug Fix | Progress bars broken (SVG setAttribute + duplicate listeners) |
| 1.4.0 | 2026-04-14 | — | Bug Fix | Critical crash — supabase-config.js path outside Vercel served dir |
| 1.5.0 | 2026-04-14 | — | Feature | Slack DM notifications when Patricia saves a note |
