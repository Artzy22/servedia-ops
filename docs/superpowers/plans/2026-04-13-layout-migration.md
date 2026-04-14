# Layout Migration — Sidebar + Command Center

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the horizontal tab bar with a persistent left sidebar and add a Command Center home screen showing live stats from all sections.

**Architecture:** Single `index.html` — CSS layout shifts from stacked header+tabs+content to a full-height flex row (sidebar + main). The `switchTab()` function becomes `switchSection()` targeting sidebar nav items. A new `panel-command` div serves as the default home, populated by `renderCommandCenter()` which reads already-loaded global data objects (`metaData`, `cuData`, `vidData`, `slackData`, etc).

**Tech Stack:** Pure HTML/CSS/JS. No build step. Deployed to GitHub Pages via `main` branch. All changes in `index.html` only.

---

## File Map

| File | Action | What changes |
|------|--------|-------------|
| `index.html:11–36` | Modify | Remove `.header`, `.tabs`, `.tab`, `.tab-badge` CSS. Add `.app-shell`, `.sidebar`, `.nav-*`, `.main-content` CSS |
| `index.html:481–497` | Modify | Replace `<div class="header">` and `<div class="tabs">` HTML with sidebar HTML + app shell wrapper |
| `index.html:499–571` | Modify | Remove `panel-briefing` panel. Add `panel-command` Command Center panel |
| `index.html:1338–1344` | Modify | Replace `switchTab()` with `switchSection()` |
| `index.html:~2549` | Modify | `loadAll()` — call `renderCommandCenter()` after data loads |
| `index.html:~3177` | Modify | Boot call: `switchSection('command')` instead of `switchTab('briefing')` |
| `index.html` (all) | Modify | Update every `switchTab(...)` call to `switchSection(...)` |

---

## Task 1: App Shell CSS

Replace the tab-based layout CSS with a full-height flex shell.

**Files:**
- Modify: `index.html:11–500` (CSS section in `<style>`)

- [ ] **Step 1: Find and remove old layout CSS**

In `index.html`, find and **delete** these CSS blocks (they will be replaced):
```
/* ── Header ── */
.header{...}
.logo{...}
.header-meta{...}
.sync-info{...}
.sync-counter{...}
.hbtn{...}

/* ── Tabs ── */
.tabs{...}
.tabs::-webkit-scrollbar{...}
.tab{...}
.tab:hover{...}
.tab.active{...}
.tab-badge{...}
```

- [ ] **Step 2: Add app shell + sidebar CSS**

Add the following CSS block in the `<style>` tag, immediately after the `/* ── Reset ── */` block:

```css
/* ── App Shell ── */
body{font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;background:#0a0a0f;color:#e0e0e0;min-height:100vh;overflow:hidden}
.app-shell{display:flex;flex-direction:column;height:100vh}
.topbar{background:#111;border-bottom:1px solid #1a1a1a;padding:10px 20px;display:flex;align-items:center;justify-content:space-between;flex-shrink:0;z-index:100}
.topbar-brand{display:flex;align-items:center;gap:10px}
.topbar-logo{width:28px;height:28px;border-radius:8px;background:linear-gradient(135deg,#7c3aed,#a78bfa);display:flex;align-items:center;justify-content:center;font-size:14px;font-weight:900;color:#fff;flex-shrink:0}
.topbar-name{font-size:15px;font-weight:800;color:#fff}
.topbar-right{display:flex;align-items:center;gap:12px}
.topbar-sync{font-size:11px;color:#333}
.topbar-btn{background:#1a1a1a;border:1px solid #2a2a2a;color:#777;padding:5px 12px;border-radius:7px;cursor:pointer;font-size:12px;font-weight:600;transition:all .15s}
.topbar-btn:hover{border-color:#7c3aed;color:#fff}

/* ── Body Row ── */
.body-row{display:flex;flex:1;overflow:hidden}

/* ── Sidebar ── */
.sidebar{width:195px;flex-shrink:0;background:#09090f;border-right:1px solid #13131f;display:flex;flex-direction:column;overflow-y:auto;padding:12px 8px 16px}
.sidebar::-webkit-scrollbar{width:4px}
.sidebar::-webkit-scrollbar-thumb{background:#1e1e2e;border-radius:4px}
.nav-group{margin-bottom:18px}
.nav-group-label{font-size:8px;font-weight:800;letter-spacing:2px;text-transform:uppercase;color:#252535;padding:0 8px;margin-bottom:5px;display:block}
.nav-item{display:flex;align-items:center;justify-content:space-between;padding:7px 8px;border-radius:7px;cursor:pointer;margin-bottom:2px;border-left:2px solid transparent;transition:all .15s}
.nav-item:hover{background:#13131f}
.nav-item.active{background:#1a1730;border-left-color:#7c3aed}
.nav-item-left{display:flex;align-items:center;gap:8px}
.nav-icon{font-size:13px;line-height:1}
.nav-label{font-size:11px;font-weight:600;color:#555;transition:color .15s}
.nav-item:hover .nav-label{color:#aaa}
.nav-item.active .nav-label{color:#c4b5fd}
.nav-badge{font-size:9px;font-weight:800;padding:2px 6px;border-radius:6px}
.nb-red{background:#450a0a;color:#f87171}
.nb-orange{background:#2a1200;color:#fb923c}
.nb-purple{background:#1e1b33;color:#a78bfa}
.nb-yellow{background:#2a1a00;color:#fbbf24}
.nb-green{background:#052e16;color:#34d399}
.nb-gray{background:#1a1a1a;color:#555}
.nav-src{font-size:8px;font-weight:700;color:#1e1e2e;letter-spacing:.5px}

/* ── Main Content ── */
.main-content{flex:1;overflow-y:auto;background:#0b0b12}
.panel{display:none}.panel.active{display:block}
.content{padding:20px 24px}
```

