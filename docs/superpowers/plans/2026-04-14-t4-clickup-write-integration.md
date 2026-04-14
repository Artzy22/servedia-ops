# T4 ClickUp Write Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Wire the Terminal 4 audit action queue into the Command Center so each fire item has an inline Execute button that posts comments or creates tasks in ClickUp directly from the dashboard.

**Architecture:** All changes are in `index.html` only — no new files. Adds T4 CSS, a new `#ccT4Section` HTML block in `panel-command`, a static `T4_ACTIONS` array with all 10 audit actions, a `t4Executed` session Map for state tracking, and four JS functions (`renderT4Actions`, `t4StartConfirm`, `t4Cancel`, `t4Execute`). `renderCommandCenter()` is modified to call `renderT4Actions()` at the end. No backend required — all ClickUp writes go direct from browser using the existing `cfg.token` from localStorage.

**Tech Stack:** Vanilla JS, ClickUp REST API v2, existing `CU_API` constant and `getCfg()` pattern already in `index.html`.

**Note on testing:** No test framework exists in this project. Each task includes a manual browser verification step instead.

---

### Task 1: Add T4 CSS

**Files:**
- Modify: `index.html` — insert after `.bfire-action:hover{color:#c4b5fd}` (around line 297)

- [ ] **Step 1: Insert T4 CSS block**

Find this exact line in `index.html`:
```css
.bfire-action:hover{color:#c4b5fd}
```

Insert immediately after it:
```css

/* ══════════ T4 ACTION QUEUE ══════════ */
.t4-card{background:#0d0d1a;border:1px solid #1e1e33;border-radius:10px;padding:14px;transition:opacity .3s}
.t4-card.t4-done{opacity:.4;pointer-events:none}
.t4-sev-critical{border-left:3px solid #ef4444}
.t4-sev-high{border-left:3px solid #fb923c}
.t4-sev-medium{border-left:3px solid #fbbf24}
.t4-label{font-size:13px;font-weight:700;color:#e0e0e0;margin-bottom:10px}
.t4-execute-wrap{display:flex;align-items:center;gap:8px;flex-wrap:wrap}
.t4-btn{background:#1a1730;border:1px solid #7c3aed;color:#a78bfa;padding:5px 14px;border-radius:7px;cursor:pointer;font-size:12px;font-weight:600;transition:all .15s}
.t4-btn:hover{background:#2a1f4a;color:#c4b5fd}
.t4-btn-confirm{background:#064e3b;border-color:#059669;color:#34d399}
.t4-btn-confirm:hover{background:#065f46}
.t4-btn-cancel{background:#1a1a1a;border-color:#333;color:#777}
.t4-btn-cancel:hover{border-color:#555;color:#aaa}
.t4-status{font-size:12px;font-weight:600}
.t4-status.executing{color:#a78bfa}
.t4-status.done{color:#34d399}
.t4-status.error{color:#f87171}
```

- [ ] **Step 2: Verify no syntax errors**

Open `index.html` in a browser (or run `python3 -m http.server 8080` in `D:/GitHub/servedia-ops` and visit `http://localhost:8080`). Dashboard should load with no console errors. The new CSS classes don't appear yet — that's expected.

---

### Task 2: Add T4 HTML section to panel-command

**Files:**
- Modify: `index.html` — insert before `<div id="ccAllClear"` inside `panel-command`

- [ ] **Step 1: Insert T4 section HTML**

Find this exact line in `index.html`:
```html
  <div id="ccAllClear" style="display:none;text-align:center;padding:40px">
```

Insert immediately before it:
```html
  <div id="ccT4Section" style="margin-bottom:20px;display:none">
    <div class="section-title" style="color:#f87171;margin-bottom:10px">⚡ T4 ACTION QUEUE</div>
    <div id="ccT4List" style="display:flex;flex-direction:column;gap:8px"></div>
  </div>
```

- [ ] **Step 2: Verify HTML structure**

Reload the dashboard. The Command Center should look identical — `#ccT4Section` is hidden by default (`display:none`). Open DevTools → Elements and confirm `#ccT4Section` and `#ccT4List` exist inside `#panel-command`.

