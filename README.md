# Argus

Real-time monitoring and management dashboard for [Hermes Agent](https://github.com/NousResearch/hermes-agent) profiles.

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

## Development Status

Early planning. Data inventory complete, architecture being designed.

## Tech Stack

- **Backend**: Python, SQLite (aggregation DB)
- **Frontend**: TBD
- **Data Source**: Hermes internal SQLite + JSON files (read-only)
