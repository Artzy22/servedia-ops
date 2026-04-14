# Multi-Role User System — Phase 1 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Wire localStorage-based identity into the Servedia Ops dashboard — user picker modal, SHA-256 admin password, role-based sidebar visibility, and assignment filter in Meta Ads.

**Architecture:** All changes land in `index.html` (single-file app, no build step). One new static JSON file `data/assignments.json` is created. On first load, a fullscreen modal prompts name selection; on subsequent loads the stored identity is read from localStorage and applied immediately. Role visibility and assignment filtering hook into existing render functions without modifying their core logic.

**Tech Stack:** Vanilla JS, localStorage, Web Crypto API (SHA-256), existing `fetchJ` utility, existing `renderMeta()` / `loadAll()` functions.

---

## File Map

| File | Action | Responsibility |
|------|--------|----------------|
| `data/assignments.json` | Create | Maps each client to their assigned media_buyer and CSM |
| `index.html:11–531` | Modify (CSS section) | Add modal overlay + sidebar footer + admin badge styles |
| `index.html:538–548` | Modify (topbar HTML) | Add user identity badge to topbar-right |
| `index.html:554–623` | Modify (sidebar HTML) | Add sidebar footer with "Logged in as / Switch user" |
| `index.html:533` | Insert (body HTML) | User picker modal HTML |
| `index.html:1276–1430` | Modify (constants) | Add `TEAM_MEMBERS`, `assignmentData`, user helpers |
| `index.html:1573` | Insert before | Add `getUser()`, `setUser()`, `clearUser()`, `showUserPicker()`, `checkAdminPassword()`, `applyRoleVisibility()`, `checkUser()` |
| `index.html:2652` | Modify | Add assignment filter toggle inside `renderMeta()` |
| `index.html:4161` | Modify | Add `assignments.json` as index 10 in `loadAll()` static fetches |
| `index.html:5368` | Modify | Call `checkUser()` before `loadAll()` in DOMContentLoaded |

---

## Task 1: Create `data/assignments.json`

**Files:**
- Create: `data/assignments.json`

- [ ] **Step 1: Create the file**

```json
{
  "assignments": [
    { "client_name": "HEC Wellness Center",          "media_buyer": "Felipe Guimaraes", "csm": "Adriana Gonzalez",  "team": "Ad-Vengers" },
    { "client_name": "Thorpe Wellness 2.0",          "media_buyer": "Anca Elamparo",    "csm": "Adriana Gonzalez",  "team": "Ad-Vengers" },
    { "client_name": "Delray Laser",                 "media_buyer": "Julianne Navarro", "csm": "Adriana Gonzalez",  "team": "Skinfluencers" },
    { "client_name": "About Face & Body MedSpa",     "media_buyer": "Julianne Navarro", "csm": "Adriana Gonzalez",  "team": "Skinfluencers" },
    { "client_name": "Ideal Weightloss",             "media_buyer": "Felipe Guimaraes", "csm": "Adriana Gonzalez",  "team": "Ad-Vengers" },
    { "client_name": "Body Slim WLC",                "media_buyer": "Anca Elamparo",    "csm": "Adriana Gonzalez",  "team": "Ad-Vengers" },
    { "client_name": "MAC OKC",                      "media_buyer": "Felipe Guimaraes", "csm": "Adriana Gonzalez",  "team": "Ad-Vengers" }
  ]
}
```

> **Note:** This is a starter file. Byron will populate the full 37-account list via the Admin Panel (Phase 3). The filter in `renderMeta()` does a case-insensitive substring match on `account_name`, so partial entries are safe.

- [ ] **Step 2: Commit**

```bash
git add data/assignments.json
git commit -m "feat: add assignments.json starter file for role-based filtering"
```

---

## Task 2: Add User System CSS

**Files:**
- Modify: `index.html` — CSS `<style>` block, just before the closing `</style>` tag at line 531

- [ ] **Step 1: Add CSS** — insert the following block immediately before `</style>` (line 531):