- [ ] **Step 3: Verify CSS parses**

Open `index.html` in a browser (double-click the file). The page should load without a white screen. The old tab layout will be broken — that's expected until Task 2.

---

## Task 2: Replace HTML Structure

Replace the old `<header>` + `<div class="tabs">` HTML with the new sidebar shell.

**Files:**
- Modify: `index.html` (HTML body section, around line 481–500)

- [ ] **Step 1: Find the old header + tabs HTML**

Locate this block in `index.html` (around line 481):
```html
<div class="header">
  <div class="logo">Servedia <span>Ops</span></div>
  ...
</div>
<div class="tabs">
  <div class="tab active" id="tab-briefing" ...>☀️ Morning Briefing ...
  <div class="tab" id="tab-meta" ...>📊 Meta Ads ...
  <div class="tab" id="tab-maintenance" ...>✅ Daily Maintenance ...
  <div class="tab" id="tab-tasks" ...>☑ Tasks ...
  <div class="tab" id="tab-video" ...>🎬 Video Editing ...
  <div class="tab" id="tab-slack" ...>💬 Slack ...
  <div class="tab" id="tab-retention" ...>⚠️ Retention ...
</div>
```

- [ ] **Step 2: Replace with app shell HTML**

Replace the entire header + tabs block with:

```html
<div class="app-shell">

<!-- ── Top Bar ── -->
<div class="topbar">
  <div class="topbar-brand">
    <div class="topbar-logo">S</div>
    <span class="topbar-name">Servedia Ops</span>
  </div>
  <div class="topbar-right">
    <span class="topbar-sync" id="syncInfo">Loading...</span>
    <button class="topbar-btn" onclick="openSettings()">⚙ Settings</button>
  </div>
</div>

<!-- ── Body Row ── -->
<div class="body-row">

<!-- ── Sidebar ── -->
<nav class="sidebar">

  <!-- Command Center -->
  <div class="nav-group">
    <div class="nav-item active" id="nav-command" onclick="switchSection('command')">
      <div class="nav-item-left">
        <span class="nav-icon">⚡</span>
        <span class="nav-label">Command Center</span>
      </div>
    </div>
  </div>

  <!-- Media Buying -->
  <div class="nav-group">
    <span class="nav-group-label">Media Buying</span>
    <div class="nav-item" id="nav-meta" onclick="switchSection('meta')">
      <div class="nav-item-left"><span class="nav-icon">📊</span><span class="nav-label">Meta Ads</span></div>
      <span class="nav-badge nb-orange" id="navBadge-meta">–</span>
    </div>
    <div class="nav-item" id="nav-maintenance" onclick="switchSection('maintenance')">
      <div class="nav-item-left"><span class="nav-icon">🗂️</span><span class="nav-label">Daily Maintenance</span></div>
      <span class="nav-badge nb-red" id="navBadge-maintenance">–</span>
    </div>
    <div class="nav-item" id="nav-retention" onclick="switchSection('retention')">
      <div class="nav-item-left"><span class="nav-icon">🏆</span><span class="nav-label">Creative Intel</span></div>
      <span class="nav-badge nb-gray" id="navBadge-retention">–</span>
    </div>
  </div>

  <!-- Clients -->
  <div class="nav-group">
    <span class="nav-group-label">Clients</span>
    <div class="nav-item" id="nav-crm" onclick="switchSection('crm')">
      <div class="nav-item-left"><span class="nav-icon">🤝</span><span class="nav-label">CRM</span></div>
      <span class="nav-badge nb-yellow" id="navBadge-crm">–</span>
    </div>
    <div class="nav-item" id="nav-onboarding" onclick="switchSection('onboarding')">
      <div class="nav-item-left"><span class="nav-icon">🚀</span><span class="nav-label">Onboarding</span></div>
    </div>
    <div class="nav-item" id="nav-watchdog" onclick="switchSection('watchdog')">
      <div class="nav-item-left"><span class="nav-icon">🆕</span><span class="nav-label">Watchdog</span></div>
      <span class="nav-badge nb-orange" id="navBadge-watchdog">–</span>
    </div>
  </div>

  <!-- Work Queue -->
  <div class="nav-group">
    <span class="nav-group-label">Work Queue</span>
    <div class="nav-item" id="nav-video" onclick="switchSection('video')">
      <div class="nav-item-left"><span class="nav-icon">🎬</span><span class="nav-label">Video Tickets</span></div>
      <span class="nav-badge nb-purple" id="navBadge-video">–</span>
    </div>
    <div class="nav-item" id="nav-tech" onclick="switchSection('tech')">
      <div class="nav-item-left"><span class="nav-icon">⚙️</span><span class="nav-label">Tech Tickets</span></div>
      <span class="nav-badge nb-red" id="navBadge-tech">–</span>
    </div>
    <div class="nav-item" id="nav-slack" onclick="switchSection('slack')">
      <div class="nav-item-left"><span class="nav-icon">💬</span><span class="nav-label">Slack</span></div>
      <span class="nav-badge nb-yellow" id="navBadge-slack">–</span>
    </div>
    <div class="nav-item" id="nav-tasks" onclick="switchSection('tasks')">
      <div class="nav-item-left"><span class="nav-icon">☑️</span><span class="nav-label">Tasks</span></div>
      <span class="nav-badge nb-gray" id="navBadge-tasks">–</span>
    </div>
  </div>

</nav>

<!-- ── Main Content ── -->
<div class="main-content" id="mainContent">
```

- [ ] **Step 3: Close the main-content and body-row divs**

At the very bottom of `index.html`, just before the closing `</body>` tag, find the existing closing tags and ensure the structure closes correctly:

```html
</div><!-- /#mainContent .main-content -->
</div><!-- /.body-row -->
</div><!-- /.app-shell -->
```

- [ ] **Step 4: Verify in browser**

Open `index.html`. You should see:
- A dark topbar with "S Servedia Ops" logo on the left and "⚙ Settings" button on the right
- A sidebar on the left with all nav groups (Media Buying / Clients / Work Queue)
- Main content area on the right

The panels inside won't render correctly yet — that's fixed in Task 3.

---

## Task 3: Replace switchTab with switchSection

Update the navigation function and remove all references to old tab IDs.

**Files:**
- Modify: `index.html` (JavaScript section, ~line 1338)

- [ ] **Step 1: Replace switchTab function**

Find (around line 1338):
```javascript
function switchTab(tab) {
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
  document.getElementById('tab-' + tab).classList.add('active');
  document.getElementById('panel-' + tab).classList.add('active');
  if (tab === 'retention') renderRetention();
}
```

Replace with:
```javascript
function switchSection(section) {
  document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
  document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
  const navEl = document.getElementById('nav-' + section);
  if (navEl) navEl.classList.add('active');
  const panelEl = document.getElementById('panel-' + section);
  if (panelEl) panelEl.classList.add('active');
  if (section === 'retention') renderRetention();
  if (section === 'command') renderCommandCenter();
}

// keep switchTab as an alias so existing internal calls don't break during migration
function switchTab(tab) { switchSection(tab); }
```

- [ ] **Step 2: Update boot call**

Find near the bottom of the JS (around line 3177):
```javascript
switchTab('briefing');
```
Replace with:
```javascript
switchSection('command');
```

- [ ] **Step 3: Update sync info display**

Find where `syncInfo` or the old header sync text is set (search for `sync-info` or `lastSync`). Update it to target the new `id="syncInfo"` span in the topbar. 

