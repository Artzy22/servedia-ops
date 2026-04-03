# Servedia Ops Center

Live dashboard pulling ClickUp tasks, Slack messages, and Meta KPIs. Hosted free on GitHub Pages, auto-updated by GitHub Actions every 30 minutes.

## Setup (10 minutes)

### Step 1 — Create GitHub repo
1. Go to github.com → New repository
2. Name it `servedia-ops`
3. Set to **Public** (required for free GitHub Pages)
4. Click Create

### Step 2 — Upload all files
Upload everything in this folder to the repo (drag and drop in GitHub UI):
- `index.html`
- `data/clickup.json`
- `data/slack.json`
- `data/meta.json`
- `.github/workflows/sync-clickup.yml`
- `.github/workflows/sync-slack.yml`
- `.github/workflows/sync-meta.yml`

### Step 3 — Add secrets (your API keys)
Go to repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add these 3 secrets:

| Secret Name | Value |
|---|---|
| `CLICKUP_API_KEY` | `pk_204101467_LKBEMU0EWTOE6UI7SN6M01W6L6PQYUY8` |
| `SLACK_BOT_TOKEN` | `xoxb-6845545681958-10842946009650-i5025G2ELlbOKZoJnJ8l8Uvt` |
| `META_ACCESS_TOKEN` | `EAANnkQyihrMBREDhA3GIpLJBJnXGogg...` (your full token) |

### Step 4 — Enable GitHub Pages
Go to repo → **Settings** → **Pages**
- Source: **Deploy from a branch**
- Branch: **main** / **/ (root)**
- Click Save

Your site will be live at:
`https://YOUR_USERNAME.github.io/servedia-ops`

### Step 5 — Update index.html with your repo name
In `index.html`, find this line near the bottom:
```javascript
const REPO = 'YOUR_GITHUB_USERNAME/servedia-ops';
```
Change it to your actual GitHub username, e.g.:
```javascript
const REPO = 'byronvale/servedia-ops';
```

### Step 6 — Run Actions manually first time
Go to repo → **Actions** tab
- Click each workflow (Sync ClickUp, Sync Slack, Sync Meta)
- Click **Run workflow** to populate data immediately
- After that they run automatically every 30 min

## Adding more Slack channels

In `.github/workflows/sync-slack.yml`, find the channels array and add more:
```javascript
const channels = [
  { id: 'C0AQGQ021AQ', name: '#flagadset' },
  { id: 'C0AQGPX58MA', name: '#summary' },
  { id: 'YOUR_CHANNEL_ID', name: '#your-channel' },  // add here
];
```

To get a channel ID: open Slack in browser → click the channel → copy the ID from the URL (starts with C)

## Schedule
- ClickUp: every 30 minutes
- Slack: every 30 minutes  
- Meta KPIs: every 2 hours (Meta API is slower)