```css
/* ══════════════════════════════════════════════
   USER PICKER MODAL
══════════════════════════════════════════════ */
.up-ov{position:fixed;inset:0;background:rgba(0,0,0,.85);display:flex;align-items:center;justify-content:center;z-index:9999;backdrop-filter:blur(4px)}
.up-ov.hidden{display:none}
.up-box{background:#111;border:1px solid #2a2a2a;border-radius:16px;padding:32px 28px;width:340px;max-width:92vw}
.up-title{font-size:17px;font-weight:800;color:#fff;margin-bottom:6px}
.up-sub{font-size:12px;color:#555;margin-bottom:22px}
.up-label{font-size:11px;font-weight:700;color:#777;text-transform:uppercase;letter-spacing:.5px;margin-bottom:6px;display:block}
.up-select{width:100%;background:#1a1a1a;border:1px solid #333;color:#ccc;padding:10px 12px;border-radius:8px;font-size:13px;cursor:pointer;margin-bottom:16px}
.up-select:focus{outline:none;border-color:#7c3aed}
.up-pwrap{margin-bottom:16px}
.up-pinput{width:100%;background:#1a1a1a;border:1px solid #333;color:#ccc;padding:10px 12px;border-radius:8px;font-size:13px}
.up-pinput:focus{outline:none;border-color:#7c3aed}
.up-perr{font-size:11px;color:#f87171;margin-top:5px;display:none}
.up-btn{width:100%;background:#7c3aed;border:none;color:#fff;padding:11px;border-radius:9px;cursor:pointer;font-size:14px;font-weight:700;transition:background .15s}
.up-btn:hover{background:#6d28d9}
/* ── Sidebar footer ── */
.sidebar-footer{margin-top:auto;padding:10px 8px 0;border-top:1px solid #13131f}
.sf-user{font-size:11px;color:#555;margin-bottom:4px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.sf-name{color:#a78bfa;font-weight:700}
.sf-role{color:#444}
.sf-switch{font-size:10px;color:#333;cursor:pointer;text-decoration:underline;transition:color .15s}
.sf-switch:hover{color:#a78bfa}
/* ── Admin badge in topbar ── */
.admin-badge{background:#1e1b33;border:1px solid #7c3aed;color:#a78bfa;font-size:10px;font-weight:800;padding:3px 9px;border-radius:6px;letter-spacing:.5px;text-transform:uppercase}
```

- [ ] **Step 2: Verify no style conflicts** — search for `.up-ov` and `.sidebar-footer` in the file to confirm no duplicates:

```bash
grep -n "up-ov\|sidebar-footer\|admin-badge" index.html
```

Expected: only the lines you just added.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add user picker modal + sidebar footer CSS"
```

---

## Task 3: Add HTML — User Picker Modal + Sidebar Footer + Topbar Badge

**Files:**
- Modify: `index.html` — three HTML insertion points

- [ ] **Step 1: Add topbar user badge** — in the `<div class="topbar-right">` block (line 543), add the user badge span after the opening `<div class="topbar-right">` tag:

Current code at lines 543–548:
```html
  <div class="topbar-right">
    <span class="topbar-sync" id="syncInfo">Loading...</span>
    <button class="topbar-btn" onclick="syncMetaNow()" id="metaSyncBtn" title="Fetch live Meta data now" style="display:none">⟳ Meta</button>
    <button class="topbar-btn" onclick="openSettings()">⚙ Settings</button>
  </div>
```

Replace with:
```html
  <div class="topbar-right">
    <span id="topbarUserBadge" style="display:none" class="admin-badge">ADMIN</span>
    <span class="topbar-sync" id="syncInfo">Loading...</span>
    <button class="topbar-btn" onclick="syncMetaNow()" id="metaSyncBtn" title="Fetch live Meta data now" style="display:none">⟳ Meta</button>
    <button class="topbar-btn" onclick="openSettings()">⚙ Settings</button>
  </div>
