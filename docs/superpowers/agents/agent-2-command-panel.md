# Agent 2 — Command Center Panel HTML

> **Working directory for this agent:** `D:/GitHub/servedia-ops-a2`
> **Branch:** `feat/agent2-command-panel`
> **Owns:** HTML panel-briefing block (~line 614–648)
> **Do NOT touch:** CSS, JS functions, sidebar HTML, any other panels

---

## Paste this prompt into Claude Code

```
You are implementing Task 2 of a 4-agent parallel build on the Servedia Ops dashboard.

PROJECT CONTEXT:
- Single file app: D:/GitHub/servedia-ops-a2/index.html (~3272 lines, vanilla HTML/CSS/JS)
- The sidebar is already built with a "Command Center" nav item: id="nav-command"
- The default view should be a Command Center panel (id="panel-command")
- Currently the old "Morning Briefing" panel (id="panel-briefing") is still in the file as the active panel.
  It must be replaced with a Command Center panel.
- renderCommandCenter() JS will be added by a separate agent — your panel just needs the right DOM structure.

EXISTING CSS you can use (already in the file):
- .stat-grid / .stat-card / .stat-label / .stat-value (.purple/.green/.blue/.yellow/.red/.orange)
- .section-title / .no-data
- .kbadge .kb-green/.kb-red/.kb-orange/.kb-yellow/.kb-gray
- .meta-grid (card grid, minmax 380px)
- .content (padding: 20px 24px)
- @keyframes ccPulse already defined

YOUR TASK (and ONLY your task):

Find and replace the entire panel-briefing block. It starts at:
```
<!-- ════════════════ MORNING BRIEFING ════════════════ -->
<div id="panel-briefing" class="panel active content">
```
And ends at the closing `</div>` of that panel (before `<div id="panel-maintenance"`).

Replace the ENTIRE panel-briefing block with this panel-command block:

```html
<!-- ════════════════ COMMAND CENTER ════════════════ -->
<div id="panel-command" class="panel active content">

  <!-- Header row -->
  <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:20px;flex-wrap:wrap;gap:12px">
    <div>
      <div style="font-size:22px;font-weight:900;color:#fff">⚡ Command Center</div>
      <div style="font-size:12px;color:#444;margin-top:3px" id="ccDate"></div>
    </div>
    <div style="display:flex;align-items:center;gap:6px;font-size:11px;color:#34d399;background:#052e16;border:1px solid #065f46;padding:5px 14px;border-radius:20px">
      <span style="width:6px;height:6px;border-radius:50%;background:#34d399;display:inline-block;animation:ccPulse 2s infinite"></span>
      Live Dashboard
    </div>
  </div>

  <!-- Top stat row — populated by renderCommandCenter() -->
  <div class="stat-grid" id="ccStatGrid">
    <div class="stat-card"><div class="stat-label">Active Accounts</div><div class="stat-value purple" id="ccActiveAccounts">–</div></div>
    <div class="stat-card"><div class="stat-label">7-Day Spend</div><div class="stat-value green" id="ccSpend7">–</div></div>
    <div class="stat-card"><div class="stat-label">Clients to Review</div><div class="stat-value orange" id="ccReviewCount">–</div></div>
    <div class="stat-card"><div class="stat-label">Video Tickets</div><div class="stat-value purple" id="ccVideoCount">–</div></div>
    <div class="stat-card"><div class="stat-label">Urgent Tasks</div><div class="stat-value red" id="ccUrgentCount">–</div></div>
    <div class="stat-card"><div class="stat-label">Slack Alerts</div><div class="stat-value yellow" id="ccSlackCount">–</div></div>
  </div>

  <!-- Fires section — needs attention now -->
  <div id="ccFireSection" style="margin-bottom:24px">
    <div class="section-title" style="color:#f87171;margin-bottom:10px">🔥 NEEDS ATTENTION NOW</div>
    <div id="ccFireGrid" class="meta-grid" style="grid-template-columns:repeat(auto-fill,minmax(320px,1fr))">
      <div class="no-data">Loading...</div>
    </div>
  </div>

  <!-- Video tickets ready -->
  <div id="ccVideoSection" style="margin-bottom:24px">
    <div class="section-title" style="color:#fb923c;margin-bottom:10px">🎬 VIDEO TICKETS — NEEDS YOUR REVIEW</div>
    <div id="ccVideoList"><div class="no-data">Loading...</div></div>
  </div>

  <!-- Urgent tasks -->
  <div id="ccTaskSection" style="margin-bottom:24px">
    <div class="section-title" style="color:#fbbf24;margin-bottom:10px">☑️ URGENT &amp; OVERDUE TASKS</div>
    <div id="ccTaskList"><div class="no-data">Loading...</div></div>
  </div>

  <!-- Slack alerts -->
  <div id="ccSlackSection" style="margin-bottom:24px">
    <div class="section-title" style="color:#a78bfa;margin-bottom:10px">💬 SLACK — NEEDS YOUR RESPONSE</div>
    <div id="ccSlackList"><div class="no-data">Loading...</div></div>
  </div>

  <!-- All clear state -->
  <div id="ccAllClear" style="display:none;text-align:center;padding:40px">
    <div style="font-size:40px;margin-bottom:12px">🎉</div>
    <div style="font-size:18px;font-weight:700;color:#34d399">All clear! Nothing urgent right now.</div>
    <div style="font-size:13px;color:#555;margin-top:6px">Head to Daily Maintenance to start your reviews.</div>
  </div>

</div>
```

IMPORTANT: The comment `<!-- ════════════════ DAILY MAINTENANCE ════════════════ -->` appears just ABOVE the panel-briefing block. Keep it. Your replacement should not remove that comment.

CONSTRAINTS:
- Only replace the panel-briefing div. Do not touch any other panel.
- Do not change CSS, JS, or sidebar HTML.
- The new panel must have class="panel active content" initially so it shows on load
  (switchSection will handle toggling later).

VERIFICATION:
After editing, search the file for "panel-briefing" — it should return ZERO results.
Search for "panel-command" — should return at least 1 result (the new panel div).
Search for "ccDate" — should return exactly 1 result inside the new panel.

COMMIT when done:
git add index.html
git commit -m "feat: replace panel-briefing with panel-command Command Center panel"

Report back: confirm panel-briefing is gone and panel-command exists with all section IDs.
```