Search for:
```javascript
document.querySelector('.sync-info')
```
If found, replace with:
```javascript
document.getElementById('syncInfo')
```

If the sync text is set inline in the HTML, ensure it references `id="syncInfo"`.

- [ ] **Step 4: Verify navigation works**

Open `index.html` in browser. Click each sidebar item. Panels should switch. The active sidebar item should highlight with purple left border. No JS errors in console.

---

## Task 4: Remove Morning Briefing Panel, Add Command Center Panel

**Files:**
- Modify: `index.html` (HTML panels section, around line 539)

- [ ] **Step 1: Delete the Morning Briefing panel**

Find and delete the entire block (around line 539–571):
```html
<div id="panel-briefing" class="panel active content">
  ...
</div>
```

- [ ] **Step 2: Add Command Center panel in its place**

Insert the following immediately after the closing `</div>` of `panel-meta`:

```html
<!-- ════════════════ COMMAND CENTER ════════════════ -->
<div id="panel-command" class="panel content">

  <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:18px;flex-wrap:wrap;gap:12px">
    <div>
      <div style="font-size:22px;font-weight:900;color:#fff">⚡ Command Center</div>
      <div style="font-size:12px;color:#333;margin-top:3px" id="ccDate"></div>
    </div>
    <div style="display:flex;align-items:center;gap:6px;font-size:11px;color:#34d399;background:#052e16;border:1px solid #065f46;padding:5px 12px;border-radius:20px">
      <span style="width:6px;height:6px;border-radius:50%;background:#34d399;display:inline-block;animation:ccPulse 2s infinite"></span>
      Live
    </div>
  </div>

  <!-- Alert chips -->
  <div id="ccAlertBar" style="display:flex;gap:8px;margin-bottom:18px;flex-wrap:wrap"></div>

  <!-- Meta KPI row -->
  <div style="font-size:9px;font-weight:800;letter-spacing:2px;text-transform:uppercase;color:#2a2a3a;margin-bottom:8px">📊 Meta Ads — 7 Day</div>
  <div style="display:grid;grid-template-columns:repeat(5,1fr);gap:10px;margin-bottom:18px" id="ccMetaKpis"></div>

  <!-- Active alerts -->
  <div style="font-size:9px;font-weight:800;letter-spacing:2px;text-transform:uppercase;color:#2a2a3a;margin-bottom:8px">🚨 Active Alerts</div>
  <div style="display:grid;grid-template-columns:repeat(3,1fr);gap:10px;margin-bottom:18px" id="ccAlerts"></div>

  <!-- Bottom row: Work Queue + Client Health -->
  <div style="display:grid;grid-template-columns:1fr 1fr;gap:14px">

    <div>
      <div style="font-size:9px;font-weight:800;letter-spacing:2px;text-transform:uppercase;color:#2a2a3a;margin-bottom:8px">⚙️ Work Queue</div>
      <div id="ccWorkQueue" style="display:flex;flex-direction:column;gap:8px"></div>
    </div>

    <div>
      <div style="font-size:9px;font-weight:800;letter-spacing:2px;text-transform:uppercase;color:#2a2a3a;margin-bottom:8px">🤝 Client Health</div>
      <div id="ccHealthGrid" style="display:grid;grid-template-columns:repeat(2,1fr);gap:8px"></div>
      <div id="ccHealthFooter" style="font-size:10px;color:#2a2a3a;text-align:center;margin-top:6px"></div>
    </div>

  </div>

</div>
```

- [ ] **Step 3: Add pulse animation CSS**

In the `<style>` block, add:
```css
@keyframes ccPulse{0%,100%{opacity:1}50%{opacity:.35}}
```

- [ ] **Step 4: Verify panel renders**

Open browser, page should load to an empty Command Center panel (all sections blank until Task 5 wires data). No errors in console.

---

## Task 5: renderCommandCenter() Function

Wire the Command Center panel to live data from the already-loaded global variables.

**Files:**
- Modify: `index.html` (JavaScript section — add after `switchSection` function)

- [ ] **Step 1: Add renderCommandCenter function**

Add the following function immediately after `switchSection()`:

```javascript
function renderCommandCenter() {
  // Date
  const ccDate = document.getElementById('ccDate');
  if (ccDate) ccDate.textContent = new Date().toLocaleDateString('en-US',{weekday:'long',month:'long',day:'numeric',year:'numeric'});

  // ── Meta KPIs ──
  const clients = (metaData && metaData.clients) || [];
  const active  = clients.filter(c => c.status === 'active');
  const totalSpend  = active.reduce((s,c) => s + ((c.last_7_days||{}).spend||0), 0);
  const totalLeads  = active.reduce((s,c) => s + ((c.last_7_days||{}).leads||0), 0);
  const cpls        = active.filter(c => (c.last_7_days||{}).cost_per_lead > 0).map(c => c.last_7_days.cost_per_lead);
  const avgCPL      = cpls.length ? cpls.reduce((a,b)=>a+b,0)/cpls.length : 0;
  const t           = getCPLThresholds();
  const redAccounts = active.filter(c => (c.last_7_days||{}).cost_per_lead > t.ok).length;
  const zeroLeads   = active.filter(c => (c.last_7_days||{}).leads === 0).length;

  const kpiEl = document.getElementById('ccMetaKpis');
  if (kpiEl) kpiEl.innerHTML = [
    {label:'Total Spend 7d', val:'$'+fmtNum(Math.round(totalSpend)), color:'#a78bfa'},
    {label:'Total Leads 7d', val:fmtNum(totalLeads), color:'#60a5fa'},
    {label:'Avg CPL', val:fmtCur(avgCPL), color: avgCPL < t.good ? '#34d399' : avgCPL < t.ok ? '#fbbf24' : '#f87171'},
    {label:'Red CPL Accounts', val:redAccounts, color:'#f87171'},
    {label:'Zero Leads', val:zeroLeads, color:'#fb923c'},
  ].map(k => `
    <div style="background:#111;border:1px solid #1a1a1a;border-radius:9px;padding:12px 14px">
      <div style="font-size:9px;color:#333;text-transform:uppercase;letter-spacing:.5px;margin-bottom:4px;font-weight:700">${k.label}</div>
      <div style="font-size:22px;font-weight:900;color:${k.color}">${k.val}</div>
    </div>
  `).join('');

  // ── Alert chips + Active Alerts panels ──
  const paymentIssues  = clients.filter(c => c.payment_issue);
  const violations     = clients.filter(c => (c.violations||[]).length > 0);
  const fatigued       = clients.filter(c => (c.fatigued_ads||[]).length > 0);
  const slackFlagged   = (slackData && slackData.flagged_messages) || [];

  const chipBar = document.getElementById('ccAlertBar');
  if (chipBar) {
    const chips = [];
    if (paymentIssues.length) chips.push({label:`🚨 ${paymentIssues.length} Billing Paused`, cls:'nb-red', section:'meta'});
    if (violations.length)    chips.push({label:`⚠️ ${violations.length} Policy Violations`, cls:'nb-orange', section:'meta'});
    if (fatigued.length)      chips.push({label:`🎨 ${fatigued.length} Creatives Fatigued`, cls:'nb-yellow', section:'meta'});
    if (slackFlagged.length)  chips.push({label:`💬 ${slackFlagged.length} Urgent Slack`, cls:'nb-purple', section:'slack'});
    chipBar.innerHTML = chips.map(c =>
      `<div onclick="switchSection('${c.section}')" style="cursor:pointer;display:flex;align-items:center;gap:6px;padding:5px 12px;border-radius:20px;font-size:11px;font-weight:700" class="${c.cls}">${c.label}</div>`
    ).join('') || '<div style="font-size:12px;color:#333">No active alerts</div>';
  }

  const alertsEl = document.getElementById('ccAlerts');
  if (alertsEl) alertsEl.innerHTML = [
    {
      title:'Billing Paused', color:'#f87171', bg:'#130505', border:'#450a0a',
      items: paymentIssues.map(c => c.account_name),
      empty: 'No billing issues'
    },
    {
      title:'Policy Violations', color:'#fb923c', bg:'#110700', border:'#2a1200',
      items: violations.map(c => `${c.account_name} — ${(c.violations||[]).length} ad${(c.violations||[]).length>1?'s':''}`),
      empty: 'No policy violations'
    },
    {
      title:'Creative Fatigue', color:'#fbbf24', bg:'#110d00', border:'#2a1a00',
      items: fatigued.map(c => `${c.account_name} — ${(c.fatigued_ads||[]).length} ad${(c.fatigued_ads||[]).length>1?'s':''}`),
      empty: 'No fatigue detected'
    }
  ].map(a => `
    <div style="background:${a.bg};border:1px solid ${a.border};border-radius:9px;padding:12px 14px">
      <div style="font-size:9px;font-weight:800;text-transform:uppercase;letter-spacing:1px;color:${a.color};margin-bottom:8px">${a.title}</div>
      ${a.items.length
        ? a.items.slice(0,4).map(item => `<div style="font-size:11px;color:#666;display:flex;align-items:center;gap:6px;margin-bottom:4px"><span style="width:5px;height:5px;border-radius:50%;background:${a.color};flex-shrink:0;display:inline-block"></span>${esc(item)}</div>`).join('')
          + (a.items.length > 4 ? `<div style="font-size:10px;color:#333;margin-top:2px">+${a.items.length-4} more</div>` : '')
        : `<div style="font-size:11px;color:#2a2a3a">${a.empty}</div>`
      }
    </div>
  `).join('');

  // ── Work Queue ──
  const wqEl = document.getElementById('ccWorkQueue');
  if (wqEl) {
    const videoTickets = (vidData && vidData.tasks) || [];
    const cuTasks      = (cuData  && cuData.tasks)  || [];
    const urgentTasks  = cuTasks.filter(t => (t.priority||'').toString() === '1' || (typeof t.priority === 'object' && t.priority?.priority === 'urgent'));
    wqEl.innerHTML = [
      {
        icon:'🎬', label:'Video Tickets', count: videoTickets.length, color:'#a78bfa',
        items: videoTickets.slice(0,2).map(t => {
          const s = typeof t.status === 'object' ? (t.status.status||'') : (t.status||'');
          return `${t.name||'Untitled'} <span style="color:#333">· ${s}</span>`;
        })
      },
      {
        icon:'⚙️', label:'Tech Tickets', count: 0, color:'#f87171',
        items: ['No tech tickets list connected yet']
      },
      {
        icon:'☑️', label:'Urgent Tasks', count: urgentTasks.length, color:'#fbbf24',
        items: urgentTasks.slice(0,2).map(t => t.name||'Untitled')
      }
    ].map(w => `
      <div style="background:#111;border:1px solid #1a1a1a;border-radius:9px;padding:10px 13px">
        <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:7px">
          <span style="font-size:11px;font-weight:700;color:#666">${w.icon} ${w.label}</span>
          <span style="font-size:15px;font-weight:900;color:${w.color}">${w.count}</span>
        </div>
        ${w.items.map(i => `<div style="font-size:10px;color:#333;padding:3px 0;border-bottom:1px solid #131313;display:flex;align-items:center;gap:5px"><span style="width:4px;height:4px;border-radius:50%;background:#2a2a3a;display:inline-block;flex-shrink:0"></span>${esc(String(i))}</div>`).join('')}
      </div>
    `).join('');
  }

  // ── Client Health snapshot ──
  const healthEl = document.getElementById('ccHealthGrid');
  const healthFooter = document.getElementById('ccHealthFooter');
  if (healthEl) {
    const dmClients = buildDmClients ? buildDmClients() : [];
    const withCPL   = dmClients.filter(c => c.cpl !== null && c.cpl > 0).sort((a,b) => b.cpl - a.cpl);
    const worst2    = withCPL.slice(0,2);
    const best2     = withCPL.slice(-2).reverse();
    const toShow    = [...worst2, ...best2].slice(0,4);

    healthEl.innerHTML = toShow.map(c => {
      const tier = getCPLTier(c.cpl);
      return `
        <div style="background:#111;border:1px solid #1a1a1a;border-radius:9px;padding:10px;text-align:center">
          <div style="font-size:13px;font-weight:900;color:${tier.color||'#e0e0e0'}">${fmtCur(c.cpl)}</div>
          <div style="font-size:9px;color:#444;margin-top:3px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap">${esc(c.name||'')}</div>
          <span style="font-size:8px;font-weight:700;padding:1px 5px;border-radius:3px;margin-top:4px;display:inline-block;background:${tier.bg||'#1a1a1a'};color:${tier.color||'#888'}">${tier.label}</span>
        </div>`;
    }).join('') || '<div style="font-size:12px;color:#333;grid-column:1/-1;padding:12px;text-align:center">No client data yet</div>';

    if (healthFooter) {
      const renewalCount = (typeof clientMgmtByName !== 'undefined' ? Object.values(clientMgmtByName) : [])
        .filter(r => r.renewalDate && Math.ceil((Number(r.renewalDate)-Date.now())/86400000) <= 30 && Math.ceil((Number(r.renewalDate)-Date.now())/86400000) >= 0).length;
      healthFooter.textContent = `${dmClients.length} clients · ${renewalCount} renewal${renewalCount!==1?'s':''} due this month`;
    }
  }

  // ── Update sidebar badges ──
  updateSidebarBadges();
}
```