```

- [ ] **Step 2: Add sidebar footer** — at line 623, the sidebar closes with `</nav>`. Insert the footer before the closing tag:

Current:
```html
  </div>

</nav>
```
(The last nav-group closing div + nav close, lines ~620–623)

Replace that `</nav>` with:
```html
  </div>

  <!-- Sidebar footer — identity + switch user -->
  <div class="sidebar-footer" id="sidebarFooter" style="display:none">
    <div class="sf-user"><span class="sf-name" id="sfName">–</span> <span class="sf-role" id="sfRole"></span></div>
    <span class="sf-switch" onclick="clearUser();location.reload()">Switch user</span>
  </div>

</nav>
```

- [ ] **Step 3: Add user picker modal** — immediately after `<body>` (line 534, which is the blank line after `<body>`), insert the modal:

Current at lines 533–536:
```html
<body>

<div class="app-shell">
```

Replace with:
```html
<body>

<!-- ════════ USER PICKER MODAL ════════ -->
<div id="userPickerOv" class="up-ov hidden">
  <div class="up-box">
    <div class="up-title">Welcome to Servedia Ops</div>
    <div class="up-sub">Who are you? Your view will be personalized.</div>
    <label class="up-label" for="upSelect">Your name</label>
    <select class="up-select" id="upSelect" onchange="onUserSelectChange()">
      <option value="">— Select your name —</option>
    </select>
    <div class="up-pwrap" id="upPwrap" style="display:none">
      <label class="up-label" for="upPass">Admin password</label>
      <input class="up-pinput" type="password" id="upPass" placeholder="Enter password" oninput="document.getElementById('upPassErr').style.display='none'">
      <div class="up-perr" id="upPassErr">Incorrect password. Try again.</div>
    </div>
    <button class="up-btn" onclick="confirmUserPick()">Enter Dashboard</button>
  </div>
</div>

<div class="app-shell">
```

- [ ] **Step 4: Verify HTML is well-formed** — check the modal opened/closed correctly:

```bash
grep -n "userPickerOv\|up-box\|app-shell" index.html | head -10
```

Expected: `userPickerOv` div opens before `app-shell`, closes before it.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add user picker modal HTML + sidebar footer + topbar badge"
```

---

## Task 4: Add `TEAM_MEMBERS`, `assignmentData`, and localStorage Helpers

**Files:**
- Modify: `index.html` — after the `const DRIVE_ROOT` line at ~1400, before the `let metaData = null` state block at ~1402

- [ ] **Step 1: Add team members array and assignment data global** — insert after `const DRIVE_ROOT = '...'` line:

```js
// ══════════════════════════════════════════════════════════════
// TEAM MEMBERS — hardcoded for Phase 1, managed via Admin Panel in Phase 3
// roles: media_buyer | csm | tech | isa | video_editor | admin
// ══════════════════════════════════════════════════════════════
const TEAM_MEMBERS = [
  { name: 'Byron Vale',           role: 'admin' },
  { name: 'Felipe Guimaraes',    role: 'media_buyer' },
  { name: 'Julianne Navarro',    role: 'csm' },
  { name: 'Adriana Gonzalez',    role: 'csm' },
  { name: 'Anca Elamparo',       role: 'media_buyer' },
  { name: 'Autumn Ismirnioglou', role: 'video_editor' },
  { name: 'Saquib Shaikh',       role: 'tech' },
];

// Admin password — SHA-256 of "Servedia2026!" — never store plain text
const ADMIN_PW_HASH = '96864b8317c1514456bfeff42cdb337dad97e4de679e78f82559a30ac63cfdbd';

// Assignment data — populated from data/assignments.json on loadAll()
let assignmentData = null;
```

- [ ] **Step 2: Add localStorage helpers** — insert after the `ADMIN_PW_HASH` and `assignmentData` lines, before the existing `let metaData = null` line:

```js
// ── User identity helpers ──
function getUser() {
  try { return JSON.parse(localStorage.getItem('servedia_user') || 'null'); } catch { return null; }
}
function setUser(obj) {
  localStorage.setItem('servedia_user', JSON.stringify(obj));
}
function clearUser() {
  localStorage.removeItem('servedia_user');
}
```

