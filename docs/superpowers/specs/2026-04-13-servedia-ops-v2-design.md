# Servedia Ops v2 — Full Platform Design

**Date:** 2026-04-13  
**Project:** Artzy22/servedia-ops  
**Owner:** Byron Vale, media buyer — Servedia (medspa marketing agency)  
**Stack:** Single `index.html`, dark theme, `#7c3aed` purple accent, pure HTML/CSS/JS + GitHub Actions

---

## Overview

A complete ops dashboard for managing 33+ Meta ad accounts, media buying workflows, tech tickets, and client relationships. The v2 redesign migrates from a horizontal tab bar to a **sidebar layout** with a **Command Center** home screen showing all live data at a glance.

---

## The One Rule — Never Breaks

The system detects, drafts, and proposes. **Byron clicks "Confirm & Execute" before anything is written to Meta API, ClickUp API, or Slack API.** Every action button shows a confirm modal. No exceptions.

---

## Integrations

| Source | What it provides | Sync method |
|--------|-----------------|-------------|
| Meta Ads API v19.0 | Spend, CPL, leads, billing status, policy flags, ad creative performance | GitHub Actions every 30 min → `data/meta.json` |
| ClickUp API | Client management tasks, video tickets, tech tickets, media tasks | Live fetch + GitHub Actions 30-min backup |
| Slack Web API | All channels last 24h — flagged messages, urgent keyword detection | GitHub Actions every 30 min → `data/slack.json` |
| GitHub Actions | All background automation — no server required | Scheduled YAML workflows |

**GoHighLevel: not included.**

---

## Layout — Sidebar (Option B)

Persistent left sidebar with grouped sections. The sidebar replaces the horizontal tab bar entirely.

```
┌─────────────────────────────────────────────┐
│  S  Servedia Ops          Last sync 3m  ⚙️  │
├──────────┬──────────────────────────────────┤
│          │                                  │
│  ⚡ Cmd  │   COMMAND CENTER / SECTION VIEW  │
│          │                                  │
│ ─ Media  │                                  │
│  📊 Meta │                                  │
│  🗂️ Maint│                                  │
│  🏆 Intel│                                  │
│          │                                  │
│ ─ Client │                                  │
│  🤝 CRM  │                                  │
│  🚀 Onbd │                                  │
│  🆕 Watch│                                  │
│          │                                  │
│ ─ Work   │                                  │
│  🎬 Video│                                  │
│  ⚙️ Tech │                                  │
│  💬 Slack│                                  │
│  ☑️ Tasks│                                  │
│          │                                  │
│  ⚙️ Sett │                                  │
└──────────┴──────────────────────────────────┘
```

Sidebar badges show live counts. Active item has purple left border + dark purple background.

---

## Sections

### ⚡ Command Center (Home)

Default view on load. Shows all live data from every section at once.

**Layout:**
1. **Alert chips row** — clickable chips for: Billing Paused, Policy Violations, Creative Fatigue, Urgent Slack. Each chip navigates to the relevant section.
2. **Meta Ads KPI row (5 stats)** — Total Spend 7d, Total Leads 7d, Avg CPL, Red CPL Accounts (>$20), Zero-Lead Accounts
3. **Active Alerts panel (3 columns)** — Billing Paused · Policy Violations · Creative Fatigue — client names listed in each
4. **Work Queue snapshot** — Video Tickets (count + top 2) · Tech Tickets (count + top 2)
5. **Client Health snapshot** — Worst and best health scores from the 33 accounts, renewal count

---

### 📊 Meta Ads

33 ad account cards. Each card shows: account name, status badge, 7d/30d metric tabs, spend/leads/CPL/CTR.

**Badges on cards:**
- 🔴 `BILLING PAUSED` — red pulsing, from `meta.json.payment_issue`
- ⚠️ `X POLICY ISSUES` — yellow, from `meta.json.violations`
- 🎨 `CREATIVE FATIGUE` — amber, from `meta.json.fatigued_ads`
- CPL tier badge — Great/Good/OK/Watch/Cut per configurable thresholds

**Sort options:** Needs Attention Score · Worst CPL · Best CPL · A-Z

**Needs Attention Score formula:**
- CPL > $28 = +100
- CPL $20–28 = +60
- CPL $15–20 = +30
- No changes 2+ days = +40
- Payment issue = +80
- Policy violation = +50
- 0 leads 3 days = +50
- Lead velocity < 1/day = +25