- [ ] **Step 2: Add updateSidebarBadges function**

Add immediately after `renderCommandCenter()`:

```javascript
function updateSidebarBadges() {
  // Meta badge — accounts with issues
  const clients = (metaData && metaData.clients) || [];
  const t = getCPLThresholds();
  const metaIssues = clients.filter(c => c.payment_issue || (c.violations||[]).length || (c.fatigued_ads||[]).length || ((c.last_7_days||{}).cost_per_lead||0) > t.watch).length;
  _setBadge('navBadge-meta', metaIssues, 'nb-orange');

  // Maintenance badge — flagged clients (from localStorage dm_state)
  const dmState = getDmState ? getDmState() : {};
  const flagged = Object.values(dmState).filter(s => s.flagged && !s.complete).length;
  _setBadge('navBadge-maintenance', flagged, 'nb-red');

  // Video badge
  const videoCount = ((vidData && vidData.tasks)||[]).length;
  _setBadge('navBadge-video', videoCount, 'nb-purple');

  // Slack badge
  const slackCount = ((slackData && slackData.flagged_messages)||[]).length;
  _setBadge('navBadge-slack', slackCount, 'nb-yellow');

  // Tasks badge
  const cuTasks = (cuData && cuData.tasks)||[];
  const urgentCount = cuTasks.filter(t => (t.priority||'').toString() === '1' || (typeof t.priority === 'object' && t.priority?.priority === 'urgent')).length;
  _setBadge('navBadge-tasks', urgentCount, 'nb-gray');
}

function _setBadge(id, count, cls) {
  const el = document.getElementById(id);
  if (!el) return;
  if (!count) { el.style.display = 'none'; return; }
  el.style.display = '';
  el.textContent = count;
  el.className = 'nav-badge ' + cls;
}
```