- [ ] **Step 3: Verify the constants section looks right**

Run a grep to confirm the new symbols exist and aren't duplicated:

```bash
grep -n "TEAM_MEMBERS\|ADMIN_PW_HASH\|assignmentData\|function getUser\|function setUser\|function clearUser" index.html
```

Expected: each appears exactly once, all within a few lines of each other.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add TEAM_MEMBERS array, ADMIN_PW_HASH, assignmentData, user helpers"
```

---

## Task 5: Add `showUserPicker()`, `checkAdminPassword()`, and `confirmUserPick()`

**Files:**
- Modify: `index.html` — insert new functions just before the existing `function getCfg()` at line 1573

- [ ] **Step 1: Add the three functions** — insert just before `function getCfg()`:

```js
// ══════════════════════════════════════════════════════════════
// USER PICKER — modal control + SHA-256 password check
// ══════════════════════════════════════════════════════════════
function showUserPicker() {
  const sel = document.getElementById('upSelect');
  // Populate dropdown from TEAM_MEMBERS
  sel.innerHTML = '<option value="">— Select your name —</option>' +
    TEAM_MEMBERS.map(m => `<option value="${m.name}">${m.name}</option>`).join('');
  document.getElementById('upPwrap').style.display = 'none';
  document.getElementById('upPassErr').style.display = 'none';
  document.getElementById('upPass').value = '';
  document.getElementById('userPickerOv').classList.remove('hidden');
}

function onUserSelectChange() {
  const sel = document.getElementById('upSelect');
  const member = TEAM_MEMBERS.find(m => m.name === sel.value);
  document.getElementById('upPwrap').style.display =
    (member && member.role === 'admin') ? 'block' : 'none';
  document.getElementById('upPassErr').style.display = 'none';
}

async function checkAdminPassword(pass) {
  const enc = new TextEncoder();
  const buf = await crypto.subtle.digest('SHA-256', enc.encode(pass));
  const hex = Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('');
  return hex === ADMIN_PW_HASH;
}

async function confirmUserPick() {
  const sel = document.getElementById('upSelect');
  const name = sel.value;
  if (!name) return;
  const member = TEAM_MEMBERS.find(m => m.name === name);
  if (!member) return;

  if (member.role === 'admin') {
    const pass = document.getElementById('upPass').value;
    const ok = await checkAdminPassword(pass);
    if (!ok) {
      document.getElementById('upPassErr').style.display = 'block';
      return;
    }
  }

  setUser({ name: member.name, role: member.role, isAdmin: member.role === 'admin' });
  document.getElementById('userPickerOv').classList.add('hidden');
  applyRoleVisibility(member.role);
}
```

- [ ] **Step 2: Verify the functions exist and reference correct element IDs**

```bash
grep -n "function showUserPicker\|function onUserSelectChange\|function checkAdminPassword\|function confirmUserPick" index.html
```

Expected: 4 matches, all consecutive, before `function getCfg`.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add user picker modal logic with SHA-256 admin password check"
```

---

## Task 6: Add `applyRoleVisibility()` and `checkUser()`

**Files:**
- Modify: `index.html` — insert two functions just before `function getCfg()` (after the functions from Task 5)

- [ ] **Step 1: Add `applyRoleVisibility()`** — the role-to-nav mapping comes directly from the spec. Insert after `confirmUserPick()`:

