# Servedia Ops — Parallel Agent Dispatch Hub

> **How this works:** Each agent file is a complete, self-contained prompt for a separate Claude Code terminal.
> Each agent gets its own git worktree (isolated branch) so they can't conflict on `index.html`.
> After all agents finish, merge branches in order.

---

## Current Build Status

| # | Agent File | Task | Branch | Done? |
|---|-----------|------|--------|-------|
| 1 | `agent-1-switchsection.md` | Replace `switchTab()` with `switchSection()` + fix boot call | `feat/agent1-switchsection` | [ ] |
| 2 | `agent-2-command-panel.md` | Replace `panel-briefing` HTML with `panel-command` | `feat/agent2-command-panel` | [ ] |
| 3 | `agent-3-render-functions.md` | Write `renderCommandCenter()` + `updateNavBadges()` + wire into `loadAll()` | `feat/agent3-render-fns` | [ ] |
| 4 | `agent-4-stub-panels.md` | Add HTML stubs: CRM, Onboarding, Watchdog, Tech Tickets panels | `feat/agent4-stubs` | [ ] |

**Agents 1–4 are independent — run all 4 terminals at the same time.**

---

## Step 1: One-Time Worktree Setup

Run this once from your main repo before starting agents:

```bash
cd D:/GitHub/servedia-ops

# Create 4 isolated worktrees, one per agent
git worktree add ../servedia-ops-a1 -b feat/agent1-switchsection
git worktree add ../servedia-ops-a2 -b feat/agent2-command-panel
git worktree add ../servedia-ops-a3 -b feat/agent3-render-fns
git worktree add ../servedia-ops-a4 -b feat/agent4-stubs
```

---

## Step 2: Start 4 Terminals

| Terminal | Working Directory | Agent Prompt |
|----------|------------------|-------------|
| 1 | `D:/GitHub/servedia-ops-a1` | Copy prompt from `agent-1-switchsection.md` |
| 2 | `D:/GitHub/servedia-ops-a2` | Copy prompt from `agent-2-command-panel.md` |
| 3 | `D:/GitHub/servedia-ops-a3` | Copy prompt from `agent-3-render-functions.md` |
| 4 | `D:/GitHub/servedia-ops-a4` | Copy prompt from `agent-4-stub-panels.md` |

In each terminal:
```bash
cd D:/GitHub/servedia-ops-a[N]
claude
# then paste the prompt from the agent file
```

---

## Step 3: Integration (after all 4 finish)

```bash
cd D:/GitHub/servedia-ops

# Merge in order — conflicts unlikely since agents own different file regions
git merge feat/agent1-switchsection --no-ff -m "feat: switchSection nav function + boot call"
git merge feat/agent2-command-panel --no-ff -m "feat: panel-command replaces panel-briefing"
git merge feat/agent3-render-fns    --no-ff -m "feat: renderCommandCenter + updateNavBadges"
git merge feat/agent4-stubs         --no-ff -m "feat: stub panels for CRM/Onboarding/Watchdog/Tech"

# Verify in browser — open index.html, confirm:
# - Sidebar works, Command Center loads by default
# - All nav items clickable without JS errors
# - Badges populate after data loads

# Clean up worktrees
git worktree remove ../servedia-ops-a1
git worktree remove ../servedia-ops-a2
git worktree remove ../servedia-ops-a3
git worktree remove ../servedia-ops-a4
```

If there's a merge conflict — it will be in `index.html`. Each agent owns a different region:
- Agent 1: JS lines ~1418–1424 (function) + ~3257 (boot)
- Agent 2: HTML lines ~614–648 (panel-briefing block)
- Agent 3: JS ~2620–2700 (loadAll + new functions above it)
- Agent 4: HTML end of panels section

---

## Adding Future Tasks

1. Copy any agent file as a template
2. Update the task, search strings, and branch name
3. Add a row to the table above
4. Follow the same worktree workflow