---

### Task 3: Add T4_ACTIONS array and t4Executed Map

**Files:**
- Modify: `index.html` — insert after `const CU_API = 'https://api.clickup.com/api/v2';` (around line 954)

- [ ] **Step 1: Insert T4_ACTIONS and t4Executed**

Find this exact line:
```js
const CU_API = 'https://api.clickup.com/api/v2';
```

Insert immediately after it:
```js

// ══════════════════════════════════════════════════════════════
// T4 ACTION QUEUE — audit snapshot 2026-04-14
// ══════════════════════════════════════════════════════════════
const T4_ACTIONS = [
  {
    id: 'fire-1',
    label: 'Video queue — 29 tasks blocked on approval',
    severity: 'critical',
    type: 'bulk_comment',
    taskIds: ['86c955edb','86c94aygp','86c94buv2','86c94caw5','86c94fg9e','86c93k20n','86c91gfy4','86c91gnfk','86c8x1wyn','86c8vp44z','86c8tvv3q','86c8tw0ea','86c8trve9','86c94df4a','86c9060q3','86c8zgxmn','86c8zgxxc','86c8xzu7q','86c8vn4p1','86c8vp6p7','86c8trn2r','86c8pmwb3','86c8rxwa5','86c8my5gt','86c8vp9gm','86c8rbpp4','86c90tff2','86c8zgz4d','86c8pbacx'],
    confirmText: 'Post comment on all 29 video tasks flagging the approval bottleneck?',
    commentText: '[T4 Audit 2026-04-14] Video queue blocked — this task is 1 of 29 stuck at needs-approval. Escalating approval bottleneck to team lead for immediate unblock.'
  },
  {
    id: 'fire-2',
    label: 'Thorpe Wellness 2.0 — Meta 403 error + onboarding overdue',
    severity: 'critical',
    type: 'comment',
    taskId: '86c8uee2j',
    confirmText: 'Post comment on Thorpe Wellness onboarding task?',
    commentText: '[T4 Audit 2026-04-14] CRITICAL: Meta API returning HTTP 403 Forbidden — ad account access has been lost. Needs immediate Business Manager audit. Also 16 days past due on post-onboarding call.'
  },
  {
    id: 'fire-3',
    label: 'Ideal Weightloss — $919 spent, 0 leads (30 days)',
    severity: 'critical',
    type: 'create_task',
    listId: '901515083916',
    confirmText: 'Create ClickUp task: "Audit Ideal Weightloss — 0 leads / $919 spend"?',
    payload: {
      name: 'Audit Ideal Weightloss — 0 leads / $919 spend',
      description: '[T4 Audit 2026-04-14] Account spent $919.63 over 30 days with 0 leads generated. 7-day spend $137.26 also 0 leads. CTR 1.69% below portfolio average. Recommend pause pending creative/offer audit.',
      priority: 1
    }
  },
  {
    id: 'fire-4',
    label: 'Delray Laser — launch ready for 34 days, never launched',
    severity: 'critical',
    type: 'comment',
    taskId: '86c7g0uke',
    confirmText: 'Post comment on Delray Laser task flagging 34-day stall?',
    commentText: '[T4 Audit 2026-04-14] CRITICAL: Task has been "launch ready" for 34 days with no action (due 2026-03-11). Launch immediately or document hold reason by EOD today. Tagging: Julianne Navarro, Felipe Guimaraes.'
  },
  {
    id: 'fire-5',
    label: 'About Face & Body MedSpa — 47 days stuck, waiting on client',
    severity: 'high',
    type: 'comment',
    taskId: '86c6wyvym',
    confirmText: 'Post comment on About Face & Body task for final outreach?',
    commentText: '[T4 Audit 2026-04-14] HIGH: 47 days stuck at "waiting on client" (due 2026-02-26). Final outreach required within 48 hours or reclassify as ghosting/churned. Tagging: Autumn Ismirnioglou, Felipe Guimaraes.'
  },
  {
    id: 'fire-6',
    label: 'Body Slim WLC — CPL $69–81, highest spend account',
    severity: 'high',
    type: 'create_task',
    listId: '901515083916',
    confirmText: 'Create ClickUp task: "Body Slim WLC — CPL Audit ($69–81)"?',
    payload: {
      name: 'Body Slim WLC — CPL Audit ($69–81)',
      description: '[T4 Audit 2026-04-14] CPL: $69.37 (7d) / $81.18 (30d). Highest spend account at $4,231/week and $14,693/month. CPM $42.38 above portfolio average. CTR 2.13% below average. Creative refresh and audience expansion audit required.',
      priority: 2
    }
  },
  {
    id: 'fire-7',
    label: 'Servedia B2B — $101 CPL on $23k/month spend',
    severity: 'high',
    type: 'create_task',
    listId: '901515083916',
    confirmText: 'Create ClickUp task: "Servedia B2B CHF — CPL Audit ($101)"?',
    payload: {
      name: 'Servedia B2B CHF — CPL Audit ($101)',
      description: '[T4 Audit 2026-04-14] Internal B2B account: $6,068 this week at $101.13 CPL. 30-day: $23,377 at $89.23 CPL. CTR 0.38%. Review offer and landing page conversion, verify lead quality definition.',
      priority: 2
    }
  },
  {
    id: 'fire-8',
    label: '4 orphaned new sign-ups — unassigned, no due date',
    severity: 'high',
    type: 'bulk_comment',
    taskIds: ['86c95u3h0','86c93targ','86c93kmfu','86c93hccm'],
    confirmText: 'Post comment on 4 orphaned new sign-up tasks to flag for assignment?',
    commentText: '[T4 Audit 2026-04-14] HIGH: This client has no assignee and no due date set. Requires immediate triage and assignment by Felipe Guimaraes. Due date target: 2026-04-16.'
  },
  {
    id: 'fire-9',
    label: 'The MAC OKC — 0 leads this week, $498 spent',
    severity: 'medium',
    type: 'create_task',
    listId: '901515083916',
    confirmText: 'Create ClickUp task: "The MAC OKC — 24hr Monitor Flag"?',
    payload: {
      name: 'The MAC OKC — 24hr Monitor Flag (0 leads / $498 this week)',
      description: '[T4 Audit 2026-04-14] Account spent $498.52 this week with 0 leads. Last 30 days: $52.91 CPL on 60 leads — performance has degraded sharply. Media buyer to review campaign delivery by 2026-04-15 EOD.',
      priority: 3
    }
  },
  {
    id: 'fire-10',
    label: '5 accounts went dark — verify pause intent',
    severity: 'medium',
    type: 'create_task',
    listId: '901515083916',
    confirmText: 'Create ClickUp task: "Verify 5 Dark Accounts — Pause Intent Check"?',
    payload: {
      name: 'Verify 5 Dark Accounts — Pause Intent Check',
      description: '[T4 Audit 2026-04-14] 5 accounts show no data this week but had activity last 30 days: Luminous Laser ($18.75 CPL, 36 leads), MLA Ad Account ($80.48 CPL, 8 leads), Skinholic 2025, Cosmetic Surgery of NY, Dr. Michelle Zubair. Confirm whether pauses are intentional before taking action.',
      priority: 3
    }
  }
];

const t4Executed = new Map(); // actionId → { state: 'idle'|'confirming'|'executing'|'done'|'error', msg?: string }
```

