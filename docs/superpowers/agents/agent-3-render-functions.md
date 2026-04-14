# Agent 3 — renderCommandCenter() + updateNavBadges() + loadAll() Wiring

> **Working directory for this agent:** `D:/GitHub/servedia-ops-a3`
> **Branch:** `feat/agent3-render-fns`
> **Owns:** New JS functions (insert before loadAll at ~line 2629) + 2 lines inside loadAll()
> **Do NOT touch:** HTML, CSS, switchTab/switchSection, any render function other than loadAll()

---

## Paste this prompt into Claude Code

```
You are implementing Task 3 of a 4-agent parallel build on the Servedia Ops dashboard.

PROJECT CONTEXT:
- Single file app: D:/GitHub/servedia-ops-a3/index.html (~3272 lines, vanilla HTML/CSS/JS)
- The Command Center panel HTML exists with these element IDs (added by another agent):
  ccDate, ccActiveAccounts, ccSpend7, ccReviewCount, ccVideoCount, ccUrgentCount, ccSlackCount,
  ccFireSection, ccFireGrid, ccVideoList, ccTaskList, ccSlackList, ccAllClear
- Global data objects available after loadAll() runs:
  metaData  — { clients: [{name, status, stats:{d7:{spend,leads,cost_per_lead}, d30:{spend}}, ...}] }
  cuData    — { tasks: [{name, status:{status}, priority:{priority}, due_date, assignees:[{username}], ...}] }
  vidData   — { tasks: [{name, status:{status}, client_name, assignees:[{username}], ...}] }
  maintData — { tasks: [{...}] }
  slData    — { messages: [{text, user, ts, ...}] } OR { items: [...] }
  clientMgmtData — keyed by ad account ID, has .clientHappiness, .refundNeeded, .renewalDate
- Byron Vale's ClickUp user ID: 204101467
- Anka matched by username containing: anka, anca, alemparo, elamparo

EXISTING HELPER FUNCTIONS you can call:
  fmt(n)         — formats number with commas
  fmtMoney(n)    — formats as $X,XXX
  esc(s)         — HTML escapes a string
  displayMe(s)   — replaces Anka/Anca/Alemparo names with "Me"
  classifyStatus(s) — maps ClickUp status string to bucket: 'todo'|'inprogress'|'waiting'|'approval'|'complete'|'needsinfo'
  calculateChurnRisk(rec) — returns 0-100 churn risk score

YOUR TASK:

--- PART A: Add renderCommandCenter() function ---

Find this comment block (just before the loadAll function, around line 2628):
```
// ══════════════════════════════════════════════════════════════
// ══ DATA LOADING ══
// ══════════════════════════════════════════════════════════════
```

Insert the following TWO new functions immediately BEFORE that comment block:

```javascript
// ══════════════════════════════════════════════════════════════
// COMMAND CENTER
// ══════════════════════════════════════════════════════════════
function updateNavBadges() {
  // Meta — active accounts
  if (metaData && metaData.clients) {
    const active = metaData.clients.filter(c => c.status === 'ACTIVE').length;
    const el = document.getElementById('navBadge-meta');
    if (el) { el.textContent = active || '–'; el.className = 'nav-badge ' + (active > 0 ? 'nb-orange' : 'nb-gray'); }
  }

  // Video — pending approval tickets
  if (vidData && vidData.tasks) {
    const pending = vidData.tasks.filter(t => {
      const s = (t.status && t.status.status || '').toLowerCase();
      return s.includes('approval') || s.includes('review');
    }).length;
    const el = document.getElementById('navBadge-video');
    if (el) { el.textContent = pending || '–'; el.className = 'nav-badge ' + (pending > 0 ? 'nb-purple' : 'nb-gray'); }
  }

  // Tasks — urgent + overdue
  if (cuData && cuData.tasks) {
    const now = Date.now();
    const urgent = cuData.tasks.filter(t => {
      const s = classifyStatus(t.status && t.status.status || '');
      if (s === 'complete') return false;
      const isUrgent = (t.priority && t.priority.priority === 'urgent');
      const isOverdue = t.due_date && Number(t.due_date) < now;
      return isUrgent || isOverdue;
    }).length;
    const el = document.getElementById('navBadge-tasks');
    if (el) { el.textContent = urgent || '–'; el.className = 'nav-badge ' + (urgent > 0 ? 'nb-red' : 'nb-gray'); }
  }

  // Slack — unread messages
  if (slData) {
    const msgs = slData.messages || slData.items || [];
    const count = msgs.length;
    const el = document.getElementById('navBadge-slack');
    if (el) { el.textContent = count || '–'; el.className = 'nav-badge ' + (count > 0 ? 'nb-yellow' : 'nb-gray'); }
  }

  // Maintenance — clients to review (has maintenance task assigned to Byron/Anka)
  if (maintData && maintData.tasks && metaData && metaData.clients) {
    const byronAnka = (a) => {
      const u = (a.username || a.email || '').toLowerCase();
      return u.includes('byron') || u.includes('anka') || u.includes('anca') || u.includes('alemparo') || u.includes('elamparo') || String(a.id) === '204101467';
    };
    const count = maintData.tasks.filter(t => t.assignees && t.assignees.some(byronAnka)).length;
    const el = document.getElementById('navBadge-maintenance');
    if (el) { el.textContent = count || '–'; el.className = 'nav-badge ' + (count > 0 ? 'nb-red' : 'nb-gray'); }
  }

  // Retention — at-risk clients
  if (clientMgmtData && Object.keys(clientMgmtData).length) {
    const atRisk = Object.values(clientMgmtData).filter(r => calculateChurnRisk(r) >= 30).length;
    const el = document.getElementById('navBadge-retention');
    if (el) { el.textContent = atRisk || '–'; el.className = 'nav-badge ' + (atRisk > 0 ? 'nb-orange' : 'nb-gray'); }
  }
}