**AI Suggestions per account:** Scale 15% (CPL < $10), Watch (CPL $20–28), Cut (CPL > $28), Pause (0 leads), budget reallocation best→worst.

---

### 🗂️ Daily Maintenance

Per-client cards for Byron's daily review workflow. Each card: client name, CPL badge, CPB badge, leads, AI suggestions, ClickUp link, Drive link, notes, flag/complete toggles.

**Filters:** All · Red CPL · Yellow · Green · Flagged · Pending Approval  
**Sort:** Needs Attention · A-Z · CPL

---

### 🏆 Creative Intel

Cross-portfolio intelligence. Finds ads with CPL < account_avg × 0.5, spend > $100, leads ≥ 5. Surfaces winning creative formulas to clone for similar-niche clients. Read-only — no action buttons, Slack alert only.

Morning briefing panel: "Creative Wins to Clone" — top performers, which other clients can use them.

---

### 🤝 CRM

Client health and relationship management powered by ClickUp custom fields + Meta data.

**Client Health Score (0–100):**
- Start: 100
- CPL > $28: −40 | CPL $20–28: −25 | CPL $15–20: −15 | CPL $10–15: −5
- Leads < 1/day: −20 | Leads < 0.5/day: −30
- Open tasks > 5: −10 | Open tasks > 10: −20
- Video ticket "Needs Approval" > 5 days: −10
- Payment issue: −40 | Policy violation: −15 | Creative fatigue: −10
- No changes 3+ days with bad CPL: −15

**Score tiers:** 80–100 green HEALTHY · 60–79 yellow OK · 40–59 orange AT RISK · 0–39 red CRITICAL

**Renewal Pipeline:** Clients with renewal date within 30 days. Shows days until renewal, happiness rating (Happy/Neutral/Unhappy from ClickUp), churn risk flag.

**Client Profile Drawer:** Full detail — billing type, niche, funnel URL, Drive folder, special rules, contact, refund flag, onboarding duration.

---

### 🚀 Onboarding

New client creation flow — "New Client" button opens multi-step form.

**Form fields:** Client name, Meta account ID, niche, daily budget, target CPL, treatment/offer, funnel URL, Drive folder (default: `1Lu4Gi3zZB9eLojJwwQNT9Zv-yMbImwYf`), special rules, assignee.

**Confirm modal:** Preview 8 checklist ClickUp tasks before anything is created.

**On Byron's confirm:**
1. POST 8 tasks to ClickUp client management list
2. Update `client-map.json` with new client entry
3. Slack alert to `#media-ads`

**Second modal:** First creative request to Saquib (video ticket, 3 business days deadline).

---

### 🆕 Watchdog

30-day monitoring for newly onboarded clients.

**Alerts (every 6 hrs via GitHub Action):**
- 0 leads after day 5 → Slack alert
- CPL > $20 in week 1 → Slack alert
- No active ads → Slack alert

**Dashboard:** "NEW — Day X" badge on client card. Turns orange on day 14 with bad CPL. Turns red on day 21 with no leads.

---

### 🎬 Video Tickets

ClickUp List `901512331541`. Shows all video editing tickets assigned to Saquib.

Columns: client name, ticket name, status, priority (urgent/high/normal/low), due date (red if overdue), ClickUp link.

Filters: All · Needs Approval · In Progress · Urgent

---

### ⚙️ Tech Tickets

ClickUp-sourced. Mixed queue: client website/funnel fixes + internal tech ops.

**Ticket types:**
- Client funnel & website — broken forms, landing page issues, link fixes
- Internal tech ops — dashboard bugs, workflow automation, tool setup

**Assignee routing:** Tagged by type, routed to Byron / Anca / Saquib.

**Columns:** Type badge, client/project, description, assignee, priority, due date, status.

---

### 💬 Slack

All channels last 24h. Flagged messages and questions pinned to top ("Needs Response" panel).

**Urgency colors:** Red = urgent keyword detected · Yellow = message ending with `?` · Gray = general

**Urgent keywords:** urgent / revision / fix / redo / ASAP / please fix / reblast / re-blast / remake / wrong / incorrect / not right / change this