- [ ] **Step 2: Verify in console**

Reload dashboard. Open DevTools → Console and run:
```js
T4_ACTIONS.length
```
Expected output: `10`

Also run:
```js
t4Executed instanceof Map
```
Expected output: `true`

---

### Task 4: Add renderT4Actions, t4StartConfirm, t4Cancel, t4Execute

**Files:**
- Modify: `index.html` — insert immediately before `function renderCommandCenter()` (around line 2577)

- [ ] **Step 1: Insert the four T4 functions**

Find this exact line:
```js
function renderCommandCenter() {
```

Insert immediately before it:
```js
// ══════════════════════════════════════════════════════════════
// T4 ACTION QUEUE — render + execute
// ══════════════════════════════════════════════════════════════
function renderT4Actions() {
  const section = document.getElementById('ccT4Section');
  const list    = document.getElementById('ccT4List');
  if (!section || !list) return;

  const cfg      = getCfg();
  const hasToken = !!cfg.token;

  const pending = T4_ACTIONS.filter(a => {
    const s = t4Executed.get(a.id);
    return !s || s.state !== 'done';
  });

  if (pending.length === 0) {
    section.style.display = 'none';
    return;
  }

  section.style.display = '';

  const sevLabel = { critical: '🔴 CRITICAL', high: '🟠 HIGH', medium: '🟡 MEDIUM' };
  const sevCls   = { critical: 't4-sev-critical', high: 't4-sev-high', medium: 't4-sev-medium' };

  list.innerHTML = T4_ACTIONS.map(action => {
    const exec   = t4Executed.get(action.id) || { state: 'idle' };
    const isDone = exec.state === 'done';

    let execHtml = '';
    if (!hasToken) {
      execHtml = `<span style="font-size:11px;color:#444">Set token in ⚙ Config to enable</span>`;
    } else if (exec.state === 'idle') {
      execHtml = `<button class="t4-btn" onclick="t4StartConfirm('${action.id}')">Execute</button>`;
    } else if (exec.state === 'confirming') {
      execHtml = `
        <span style="font-size:11px;color:#ccc;margin-right:6px;flex:1">${esc(action.confirmText)}</span>
        <button class="t4-btn t4-btn-confirm" onclick="t4Execute('${action.id}')">✓ Confirm</button>
        <button class="t4-btn t4-btn-cancel" onclick="t4Cancel('${action.id}')">✗ Cancel</button>`;
    } else if (exec.state === 'executing') {
      execHtml = `<span class="t4-status executing">⏳ Running…</span>`;
    } else if (exec.state === 'done') {
      execHtml = `<span class="t4-status done">${esc(exec.msg || '✅ Done')}</span>`;
    } else if (exec.state === 'error') {
      execHtml = `<span class="t4-status error">❌ Failed: ${esc(exec.msg || 'unknown error')}</span>`;
    }

    return `<div class="t4-card ${sevCls[action.severity] || ''} ${isDone ? 't4-done' : ''}">
      <div style="display:flex;align-items:center;gap:8px;margin-bottom:4px">
        <span style="font-size:10px;font-weight:800;color:#555">${sevLabel[action.severity] || ''}</span>
      </div>
      <div class="t4-label">${esc(action.label)}</div>
      <div class="t4-execute-wrap">${execHtml}</div>
    </div>`;
  }).join('');
}

function t4StartConfirm(id) {
  t4Executed.set(id, { state: 'confirming' });
  renderT4Actions();
}

function t4Cancel(id) {
  t4Executed.set(id, { state: 'idle' });
  renderT4Actions();
}

async function t4Execute(id) {
  const action = T4_ACTIONS.find(a => a.id === id);
  if (!action) return;
  const cfg = getCfg();
  if (!cfg.token) return;

  t4Executed.set(id, { state: 'executing' });
  renderT4Actions();

  const headers = { 'Authorization': cfg.token, 'Content-Type': 'application/json' };

  try {
    if (action.type === 'comment') {
      const resp = await fetch(`${CU_API}/task/${action.taskId}/comment`, {
        method: 'POST',
        headers,
        body: JSON.stringify({ comment_text: action.commentText, notify_all: false })
      });
      if (!resp.ok) {
        const e = await resp.json().catch(() => ({}));
        throw new Error(e.err || `HTTP ${resp.status}`);
      }
      t4Executed.set(id, { state: 'done', msg: '✅ Done' });

    } else if (action.type === 'bulk_comment') {
      const results = await Promise.allSettled(
        action.taskIds.map(tid =>
          fetch(`${CU_API}/task/${tid}/comment`, {
            method: 'POST',
            headers,
            body: JSON.stringify({ comment_text: action.commentText, notify_all: false })
          }).then(r => { if (!r.ok) throw new Error(`HTTP ${r.status}`); return r; })
        )
      );
      const succeeded = results.filter(r => r.status === 'fulfilled').length;
      const total     = action.taskIds.length;
      if (succeeded === total) {
        t4Executed.set(id, { state: 'done', msg: `✅ ${total}/${total}` });
      } else {
        const failed = total - succeeded;
        t4Executed.set(id, { state: 'done', msg: `⚠️ ${succeeded}/${total} — ${failed} failed` });
      }

    } else if (action.type === 'create_task') {
      const resp = await fetch(`${CU_API}/list/${action.listId}/task`, {
        method: 'POST',
        headers,
        body: JSON.stringify(action.payload)
      });
      if (!resp.ok) {
        const e = await resp.json().catch(() => ({}));
        throw new Error(e.err || `HTTP ${resp.status}`);
      }
      t4Executed.set(id, { state: 'done', msg: '✅ Task created' });
    }
  } catch(e) {
    t4Executed.set(id, { state: 'error', msg: e.message });
    setTimeout(() => {
      if ((t4Executed.get(id) || {}).state === 'error') {
        t4Executed.set(id, { state: 'idle' });
        renderT4Actions();
      }
    }, 4000);
  }

  renderT4Actions();
}

```