```js
function applyRoleVisibility(role) {
  // Nav IDs → roles that can see them
  const ROLE_NAV = {
    'nav-meta':        ['media_buyer', 'admin'],
    'nav-maintenance': ['media_buyer', 'admin'],
    'nav-retention':   ['media_buyer', 'admin'],
    'nav-creative':    ['media_buyer', 'admin'],
    'nav-crm':         ['csm', 'admin'],
    'nav-onboarding':  ['csm', 'tech', 'admin'],
    'nav-watchdog':    ['csm', 'admin'],
    'nav-video':       ['video_editor', 'media_buyer', 'admin'],
    'nav-tech':        ['tech', 'admin'],
    'nav-slack':       ['csm', 'admin'],
    'nav-tasks':       ['media_buyer', 'csm', 'tech', 'isa', 'admin'],
  };

  // Show/hide each nav item
  for (const [navId, allowedRoles] of Object.entries(ROLE_NAV)) {
    const el = document.getElementById(navId);
    if (!el) continue;
    // Hide the parent nav-group if ALL its children are hidden
    el.style.display = allowedRoles.includes(role) ? '' : 'none';
  }

  // Hide entire nav-groups whose label children are all hidden
  document.querySelectorAll('.nav-group').forEach(group => {
    const items = group.querySelectorAll('.nav-item');
    const allHidden = Array.from(items).every(el => el.style.display === 'none');
    group.style.display = allHidden ? 'none' : '';
  });

  // Topbar admin badge
  const badge = document.getElementById('topbarUserBadge');
  if (badge) badge.style.display = role === 'admin' ? '' : 'none';

  // Sidebar footer
  const user = getUser();
  if (user) {
    const footer = document.getElementById('sidebarFooter');
    if (footer) {
      footer.style.display = '';
      document.getElementById('sfName').textContent = user.name;
      document.getElementById('sfRole').textContent = '· ' + user.role.replace('_', ' ');
    }
  }
}
```

- [ ] **Step 2: Add `checkUser()`** — insert immediately after `applyRoleVisibility()`:

```js
function checkUser() {
  const user = getUser();
  if (!user || !user.name || !user.role) {
    showUserPicker();
    return;
  }
  applyRoleVisibility(user.role);
}
```

- [ ] **Step 3: Verify**

```bash
grep -n "function applyRoleVisibility\|function checkUser" index.html
```

Expected: both match exactly once, after `confirmUserPick`.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add applyRoleVisibility and checkUser for role-based sidebar"
```

---

## Task 7: Wire `checkUser()` into `DOMContentLoaded`

**Files:**
- Modify: `index.html:5368` — the `DOMContentLoaded` block

- [ ] **Step 1: Modify DOMContentLoaded** — current code at lines 5368–5378:

```js
document.addEventListener('DOMContentLoaded', () => {
  switchSection('command');
  renderTasks();   // show loading placeholder immediately
  renderVideo();   // show loading placeholder immediately
  // Show ⟳ Meta button if token is saved
  if (localStorage.getItem('meta_access_token')) {
    const btn = document.getElementById('metaSyncBtn');
    if (btn) btn.style.display = '';
  }
  loadAll();
});
```

Replace with:

```js
document.addEventListener('DOMContentLoaded', () => {
  checkUser();     // show picker if no identity, apply role visibility if returning user
  switchSection('command');
  renderTasks();   // show loading placeholder immediately
  renderVideo();   // show loading placeholder immediately
  // Show ⟳ Meta button if token is saved
  if (localStorage.getItem('meta_access_token')) {
    const btn = document.getElementById('metaSyncBtn');
    if (btn) btn.style.display = '';
  }
  loadAll();
});
```

- [ ] **Step 2: Verify the call is first**

```bash
grep -n -A 3 "DOMContentLoaded" index.html | tail -20
```

Expected: `checkUser()` is the first call inside the handler.

- [ ] **Step 3: Manual smoke test in browser**

Open `index.html` locally. Expected behavior:
1. Modal appears on first load — "Welcome to Servedia Ops — Who are you?"
2. Dropdown shows all 7 team members
3. Selecting "Byron Vale" shows a password field
4. Selecting any non-admin name hides the password field
5. Picking "Felipe Guimaraes" + clicking "Enter Dashboard" stores user in localStorage and hides the modal
6. Sidebar shows only Media Buying + Video Tickets + Tasks (nav-crm, nav-tech, nav-slack hidden)
7. Sidebar footer shows "Felipe Guimaraes · media buyer" + "Switch user" link
8. Reloading the page skips the modal and goes directly to the dashboard
9. Clicking "Switch user" clears localStorage and shows the picker again
10. Picking "Byron Vale" with wrong password shows error message; with correct password "Servedia2026!" enters admin mode, badge shows in topbar

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: call checkUser on DOMContentLoaded — Phase 1 user system wired"
```

