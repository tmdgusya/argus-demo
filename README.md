# Argus

Real-time monitoring and management dashboard for [Hermes Agent](https://github.com/NousResearch/hermes-agent) profiles.

## Environment Setup (Step 1)

This section is the minimum setup required before creating `backend/frontend/dba` profiles.
Follow the steps in order. Do not skip verification commands.

### 1.1 Clone the Repository

```bash
mkdir -p ~/projects
cd ~/projects
git clone <YOUR_ARGUS_REPO_URL> argus
cd argus
```

If the repo is already cloned, just run:

```bash
cd ~/projects/argus
```

### 1.2 Install Required Tools

Required versions:
- Hermes Agent CLI installed and executable
- GitHub CLI (`gh`) authenticated
- Python `3.11+`
- Node.js `20+`

Install Hermes Agent first (official quickstart installer):

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Install Python + GitHub CLI:

macOS (Homebrew)
```bash
brew install python@3.11 gh
```

Ubuntu/Debian
```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip gh curl
```

Install Node.js 20+ (recommended: nvm, works reliably on both macOS/Linux):

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
nvm install 20
nvm use 20
```

### 1.3 Authenticate GitHub CLI

```bash
gh auth login
gh auth status
```

`gh auth status` must show an authenticated account.

### 1.4 Verify Versions and Commands

Run all checks:

```bash
hermes --version
gh --version
gh auth status
python3 --version
node --version
npm --version
```

Pass criteria:
- `hermes --version` prints a version (no "command not found")
- `gh auth status` shows "Logged in"
- `python3 --version` is `3.11` or higher
- `node --version` is `v20` or higher

### 1.5 Quick Preflight Script (Optional)

Run this to check everything in one go:

```bash
set -e
command -v hermes >/dev/null && echo "[OK] hermes installed" || echo "[FAIL] hermes missing"
command -v gh >/dev/null && echo "[OK] gh installed" || echo "[FAIL] gh missing"
command -v python3 >/dev/null && echo "[OK] python3 installed" || echo "[FAIL] python3 missing"
command -v node >/dev/null && echo "[OK] node installed" || echo "[FAIL] node missing"
echo "---- versions ----"
hermes --version || true
gh --version || true
python3 --version || true
node --version || true
npm --version || true
```

### 1.6 Next Step

Once all checks pass, continue with profile/channel/token setup:
- Full team setup guide: [SETUP.md](./SETUP.md)
- Role-specific guides:
  - [backend/SETUP.md](./backend/SETUP.md)
  - [frontend/SETUP.md](./frontend/SETUP.md)
  - [dba/SETUP.md](./dba/SETUP.md)

## Profile Setup (Step 2)

This step creates three isolated Hermes profiles for Argus roles:
- `backend`
- `frontend`
- `dba`

Each profile keeps its own `config`, `.env`, sessions, and skills.

### 2.1 Check Current Profiles

```bash
hermes profile list
```

You should see at least `default`.

### 2.2 Create Profiles (Official Command Pattern)

Use `--clone --clone-from default` so all students start from the same known source profile:

```bash
hermes profile create backend --clone --clone-from default
hermes profile create frontend --clone --clone-from default
hermes profile create dba --clone --clone-from default
```

Why this exact form:
- `--clone` copies `config.yaml`, `.env`, and `SOUL.md`.
- `--clone-from default` avoids accidental cloning from a different currently active profile.

### 2.3 Verify Profiles

```bash
hermes profile list
hermes profile show backend
hermes profile show frontend
hermes profile show dba
```

Expected:
- All three profiles appear in `hermes profile list`
- `hermes profile show <name>` shows valid profile paths under `~/.hermes/profiles/`

### 2.4 Verify/Regenerate Profile Aliases

Hermes usually creates wrapper aliases automatically (for example `~/.local/bin/backend`).
If a command is missing, regenerate explicitly:

```bash
hermes profile alias backend
hermes profile alias frontend
hermes profile alias dba
```

Then test:

```bash
backend chat
frontend chat
dba chat
```

If alias commands are not found, make sure `~/.local/bin` is in `PATH`:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### 2.5 Pairing Check (Important for Team Messaging)

`pairing` is not required to *create profiles*, but it is important when operating bots in Telegram/Slack with multiple teammates.

Recommended team behavior from Hermes docs:
- Keep allowlist strict, or
- Use DM pairing flow (recommended for teams)

Core pairing commands:

```bash
hermes pairing list
hermes pairing approve telegram <PAIRING_CODE>
hermes pairing revoke telegram <USER_ID>
hermes pairing clear-pending
```

Notes:
- Pairing is used when unknown users DM the bot and `unauthorized_dm_behavior` is `pair` (default behavior).
- Do not enable open access (`ALLOW_ALL_USERS`) on bots with terminal/file tools.

### 2.6 Next Step

After Step 2, continue to platform token/channel setup and role-specific SOUL/skills:
- [SETUP.md](./SETUP.md)
- [backend/SETUP.md](./backend/SETUP.md)
- [frontend/SETUP.md](./frontend/SETUP.md)
- [dba/SETUP.md](./dba/SETUP.md)

## Official References

- Quickstart: <https://hermes-agent.nousresearch.com/docs/getting-started/quickstart/>
- CLI Commands: <https://hermes-agent.nousresearch.com/docs/reference/cli-commands/>
- Profile Commands: <https://hermes-agent.nousresearch.com/docs/reference/profile-commands/>
- Security (DM pairing): <https://hermes-agent.nousresearch.com/docs/user-guide/security/>
- Telegram setup: <https://hermes-agent.nousresearch.com/docs/user-guide/messaging/telegram/>

## What It Does

Hermes Agent supports multiple **independent profiles** — each with its own config, sessions, memory, gateway, credentials, and logs. Argus watches them all from a single web UI.

```
Web UI
├── Profile Selector
│   ├── default   → ~/.hermes/
│   ├── backend   → ~/.hermes/profiles/backend/
│   ├── frontend  → ~/.hermes/profiles/frontend/
│   └── ...       → dynamically discovered
│
├── Per-Profile Dashboard
│   ├── Status    — gateway alive/stopped, active sessions, model info
│   ├── Activity  — live message stream, current tool calls in progress
│   ├── Cost      — token usage, estimated $, cache hit ratio
│   ├── Tools     — usage distribution, call frequency
│   └── Auth      — credential pool health per provider
│
└── Cross-Profile Overview
    ├── total cost across all profiles
    ├── token consumption comparison
    └── activity heatmap by profile
```

## How It Works

Argus reads Hermes internal data sources directly — no API, no plugin, no agent modification required.

| Source | What We Read |
|--------|-------------|
| `~/.hermes/state.db` (SQLite) | sessions, messages, tool calls, token usage, costs |
| `~/.hermes/profiles/*/state.db` | same, per profile |
| `memory-decay/memories.db` | memory count, activation history, decay state |
| `gateway_state.json` + `gateway.pid` | process alive check, platform connection status |
| `auth.json` | credential pool status per provider |
| `config.yaml` | model, provider, compression, security settings |
| `logs/gateway.log` | inbound messages, response times, delivery events |
| `logs/errors.log` | error patterns, stack traces, rate limits |
| `cron/jobs.json` | scheduled job status and execution history |

### Data Flow

```
Hermes Profiles
  │
  │  (read-only, WAL-safe concurrent access)
  ▼
Argus Collector (polls every 3-5s)
  │
  ▼
argus.db (SQLite — aggregated data)
  │
  ▼
Web UI (served via built-in HTTP server)
```

## Data Sources (Full Inventory)

See [DATA-INVENTORY.md](./DATA-INVENTORY.md) for the complete breakdown of every Hermes data source, table schemas, column types, and monitoring priority.

## Student Policy Docs (Demo)

For lecture/demo operation guides and copy-paste policy prompts:

- [policies/00-INDEX.md](./policies/00-INDEX.md)
- [policies/01-LECTURE-ORCHESTRATOR.md](./policies/01-LECTURE-ORCHESTRATOR.md)
- [policies/02-BACKEND-AGENT-POLICY.md](./policies/02-BACKEND-AGENT-POLICY.md)
- [policies/03-DBA-AGENT-POLICY.md](./policies/03-DBA-AGENT-POLICY.md)
- [policies/04-FRONTEND-AGENT-POLICY.md](./policies/04-FRONTEND-AGENT-POLICY.md)
- [policies/05-HANDOFF-TEMPLATES.md](./policies/05-HANDOFF-TEMPLATES.md)
- [policies/06-LINEAR-TICKET-BACKLOG.md](./policies/06-LINEAR-TICKET-BACKLOG.md)
- [policies/07-PM-OPERATIONS.md](./policies/07-PM-OPERATIONS.md)
- [.agent/README.md](./.agent/README.md)

## Development Status

Early planning. Data inventory complete, architecture being designed.

## Tech Stack

- **Backend**: Python, SQLite (aggregation DB)
- **Frontend**: TBD
- **Data Source**: Hermes internal SQLite + JSON files (read-only)