- [ ] **Step 2: Verify functions load**

Reload dashboard. Open DevTools → Console and run:
```js
typeof renderT4Actions
```
Expected output: `"function"`

Run:
```js
typeof t4Execute
```
Expected output: `"function"`

---

### Task 5: Wire renderT4Actions into renderCommandCenter and commit

**Files:**
- Modify: `index.html` — add one call at the end of `renderCommandCenter()`

- [ ] **Step 1: Add renderT4Actions() call at end of renderCommandCenter**

Inside `renderCommandCenter()`, find the last section — the All Clear logic. It looks like this:
```js
  // All-clear
  const anyCritical = (fires && fires.length > 0) ||
```

If that exact pattern isn't found, look for the closing of the function — the last `}` of `renderCommandCenter`. Find the line just before it that ends the Slack section:
```js
    document.getElementById('ccSlackSection').style.display = msgs.length === 0 ? 'none' : '';
```

In either case, find the `}` that closes `renderCommandCenter` and insert `renderT4Actions();` on the line immediately before it:

```js
  renderT4Actions();
}
```

The final closing of `renderCommandCenter` should look like:
```js
  // ... existing all-clear logic ...
  renderT4Actions();
}
```

- [ ] **Step 2: Manual end-to-end verification**

Reload dashboard and navigate to Command Center. Verify:

1. **T4 section appears** — `⚡ T4 ACTION QUEUE` section is visible with all 10 fire cards
2. **Severity borders** — critical cards have red left border, high have orange, medium have yellow
3. **No token state** — if `cfg.token` is empty (open ⚙ Config, clear the token field), each card shows `"Set token in ⚙ Config to enable"` instead of Execute button
4. **Execute flow (with token set)** — click Execute on any card → confirm text + Confirm/Cancel buttons appear inline → click Cancel → returns to Execute button
5. **Confirm flow** — click Execute → click Confirm → card shows `⏳ Running…` then either `✅ Done` / error message

For a safe smoke test without a real token, open DevTools Console and run:
```js
t4StartConfirm('fire-1');
```
Expected: fire-1 card shows confirm text with Confirm/Cancel buttons.

```js
t4Cancel('fire-1');
```
Expected: fire-1 card returns to Execute button.

- [ ] **Step 3: Commit**

```bash
cd D:/GitHub/servedia-ops
git add index.html
git commit -m "feat: T4 action queue — inline execute buttons in Command Center

Adds 10 T4 audit actions to Command Center with inline confirm/execute
flow. Supports comment, bulk_comment, and create_task ClickUp API writes.
Token-gated — shows hint when cfg.token is unset.

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

- [ ] **Step 4: Push to remote**

```bash
git push origin main
```

Expected: GitHub Actions `deploy-pages.yml` triggers and publishes the updated dashboard to GitHub Pages within ~60 seconds.
