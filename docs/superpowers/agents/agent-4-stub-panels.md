# Agent 4 — Stub Panels: CRM, Onboarding, Watchdog, Tech Tickets

> **Working directory for this agent:** `D:/GitHub/servedia-ops-a4`
> **Branch:** `feat/agent4-stubs`
> **Owns:** HTML — insert new panels after the existing panel-retention block
> **Do NOT touch:** CSS, JS, sidebar HTML, any existing panels

---

## Paste this prompt into Claude Code

```
You are implementing Task 4 of a 4-agent parallel build on the Servedia Ops dashboard.

PROJECT CONTEXT:
- Single file app: D:/GitHub/servedia-ops-a4/index.html (~3272 lines, vanilla HTML/CSS/JS)
- The sidebar has nav items for: crm, onboarding, watchdog, tech
  But index.html has NO corresponding panel divs for these sections yet.
  Clicking them calls switchSection('crm') etc but there's no panel-crm to activate.
- Your job: add placeholder panels for each so the nav doesn't break. 
  These stubs will be fleshed out in future build sessions.

EXISTING CSS you can use:
  .panel / .panel.active / .content
  .no-data
  .section-title
  .stat-grid / .stat-card / .stat-label / .stat-value
  .kbadge .kb-green/.kb-red/.kb-yellow/.kb-gray/.kb-blue

FIND the insertion point — search for this line in index.html:
```
<!-- ════════════════ RETENTION ════════════════ -->
```
That marks the retention panel. Scroll to find its CLOSING </div> tag.
After that closing </div>, insert all 4 new stub panels.

INSERT these 4 panels after the retention panel's closing </div>:

```html
<!-- ════════════════ CRM ════════════════ -->
<div id="panel-crm" class="panel content">
  <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:20px">
    <div>
      <div style="font-size:22px;font-weight:900;color:#fff">🤝 CRM</div>
      <div style="font-size:12px;color:#444;margin-top:3px">Client relationship management</div>
    </div>
  </div>
  <div style="background:#111;border:1px solid #1e1e2e;border-radius:12px;padding:32px;text-align:center">
    <div style="font-size:32px;margin-bottom:12px">🤝</div>
    <div style="font-size:16px;font-weight:700;color:#a78bfa;margin-bottom:8px">CRM — Coming Soon</div>
    <div style="font-size:13px;color:#555;max-width:400px;margin:0 auto;line-height:1.6">
      Full client relationship view: contact info, happiness scores, renewal dates, deal history, and communication log — all pulled from ClickUp Client Management.
    </div>
  </div>
</div>

<!-- ════════════════ ONBOARDING ════════════════ -->
<div id="panel-onboarding" class="panel content">
  <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:20px">
    <div>
      <div style="font-size:22px;font-weight:900;color:#fff">🚀 Onboarding</div>
      <div style="font-size:12px;color:#444;margin-top:3px">New client onboarding pipeline</div>
    </div>
  </div>
  <div style="background:#111;border:1px solid #1e1e2e;border-radius:12px;padding:32px;text-align:center">
    <div style="font-size:32px;margin-bottom:12px">🚀</div>
    <div style="font-size:16px;font-weight:700;color:#60a5fa;margin-bottom:8px">Onboarding — Coming Soon</div>
    <div style="font-size:13px;color:#555;max-width:400px;margin:0 auto;line-height:1.6">
      Track new clients through onboarding stages: agreement signed, ad account access, first campaign launch, and early performance check-ins.
    </div>
  </div>
</div>

<!-- ════════════════ WATCHDOG ════════════════ -->
<div id="panel-watchdog" class="panel content">
  <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:20px">
    <div>
      <div style="font-size:22px;font-weight:900;color:#fff">🆕 Watchdog</div>
      <div style="font-size:12px;color:#444;margin-top:3px">New lead monitoring</div>
    </div>
  </div>
  <div style="background:#111;border:1px solid #1e1e2e;border-radius:12px;padding:32px;text-align:center">
    <div style="font-size:32px;margin-bottom:12px">🆕</div>
    <div style="font-size:16px;font-weight:700;color:#fb923c;margin-bottom:8px">Watchdog — Coming Soon</div>
    <div style="font-size:13px;color:#555;max-width:400px;margin:0 auto;line-height:1.6">
      Real-time alerts for new Meta leads across all accounts — so no lead goes cold. Integrates with ClickUp task creation and Slack notifications.
    </div>
  </div>
</div>

<!-- ════════════════ TECH TICKETS ════════════════ -->
<div id="panel-tech" class="panel content">
  <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:20px">
    <div>
      <div style="font-size:22px;font-weight:900;color:#fff">⚙️ Tech Tickets</div>
      <div style="font-size:12px;color:#444;margin-top:3px">Technical issues and requests</div>
    </div>
  </div>
  <div style="background:#111;border:1px solid #1e1e2e;border-radius:12px;padding:32px;text-align:center">
    <div style="font-size:32px;margin-bottom:12px">⚙️</div>
    <div style="font-size:16px;font-weight:700;color:#f87171;margin-bottom:8px">Tech Tickets — Coming Soon</div>
    <div style="font-size:13px;color:#555;max-width:400px;margin:0 auto;line-height:1.6">
      Track tech/dev requests from the team — ad account issues, pixel problems, landing page bugs, and integration requests — pulled from a dedicated ClickUp list.
    </div>
  </div>
</div>
```

HOW TO FIND THE INSERTION POINT:
1. Search for `<!-- ════════════════ RETENTION ════════════════ -->`
2. Find the `<div id="panel-retention"` that follows it
3. Locate its closing `</div>` — it's a top-level panel div, so find the </div> that closes the panel
4. Insert all 4 new panels immediately after that closing </div>

TIP: The retention panel starts with `<div id="panel-retention" class="panel content">` — 
search for `id="panel-retention"` to find it quickly.

CONSTRAINTS:
- Only add new content. Do not modify or delete any existing panels.
- Do not touch CSS, JS, or sidebar HTML.
- Each panel must have class="panel content" (without "active" — switchSection handles activation).

VERIFICATION:
Search for each panel ID — all should return exactly 1 result:
- "panel-crm" → 1 result
- "panel-onboarding" → 1 result
- "panel-watchdog" → 1 result
- "panel-tech" → 1 result

Also verify the retention panel is still intact by searching "panel-retention" — should still exist.

COMMIT when done:
git add index.html
git commit -m "feat: add stub panels for CRM, Onboarding, Watchdog, Tech Tickets"

Report back: confirm all 4 panels were inserted and the retention panel is untouched.
```