function renderCommandCenter() {
  // Date greeting
  const dateEl = document.getElementById('ccDate');
  if (dateEl) {
    const d = new Date();
    const h = d.getHours();
    const greeting = h < 12 ? 'Good morning' : h < 17 ? 'Good afternoon' : 'Good evening';
    dateEl.textContent = `${greeting}, Byron — ${d.toLocaleDateString('en-US', {weekday:'long', month:'long', day:'numeric'})}`;
  }

  // Stat cards
  if (metaData && metaData.clients) {
    const clients = metaData.clients;
    const active = clients.filter(c => c.status === 'ACTIVE');
    const spend7 = active.reduce((s, c) => s + (c.stats && c.stats.d7 && c.stats.d7.spend || 0), 0);
    const el1 = document.getElementById('ccActiveAccounts');
    const el2 = document.getElementById('ccSpend7');
    if (el1) el1.textContent = active.length;
    if (el2) el2.textContent = fmtMoney(spend7);
  }

  if (cuData && cuData.tasks) {
    const now = Date.now();
    const urgent = cuData.tasks.filter(t => {
      const s = classifyStatus(t.status && t.status.status || '');
      if (s === 'complete') return false;
      return (t.priority && t.priority.priority === 'urgent') || (t.due_date && Number(t.due_date) < now);
    });
    const el = document.getElementById('ccUrgentCount');
    if (el) { el.textContent = urgent.length; el.className = 'stat-value ' + (urgent.length > 0 ? 'red' : 'green'); }
  }

  if (vidData && vidData.tasks) {
    const pending = vidData.tasks.filter(t => {
      const s = (t.status && t.status.status || '').toLowerCase();
      return s.includes('approval') || s.includes('review');
    });
    const el = document.getElementById('ccVideoCount');
    if (el) { el.textContent = pending.length; el.className = 'stat-value ' + (pending.length > 0 ? 'purple' : 'green'); }
  }

  if (slData) {
    const msgs = slData.messages || slData.items || [];
    const el = document.getElementById('ccSlackCount');
    if (el) { el.textContent = msgs.length; el.className = 'stat-value ' + (msgs.length > 0 ? 'yellow' : 'green'); }
  }

  // Fires — Meta accounts with significant drops or flags
  const fireGrid = document.getElementById('ccFireGrid');
  if (fireGrid && metaData && metaData.clients) {
    const fires = metaData.clients.filter(c => {
      if (c.status !== 'ACTIVE') return false;
      const d7 = c.stats && c.stats.d7 || {};
      const cpl = d7.cost_per_lead || 0;
      const leads = d7.leads || 0;
      const cm = clientMgmtData && (clientMgmtData[c.account_id] || null);
      if (cm && cm.refundNeeded) return true;
      if (cm && (cm.clientHappiness || '').toLowerCase() === 'unhappy') return true;
      if (leads === 0 && d7.spend > 0) return true; // spending but no leads
      if (cpl > 0 && cpl > 300) return true; // very high CPL
      return false;
    }).slice(0, 6);

    if (fires.length === 0) {
      fireGrid.innerHTML = '<div class="no-data" style="color:#34d399">No fires right now ✓</div>';
    } else {
      fireGrid.innerHTML = fires.map(c => {
        const d7 = c.stats && c.stats.d7 || {};
        const cm = clientMgmtData && (clientMgmtData[c.account_id] || null);
        const why = [];
        if (cm && cm.refundNeeded) why.push('<span class="kbadge kb-red">Refund</span>');
        if (cm && (cm.clientHappiness || '').toLowerCase() === 'unhappy') why.push('<span class="kbadge kb-red">Unhappy</span>');
        if ((d7.leads || 0) === 0 && d7.spend > 0) why.push('<span class="kbadge kb-orange">0 Leads</span>');
        if ((d7.cost_per_lead || 0) > 300) why.push(`<span class="kbadge kb-orange">CPL $${Math.round(d7.cost_per_lead)}</span>`);
        return `<div class="accard flagged" style="padding:14px">
          <div class="ac-header">
            <div class="ac-name" title="${esc(c.name)}">${esc(c.name)}</div>
            <div style="display:flex;gap:4px;flex-wrap:wrap">${why.join('')}</div>
          </div>
          <div style="font-size:11px;color:#555">
            7d Spend: <b style="color:#e0e0e0">${fmtMoney(d7.spend||0)}</b> &nbsp;|&nbsp;
            Leads: <b style="color:#e0e0e0">${d7.leads||0}</b>
            ${d7.cost_per_lead ? `&nbsp;|&nbsp; CPL: <b style="color:#f87171">$${Math.round(d7.cost_per_lead)}</b>` : ''}
          </div>
        </div>`;
      }).join('');
    }
    document.getElementById('ccFireSection').style.display = fires.length === 0 ? 'none' : '';
  }

  // Video tickets needing review
  const ccVideoList = document.getElementById('ccVideoList');
  if (ccVideoList && vidData && vidData.tasks) {
    const pending = vidData.tasks.filter(t => {
      const s = (t.status && t.status.status || '').toLowerCase();
      return s.includes('approval') || s.includes('review');
    }).slice(0, 8);
    if (pending.length === 0) {
      ccVideoList.innerHTML = '<div class="no-data" style="color:#34d399">No video tickets awaiting review ✓</div>';
      document.getElementById('ccVideoSection').style.display = 'none';
    } else {
      ccVideoList.innerHTML = `<div style="display:flex;flex-direction:column;gap:6px">${
        pending.map(t => `<div style="background:#111;border:1px solid #1e1e1e;border-radius:8px;padding:10px 14px;display:flex;align-items:center;justify-content:space-between;gap:10px">
          <span style="font-size:13px;color:#e0e0e0;flex:1;min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${esc(t.name)}</span>
          <div style="display:flex;align-items:center;gap:8px;flex-shrink:0">
            ${t.client_name ? `<span style="font-size:11px;color:#555">${esc(t.client_name)}</span>` : ''}
            <span class="kbadge kb-approval">Needs Review</span>
          </div>
        </div>`).join('')
      }</div>`;
    }
  }

  // Urgent tasks
  const ccTaskList = document.getElementById('ccTaskList');
  if (ccTaskList && cuData && cuData.tasks) {
    const now = Date.now();
    const urgent = cuData.tasks.filter(t => {
      const s = classifyStatus(t.status && t.status.status || '');
      if (s === 'complete') return false;
      return (t.priority && t.priority.priority === 'urgent') || (t.due_date && Number(t.due_date) < now);
    }).slice(0, 8);
    if (urgent.length === 0) {
      ccTaskList.innerHTML = '<div class="no-data" style="color:#34d399">No urgent or overdue tasks ✓</div>';
      document.getElementById('ccTaskSection').style.display = 'none';
    } else {
      ccTaskList.innerHTML = `<div style="display:flex;flex-direction:column;gap:6px">${
        urgent.map(t => {
          const overdue = t.due_date && Number(t.due_date) < now;
          const isUrgent = t.priority && t.priority.priority === 'urgent';
          return `<div style="background:#111;border:1px solid #1e1e1e;border-radius:8px;padding:10px 14px;display:flex;align-items:center;justify-content:space-between;gap:10px">
            <span style="font-size:13px;color:#e0e0e0;flex:1;min-width:0;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${esc(t.name)}</span>
            <div style="display:flex;gap:4px;flex-shrink:0">
              ${isUrgent ? '<span class="kbadge kb-red">Urgent</span>' : ''}
              ${overdue ? '<span class="kbadge kb-orange">Overdue</span>' : ''}
            </div>
          </div>`;
        }).join('')
      }</div>`;
    }
  }

  // Slack
  const ccSlackList = document.getElementById('ccSlackList');
  if (ccSlackList && slData) {
    const msgs = (slData.messages || slData.items || []).slice(0, 6);
    if (msgs.length === 0) {
      ccSlackList.innerHTML = '<div class="no-data" style="color:#34d399">No Slack alerts ✓</div>';
      document.getElementById('ccSlackSection').style.display = 'none';
    } else {
      ccSlackList.innerHTML = `<div style="display:flex;flex-direction:column;gap:6px">${
        msgs.map(m => `<div style="background:#111;border:1px solid #1e1e1e;border-radius:8px;padding:10px 14px">
          <div style="font-size:11px;color:#a78bfa;margin-bottom:3px;font-weight:600">${esc(m.user || m.username || 'Slack')}</div>
          <div style="font-size:12px;color:#ccc;line-height:1.4">${esc((m.text || '').slice(0, 200))}</div>
        </div>`).join('')
      }</div>`;
    }
  }

  // All clear?
  const allClear = document.getElementById('ccAllClear');
  if (allClear) {
    const hasFires = document.getElementById('ccFireSection') && document.getElementById('ccFireSection').style.display !== 'none';
    const hasVideo = document.getElementById('ccVideoSection') && document.getElementById('ccVideoSection').style.display !== 'none';
    const hasTasks = document.getElementById('ccTaskSection') && document.getElementById('ccTaskSection').style.display !== 'none';
    const hasSlack = document.getElementById('ccSlackSection') && document.getElementById('ccSlackSection').style.display !== 'none';
    allClear.style.display = (!hasFires && !hasVideo && !hasTasks && !hasSlack) ? '' : 'none';
  }

  // Update nav badges while we're here
  updateNavBadges();
}

```

--- PART B: Wire into loadAll() ---

Inside loadAll(), find this line:
```
    renderMaintenance();
    renderBriefing();
    renderRetention();
```

Replace it with:
```
    renderMaintenance();
    renderRetention();
    renderCommandCenter();
    updateNavBadges();
```

(Remove the renderBriefing() call — that panel no longer exists.)

CONSTRAINTS:
- Do not touch switchTab, switchSection, CSS, HTML panels, or any other render function.
- Do not remove or restructure loadAll() — only change the 3 lines specified.
- The two new functions must be inserted BEFORE the loadAll function, not inside it.

VERIFICATION:
1. Search for "renderBriefing()" — should return 0 results (all removed).
2. Search for "renderCommandCenter()" — should return at least 2 results: the function definition and the call in loadAll().
3. Search for "updateNavBadges()" — should return at least 2 results.

COMMIT when done:
git add index.html
git commit -m "feat: add renderCommandCenter, updateNavBadges, wire into loadAll"

Report back: confirm renderBriefing is removed, both new functions exist and are called from loadAll.
```