- [ ] **Step 3: Call updateSidebarBadges after loadAll**

In `loadAll()` (around line 2549), find the final `.then()` or the end of the async function after all data is set. Add:

```javascript
updateSidebarBadges();
```

Right after the existing badge/UI update calls at the end of `loadAll`.

- [ ] **Step 4: Verify Command Center renders**

Open `index.html` in browser. The Command Center panel should show:
- A date line
- Alert chips (or "No active alerts" if no issues in data)
- 5 Meta KPI boxes with real numbers from `data/meta.json`
- 3 alert panels (Billing / Policy / Fatigue)
- Work Queue showing video ticket count and urgent task count
- Client health grid showing 4 client CPL scores

No JS errors in console.

---

## Task 6: Add Stub Panels for New Sections

Add placeholder panels for sections that don't exist yet (CRM, Onboarding, Watchdog, Tech Tickets) so sidebar navigation doesn't break.

**Files:**
- Modify: `index.html` (HTML panels section — add after existing panels)

- [ ] **Step 1: Add stub panels before the closing </div> of main-content**

Find the last panel in `index.html` (the settings panel or the modal section). Add the following stubs after the last panel:

```html
<!-- ════════════════ CRM (stub) ════════════════ -->
<div id="panel-crm" class="panel content">
  <div style="font-size:18px;font-weight:800;color:#fff;margin-bottom:8px">🤝 CRM</div>
  <div style="font-size:13px;color:#555">Client health scores, renewal pipeline, and retention signals. Coming in Plan 3.</div>
</div>

<!-- ════════════════ ONBOARDING (stub) ════════════════ -->
<div id="panel-onboarding" class="panel content">
  <div style="font-size:18px;font-weight:800;color:#fff;margin-bottom:8px">🚀 Onboarding</div>
  <div style="font-size:13px;color:#555">New client setup flow. Coming in Plan 3.</div>
</div>

<!-- ════════════════ WATCHDOG (stub) ════════════════ -->
<div id="panel-watchdog" class="panel content">
  <div style="font-size:18px;font-weight:800;color:#fff;margin-bottom:8px">🆕 Watchdog</div>
  <div style="font-size:13px;color:#555">30-day monitoring for new clients. Coming in Plan 3.</div>
</div>

<!-- ════════════════ TECH TICKETS (stub) ════════════════ -->
<div id="panel-tech" class="panel content">
  <div style="font-size:18px;font-weight:800;color:#fff;margin-bottom:8px">⚙️ Tech Tickets</div>
  <div style="font-size:13px;color:#555">Client funnel fixes and internal tech ops queue. Coming in Plan 3.</div>
</div>
```

