# Agent 1 — switchSection JS Function + Boot Call

> **Working directory for this agent:** `D:/GitHub/servedia-ops-a1`
> **Branch:** `feat/agent1-switchsection`
> **Owns:** JS nav function (~line 1418) + boot call (~line 3257)
> **Do NOT touch:** HTML structure, CSS, loadAll(), renderCommandCenter(), panels

---

## Paste this prompt into Claude Code

```
You are implementing Task 1 of a 4-agent parallel build on the Servedia Ops dashboard.

PROJECT CONTEXT:
- Single file app: D:/GitHub/servedia-ops-a1/index.html (~3272 lines, vanilla HTML/CSS/JS)
- Deployed to GitHub Pages. No build step. All changes go in index.html only.
- The sidebar HTML is already built. Nav items call switchSection('x') via onclick.
- Problem: switchTab() still references old .tab DOM elements that no longer exist.
  When the app boots it calls switchTab('briefing') which crashes because there's no
  element with id="tab-briefing" anymore. This must be fixed.

YOUR TASK (and ONLY your task):

1. REPLACE the switchTab function body (around line 1418).

Find this exact block:
```
// ══════════════════════════════════════════════════════════════
// TAB NAVIGATION
// ══════════════════════════════════════════════════════════════
function switchTab(tab) {
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
  document.getElementById('tab-' + tab).classList.add('active');
  document.getElementById('panel-' + tab).classList.add('active');
  if (tab === 'retention') renderRetention();
}
```

Replace it with:
```
// ══════════════════════════════════════════════════════════════
// NAVIGATION
// ══════════════════════════════════════════════════════════════
function switchSection(section) {
  document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
  document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
  const navEl = document.getElementById('nav-' + section);
  if (navEl) navEl.classList.add('active');
  const panelEl = document.getElementById('panel-' + section);
  if (panelEl) panelEl.classList.add('active');
  if (section === 'retention') renderRetention();
  if (section === 'command') { if (typeof renderCommandCenter === 'function') renderCommandCenter(); }
}

// Alias — keeps any remaining internal switchTab() calls working
function switchTab(tab) { switchSection(tab); }
```

2. UPDATE the boot call (around line 3257).

Find this exact block:
```
document.addEventListener('DOMContentLoaded', () => {
  switchTab('briefing');
  renderTasks();   // show loading placeholder immediately
  renderVideo();   // show loading placeholder immediately
  loadAll();
});
```

Replace it with:
```
document.addEventListener('DOMContentLoaded', () => {
  switchSection('command');
  renderTasks();   // show loading placeholder immediately
  renderVideo();   // show loading placeholder immediately
  loadAll();
});
```

CONSTRAINTS:
- Only edit these two locations. Nothing else.
- Do not rename or remove the switchTab alias — other agents may depend on it during merge.
- Do not touch CSS, HTML, loadAll(), or any render functions.

VERIFICATION:
After editing, search the file for "getElementById('tab-" — there should be ZERO results
(the new switchSection uses getElementById('nav-' + section) and getElementById('panel-' + section)).
If you find any remaining "getElementById('tab-", check if it's in the new function body.

COMMIT when done:
git add index.html
git commit -m "feat: replace switchTab with switchSection, update boot call"

Report back: confirm both edits are done and the old getElementById('tab-' + ...) is gone from the function.
```
