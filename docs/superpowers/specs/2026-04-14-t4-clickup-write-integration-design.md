# T4 ClickUp Write Integration — Design Spec
**Date:** 2026-04-14
**Status:** Approved
**Author:** Terminal 4 / Byron Vale

---

## Overview

Wire the Terminal 4 Ops Agent audit action queue into the Command Center so flagged fire items can be executed directly from the dashboard. Uses the existing ClickUp API token (already stored in localStorage) and the existing `CU_API` constant — no new secrets, no new files.

---

## Architecture

All changes are contained in `index.html`. No new files are created.

### New additions

**`T4_ACTIONS` array** (static, defined at module level)
Defines all auditable actions. Each entry describes one executable operation. Populated from T4 audit output — updated manually as the portfolio changes.

**`t4Executed` Map** (session-only)
Tracks per-action state: `'idle' | 'confirming' | 'executing' | 'done' | 'error'`.
Resets on page reload. Not persisted to localStorage or any external store.

**`executeT4Action(id)`** function
Dispatches to the correct API call based on `action.type`. Manages state transitions in `t4Executed` and re-renders the affected card.

**`renderCommandCenter()` modified**
Each fire card receives an injected Execute button block. Button visibility is gated on `cfg.token` — renders a muted hint if no token is configured.

---

## Action Schema

```js
{
  id:          'fire-1',              // unique string, stable across renders
  label:       'Video queue — 29 tasks blocked',
  severity:    'critical',           // 'critical' | 'high' | 'medium'
  type:        'bulk_comment',       // 'comment' | 'bulk_comment' | 'create_task'
  taskIds:     ['86c9...', ...],     // bulk_comment only — array of ClickUp task IDs
  taskId:      '86c9...',            // comment only — single task ID
  listId:      '901515083916',       // create_task only — target list ID
  confirmText: 'Post comments to 29 video tasks?',
  commentText: '[T4 Audit 2026-04-14] ...',   // comment + bulk_comment
  payload:     { name, description, priority, assignees }  // create_task only
}
```

### Action types defined for April 14 audit

| # | ID | Type | Target |
|---|----|------|--------|
| 1 | fire-1 | bulk_comment | 29 video editing task IDs |
| 2 | fire-2 | comment | Thorpe Wellness onboarding task |
| 3 | fire-3 | create_task | Onboarding list — Ideal Weightloss audit |
| 4 | fire-4 | comment | Delray Laser onboarding task |
| 5 | fire-5 | comment | About Face & Body onboarding task |
| 6 | fire-6 | create_task | Onboarding list — Body Slim WLC media review |
| 7 | fire-7 | create_task | Onboarding list — Servedia B2B CPL audit |
| 8 | fire-8 | comment | 4 orphaned new sign-up tasks (bulk_comment) |
| 9 | fire-9 | create_task | Onboarding list — MAC OKC 24hr monitor |
| 10 | fire-10 | bulk_comment | 5 dark account status check tasks |

---

## Inline Confirm UX

The Execute button lives inside each Command Center fire card. State transitions:

```
idle        → "Execute" button (.fbtn)
confirming  → "[confirmText] · [✓ Confirm] [✗ Cancel]"  (replaces button in-place)
executing   → "⏳ Running…"  (disabled)
done        → "✅ Done"  · card fades to 40% opacity, pointer-events: none
error       → "❌ Failed: [message]"  · reverts to Execute after 4s
```

For `bulk_comment`, done state shows `"✅ 29/29"` on full success or `"⚠️ 26/29 — 3 failed"` on partial failure.

**Token gate:** If `cfg.token` is empty, the Execute button is replaced with a muted inline hint: `"Set token in ⚙ Config to enable"`.

---

## API Calls

All calls use `Authorization: cfg.token` and `Content-Type: application/json`.

### comment
```
POST https://api.clickup.com/api/v2/task/{taskId}/comment
Body: { comment_text: "[T4 Audit 2026-04-14] {text}", notify_all: false }
```

### bulk_comment
```js
Promise.allSettled(taskIds.map(id =>
  fetch(`${CU_API}/task/${id}/comment`, { method: 'POST', headers, body })
))
```
Returns count of resolved vs rejected. Partial success is non-fatal — surfaced in card state.

### create_task
```
POST https://api.clickup.com/api/v2/list/901515083916/task
Body: { name, description, priority, assignees }
```
List ID `901515083916` is the existing onboarding list — already used in `sync-clickup.yml`.

### Error handling
Non-2xx responses parse `response.json()` and surface `err.err || "HTTP {status}"` — same pattern as existing `postComment()`.

---

## Task ID Resolution

Task IDs for comment actions are resolved from `cuData` (the ClickUp JSON already loaded in memory at render time) by matching `task.name` to the client name in the action definition. No extra fetch needed.

For the video editing bulk comment, task IDs come from `vidData` filtered to `status === 'needs approval'`.

---

## Constraints

- **No auto-execution.** The Execute button always requires explicit user confirmation.
- **Session-only state.** `t4Executed` is not persisted. Refreshing the page resets all card states.
- **No backend required.** All writes go directly from the browser to the ClickUp API using the token in localStorage.
- **Token required.** If `cfg.token` is not set, no write operations are possible — the UI communicates this clearly per card.
- **Byron confirms all writes.** No action fires without the Confirm click.

---

## Out of Scope

- Meta Ads campaign pause (requires Meta API write scope — separate integration)
- Slack notifications (no Slack webhook wired)
- Persisting action history across sessions
- Undo / rollback of posted comments