---

## Task 8: Add `assignments.json` to `loadAll()` Static Fetches

**Files:**
- Modify: `index.html:4161` — `loadAll()` function

- [ ] **Step 1: Add fetch to staticFetches array** — current last entry in the static fetches array is index 9 (`watchdog.json`):

```js
      fetchJ(`${BASE}/watchdog.json?_=${ts}`),        // 9 — 30-day watchdog (F13)
```

Add index 10 immediately after:

```js
      fetchJ(`${BASE}/watchdog.json?_=${ts}`),        // 9 — 30-day watchdog (F13)
      fetchJ(`${BASE}/assignments.json?_=${ts}`),     // 10 — account assignments (Phase 1)
```

- [ ] **Step 2: Handle the result** — after the existing `staticFetches[9]` result handler:

```js
    if (staticFetches[9].status === 'fulfilled' && staticFetches[9].value) { watchdogData      = staticFetches[9].value; }
```

Add immediately after:

```js
    if (staticFetches[10].status === 'fulfilled' && staticFetches[10].value) {
      assignmentData = staticFetches[10].value;
      window.assignmentData = assignmentData; // T4 actions also read from window.assignmentData
    }
```

- [ ] **Step 3: Verify**

```bash
grep -n "assignments.json\|staticFetches\[10\]" index.html
```

Expected: the fetch at index 10 and the result handler both appear.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: load assignments.json in loadAll static fetches"
```

---

## Task 9: Add Meta Assignment Filter Toggle to `renderMeta()`

**Files:**
- Modify: `index.html:2652` — `renderMeta()` function
- Modify: `index.html:638` — Meta Ads panel filters HTML

- [ ] **Step 1: Add toggle button HTML to Meta Ads panel** — inside `<div class="filters">` at line 638, add a toggle button after the existing sort select (before the `+ New Client` button):

Current code at ~line 650:
```html
    <button style="background:#7c3aed;border:none;color:#fff;padding:7px 14px;border-radius:8px;cursor:pointer;font-size:12px;font-weight:700;white-space:nowrap" onclick="openOnboardingModal()">+ New Client</button>
```

Add immediately before it:
```html
    <button class="fbtn" id="metaAssignToggle" onclick="toggleMetaAssign()" style="display:none">👤 My accounts</button>
