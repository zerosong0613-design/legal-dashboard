# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**법무팀 사건 관리 시스템** (Legal Team Case Management System) — a single-file HTML SaaS dashboard for SK Chemical's legal department. The entire application lives in `index.html` (~5000 lines). `SETUP.md` documents SharePoint/Power Automate integration steps.

No build step, no bundler, no package.json. Open `index.html` directly in a browser to run.

## Architecture

### Single-file structure (`index.html`)
The file is organized top-to-bottom in three sections:
1. **`<style>`** (lines ~1–830): CSS with CSS custom properties in `:root`. Design system uses `--navy`/`--gold` color scheme, 4px spacing scale, 6-step typography scale.
2. **HTML body** (lines ~835–1343): Sidebar + main layout. Each page is a `<div class="screen" id="screen-{name}">` toggled via `showScreen()`.
3. **`<script>`** (lines ~1344–end): All JavaScript in one `<script>` block — no modules, no classes, all top-level functions.

### External dependencies (CDN only)
- **Chart.js 4.4.0** — report charts
- **PptxGenJS** — PowerPoint export

### Screen/Page structure
Five screens controlled by sidebar navigation:
- `screen-dashboard` — summary cards, D-day deadlines, category tabs, lawyer grid
- `screen-cases` — filterable case list with table/kanban/calendar views
- `screen-archive` — closed cases (status=done)
- `screen-report` — monthly/annual reports with charts, PDF/CSV/PPTX export
- `screen-cost` — legal cost management (routine costs + per-case costs)
- `screen-detail` — full-page case detail view (replaces content area, has its own step-bar and tabbed sections)

### Data layer
- **localStorage** is the sole data store. Key: `legal_v2` stores the cases array as JSON.
- Other localStorage keys: `legalUser` (auth), `legal_routine_costs`, `legal_favs`, `legal_notifs`, `legal_cost_types`, `legal_search_history`, `webhookUrl`, `sidebarMini`.
- `save()` persists the `cases` array. All mutations must call `save()` then `renderAll()`.
- Each case object has: `id`, `caseNum`, `caseName`, `category` (LIT/ADV/INT/REG/ETC), `status` (new/review/process/done/hold), `lawyer`, `date`, `deadlines[]`, `memos[]`, `files[]`, `parties[]`, `costs[]`, `tags[]`, `summary`, `confidential`, `allowedUsers`, plus timeline/closure fields.

### Authentication
Simple PIN-based auth stored in localStorage. Three roles: `lawyer` (매니저), `manager` (팀장), `executive` (임원). Access control via `canAccess(c)` — confidential cases restricted by `allowedUsers` list.

### Key rendering pattern
`renderAll()` is the main re-render entry point — it calls `renderSummary()`, `renderDday()`, `renderCatTabs()`, `renderCatPanel()`, `renderLawyers()`, `renderTable()`, `renderArchive()`, `renderPinnedSection()`. The detail page uses `renderDetailPage(c)` which calls individual `renderDp*()` functions.

### AI integration points
- **AI Summary** (`renderDpSummary`): Anthropic API call for case summarization
- **AI File Agent** (`agentAsk`): Document analysis agent with file upload
- **Global Chat** (`sendGlobalChat`): Floating chatbot panel
- API key expected in `CONFIG.ANTHROPIC_API_KEY` (or proxied via Vercel API route)

### SharePoint/Power Automate integration
Controlled by `CONFIG` object at script top. `OFFLINE_MODE: true` uses localStorage only. When `false`, reads/writes go to SharePoint Lists via REST API. Power Automate Flow URLs handle submissions (F-01), saves (F-06), and revision requests (F-07).

## UI conventions
- Categories: LIT (소송·중재, red), ADV (일반 자문, blue), INT (내부 자문, purple), REG (규제·행정, orange), ETC (기타, gray)
- Statuses: new (접수), review (검토중), process (처리중), done (완료), hold (보류)
- Case numbers follow pattern: `{YYYY}-{CAT}-{SEQ}` (e.g., `2026-LIT-001`)
- All user-facing text is Korean
- XSS prevention via `esc()` helper for all user content rendering
- Mobile responsive with bottom tab bar at 768px breakpoint