**Create Task modal (Byron confirms):** Detect client from message, pre-fill ClickUp task. Assignee routing: `#video-editing` → Saquib, `#media-ads` → Byron/Anca.

**Tab badge:** Count of flagged + question messages.

---

### ☑️ Tasks

ClickUp List `901515083916` (media tickets/main tasks). Byron + Anca's task queue.

Filters: All · Urgent · Overdue · Byron · Anca · Focus Queue (drag-to-prioritize)

---

### ⚙️ Settings

CPL threshold configuration (Great/Good/OK/Watch — Cut is auto above Watch).

ClickUp API token input, client management list ID, live mode toggle.

---

## GitHub Actions — Automation Engine

| Workflow | Schedule | What it does |
|----------|----------|--------------|
| `sync-meta.yml` | Every 30 min | Meta KPIs for all 33 accounts |
| `sync-clickup.yml` | Every 30 min | Main task list |
| `sync-slack.yml` | Every 30 min | All channels + keyword flagging |
| `sync-maintenance.yml` | Every 30 min | Maintenance list |
| `sync-video-editing.yml` | Every 15 min | Video ticket list |
| `sync-drive.yml` | Every 6 hrs | Drive folder index |
| `build-client-map.yml` | Weekly Sun 6am + dispatch | Fuzzy-match Meta → ClickUp |
| `check-payment-failures.yml` | Daily 8am UTC | Meta billing status scan |
| `check-policy-violations.yml` | Daily 7am UTC | Disapproved/flagged ads scan |
| `check-creative-fatigue.yml` | Daily 9am UTC | CTR trend analysis |
| `cross-portfolio-intel.yml` | Daily 10am UTC | Cross-account winning creatives |
| `slack-task-detector.yml` | Every 30 min | Keyword scan → flagged_messages |
| `watchdog.yml` | Every 6 hrs | New client 30-day monitoring |

---

## Data Files

| File | Updated by | Contents |
|------|-----------|----------|
| `data/meta.json` | sync-meta | Ad account KPIs + payment/violation/fatigue flags |
| `data/clickup.json` | sync-clickup | Main task list |
| `data/slack.json` | sync-slack | Messages + flagged_messages array |
| `data/maintenance.json` | sync-maintenance | Maintenance list tasks |
| `data/video-editing.json` | sync-video-editing | Video ticket tasks |
| `data/drive-folders.json` | sync-drive | Drive folder structure |
| `data/client-map.json` | build-client-map | Meta account ID → ClickUp task ID mappings |

---

## CPL Tier Colors

| Tier | Range | Color | Label |
|------|-------|-------|-------|
| Great | < $10 | `#22c55e` | Great |
| Good | $10–$12 | `#86efac` | Good |
| OK | $12–$15 | `#fbbf24` | OK |
| Watch | $15–$20 | `#f97316` | Watch |
| Bad | $20–$28 | `#ef4444` | Bad — Act |
| Cut | > $28 | `#991b1b` white text | CUT |

All thresholds are configurable in Settings and apply everywhere CPL is shown.

---

## Build Order

1. **Layout migration** — sidebar replaces tab bar, Command Center home (index.html refactor)
2. **F1** — `build-client-map.yml` — unlocks F2–F13
3. **F2** — `check-payment-failures.yml` + billing badge UI
4. **F3** — `check-policy-violations.yml` + policy badge UI
5. **F7** — `check-creative-fatigue.yml` + fatigue badge UI
6. **F11** — Client health score (CRM section)
7. **F8** — `cross-portfolio-intel.yml` + Creative Intel section
8. **F9** — `slack-task-detector.yml` (GitHub Action side)
9. **CRM section** — full client profiles, renewal pipeline
10. **F12** — New client onboarding flow
11. **Tech Tickets section** — new ClickUp list + routing
12. **Watchdog section** — `watchdog.yml` + "NEW — Day X" UI
13. **Command Center** — wire all live data into home screen

---

## What Does NOT Change

- Byron must confirm all writes — no exceptions
- Pure HTML/CSS/JS — no React, no build step, deploys to GitHub Pages
- Dark theme `#0d0d0d` background, `#7c3aed` purple accent
- CPL tiers stored in localStorage, apply everywhere
- All data served from `raw.githubusercontent.com/Artzy22/servedia-ops/main/data/`