```

- [ ] **Step 2: Add `toggleMetaAssign()` helper and `metaShowMine` flag** — add after `let assignmentData = null;` (from Task 4, near line 1402):

```js
let metaShowMine = false; // toggled per-session; default depends on role
```

And add the toggle function just before `function renderMeta()` at line 2652:

```js
function toggleMetaAssign() {
  metaShowMine = !metaShowMine;
  const btn = document.getElementById('metaAssignToggle');
  if (btn) btn.classList.toggle('active', metaShowMine);
  renderMeta();
}
```

- [ ] **Step 3: Add assignment filter logic inside `renderMeta()`** — at the top of `renderMeta()`, after `if (!metaData) return;`, add:

Current:
```js
function renderMeta() {
  if (!metaData) return;
  const clients = metaData.clients || [];
```

Replace with:
```js
function renderMeta() {
  if (!metaData) return;
  const clients = metaData.clients || [];

  // Assignment filter — show/configure toggle based on current user
  const user = getUser();
  const toggleBtn = document.getElementById('metaAssignToggle');
  if (toggleBtn) {
    if (user && user.role === 'media_buyer' && assignmentData) {
      toggleBtn.style.display = '';
      // Default to "my accounts" for media buyers on first render
      if (!toggleBtn.dataset.initialized) {
        metaShowMine = true;
        toggleBtn.classList.add('active');
        toggleBtn.dataset.initialized = '1';
      }
    } else {
      toggleBtn.style.display = 'none';
      metaShowMine = false;
    }
  }
```

- [ ] **Step 4: Add the filter to the `list` build** — in `renderMeta()`, the `let list = clients.filter(...)` block currently looks like (lines ~2659–2664):

```js
  let list = clients.filter(c => {
    if (search && !(c.account_name || '').toLowerCase().includes(search)) return false;
    if (filt === 'active'  && c.status !== 'active') return false;
    if (filt === 'no_data' && c.status === 'active') return false;
    return true;
  });
```

Replace with:
```js
  // Build assigned account name set for this user (case-insensitive)
  let myAccountNames = null;
  if (metaShowMine && user && assignmentData) {
    myAccountNames = new Set(
      (assignmentData.assignments || [])
        .filter(a => a.media_buyer === user.name)
        .map(a => (a.client_name || '').toLowerCase())
    );
  }

  let list = clients.filter(c => {
    if (search && !(c.account_name || '').toLowerCase().includes(search)) return false;
    if (filt === 'active'  && c.status !== 'active') return false;
    if (filt === 'no_data' && c.status === 'active') return false;
    if (myAccountNames !== null) {
      const nameLC = (c.account_name || '').toLowerCase();
      if (!Array.from(myAccountNames).some(n => nameLC.includes(n) || n.includes(nameLC))) return false;
    }
    return true;
  });
```

- [ ] **Step 5: Update toggle button label to show count** — after the `list` is built and before the sort block, update the toggle button text:

```js
  if (toggleBtn && toggleBtn.style.display !== 'none') {
    const myCount = metaShowMine ? list.length : (assignmentData
      ? (assignmentData.assignments || []).filter(a => user && a.media_buyer === user.name).length
      : 0);
    toggleBtn.textContent = metaShowMine
      ? `👤 My accounts (${myCount})`
      : `👁 All accounts (${list.length})`;
  }
```

- [ ] **Step 6: Manual smoke test**

As Felipe Guimaraes (media_buyer), open Meta Ads section. Expected:
- Toggle button "👤 My accounts (N)" visible, active (purple), showing only Felipe's assigned accounts
- Click toggle → button shows "👁 All accounts (37)", all accounts visible
- As Byron Vale (admin), toggle button is hidden, all 37 accounts show

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: add Meta Ads assignment filter toggle for media buyers"
```

---

## Task 10: Final Smoke Test + Push

**Files:**
- None new — verification only

- [ ] **Step 1: Check all 7 roles manually**

Open the dashboard in a browser (local or GitHub Pages). Test each user:

| User | Expected visible nav items |
|------|---------------------------|
| Felipe Guimaraes (media_buyer) | Command, Meta Ads, Daily Maintenance, Retention Risk, Creative Intel, Video Tickets, Tasks |
| Adriana Gonzalez (csm) | Command, CRM, Onboarding, Watchdog, Slack, Tasks |
| Saquib Shaikh (tech) | Command, Tech Tickets, Onboarding, Tasks |
| Autumn Ismirnioglou (video_editor) | Command, Video Tickets |
| Byron Vale (admin, pw: Servedia2026!) | All items + topbar ADMIN badge |

- [ ] **Step 2: Check "Switch user" flow**

1. Log in as Felipe
2. Click "Switch user" in sidebar footer
3. Modal appears again
4. Pick Byron Vale, enter correct password
5. All nav items visible, ADMIN badge shows in topbar

- [ ] **Step 3: Check localStorage shape**

Open browser DevTools → Application → localStorage. Key `servedia_user` should contain:
```json
{"name":"Byron Vale","role":"admin","isAdmin":true}
```

- [ ] **Step 4: Push to GitHub Pages**

```bash
git push origin main
```

Wait ~30 seconds, then verify at `https://artzy22.github.io/servedia-ops/`.

---

*Servedia Ops — Phase 1 Multi-Role User System Plan*
*Generated: 2026-04-14*