- [ ] **Step 2: Verify all nav items navigate**

Click every sidebar item. Each should switch to its panel with no "element not found" JS errors. CRM / Onboarding / Watchdog / Tech Tickets should show their stub text.

---

## Task 7: Fix Internal switchTab Calls + Sync Info

Update the remaining `switchTab()` calls in the code to use `switchSection()`, and wire the sync info display to the new topbar element.

**Files:**
- Modify: `index.html` (JavaScript section)

- [ ] **Step 1: Update sync info target**

Search for the code that sets the old `.sync-info` element text (it will reference `syncInfo` or `.sync-info` or `lastSync`). In `loadAll()`, find lines like:

```javascript
document.querySelector('.sync-info').innerHTML = ...
```

Replace `.querySelector('.sync-info')` with `.getElementById('syncInfo')`. The format should be shortened to fit the topbar:

```javascript
const syncEl = document.getElementById('syncInfo');
if (syncEl) syncEl.textContent = 'Last sync: ' + new Date().toLocaleTimeString([],{hour:'2-digit',minute:'2-digit'});
```

- [ ] **Step 2: Find remaining switchTab calls**

Run a search in the file for `switchTab(` — the alias in Task 3 handles these, but verify none are broken. Key ones that reference old panel names:

- `switchTab('briefing')` → change to `switchSection('command')`
- `switchTab('maintenance')` → already works via alias
- `switchTab('video')` → already works via alias
- `switchTab('slack')` → already works via alias
- `switchTab('tasks')` → already works via alias
- `switchTab('retention')` → already works via alias (now labeled "Creative Intel")

- [ ] **Step 3: Final full verification**

Open `index.html` in browser and verify all of the following:

1. Page loads to Command Center with live Meta KPI numbers
2. Sidebar shows badge counts on Meta Ads, Daily Maintenance, Video Tickets, Slack, Tasks
3. All sidebar items navigate to their panels without errors
4. Meta Ads panel renders all 33 account cards correctly
5. Daily Maintenance panel renders client cards correctly
6. Video Editing panel renders tickets correctly
7. Slack panel renders messages correctly
8. Tasks panel renders correctly
9. Settings panel opens and CPL thresholds work
10. Creative Intel (Retention tab) renders correctly
11. No console errors

---

## Task 8: Commit and Deploy

- [ ] **Step 1: Stage and commit**

```bash
git add index.html
git commit -m "feat: migrate to sidebar layout with Command Center home

- Replace horizontal tab bar with persistent sidebar navigation
- Add Command Center panel with live Meta KPIs, alert chips, work queue, client health
- Add stub panels for CRM, Onboarding, Watchdog, Tech Tickets
- Remove Morning Briefing panel
- switchSection() replaces switchTab() (alias kept for compatibility)
- updateSidebarBadges() populates live badge counts"
```

- [ ] **Step 2: Push to main**

```bash
git pull --rebase origin main
git push origin main
```

- [ ] **Step 3: Verify GitHub Pages deployment**

Wait ~2 minutes. Open `https://artzy22.github.io/servedia-ops/` in browser.

Confirm:
- Sidebar visible on left
- Command Center loads as home
- Settings button works
- All navigation items functional

---

## Self-Review Checklist

**Spec coverage:**
- [x] Sidebar layout (Option B) — Tasks 1–2
- [x] Command Center home with alert chips, Meta KPIs, alerts, work queue, client health — Tasks 4–5
- [x] Morning Briefing removed — Task 4
- [x] Sidebar badges wired to live data — Task 5
- [x] New section stubs (CRM, Onboarding, Watchdog, Tech) — Task 6
- [x] All existing panels preserved (Meta, Maintenance, Video, Tasks, Slack, Retention/Creative Intel, Settings) — Task 7
- [x] Deploy to GitHub Pages — Task 8

**Gaps from spec not in this plan (intentional — covered in Plans 2 and 3):**
- F1–F13 GitHub Actions workflows → Plan 2
- CRM full implementation → Plan 3
- Onboarding full implementation → Plan 3
- Tech Tickets full implementation → Plan 3
- Watchdog full implementation → Plan 3
- Client health score formula → Plan 3

**Type consistency:** `switchSection`, `updateSidebarBadges`, `_setBadge`, `renderCommandCenter` — all defined in Task 3 and 5, referenced consistently throughout.

**No placeholders:** All code is complete and runnable.
