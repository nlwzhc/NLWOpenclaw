# OpenClaw — Install & Operation Guide

## Prerequisites

- Ubuntu server (tested on kernel 6.8.0)
- Node.js v22+ via nvm
- `systemd --user` for gateway service

## Installation

```bash
# Install via npm (global)
npm install -g openclaw

# Or with nvm:
source ~/.nvm/nvm.sh && nvm use 24
npm install -g openclaw

# Verify
openclaw --version
```

## Configuration

Config lives at `~/.openclaw/openclaw.json`. Start from `setup/openclaw.json` in this repo.

### Environment file

Create `~/.openclaw/.env` with:

```env
TELEGRAM_BOT_TOKEN=<main/CEO-Benny bot token from BotFather>
TELEGRAM_BOT_TOKEN_OPS=<nova_ops bot token from BotFather>
SHOPIFY_ADMIN_TOKEN=<Shopify Admin API token>
ANTHROPIC_API_KEY=<Anthropic API key>
```

### Workspace setup

The main agent (CEO-Benny) uses `~/.openclaw/workspace/` directly. Sub-agents each have a subdirectory under `~/.openclaw/workspace/agents/<id>/`.

Copy files from this repo:

```bash
# Main workspace
cp workspace/IDENTITY.md ~/.openclaw/workspace/
cp workspace/AGENTS.md ~/.openclaw/workspace/
mkdir -p ~/.openclaw/workspace/lib/company ~/.openclaw/workspace/lib/ops
cp workspace/lib/company/IDENTITY.md ~/.openclaw/workspace/lib/company/
cp workspace/lib/ops/TOOLS.md ~/.openclaw/workspace/lib/ops/

# nova_ops agent
mkdir -p ~/.openclaw/workspace/agents/nova_ops
cp agents/nova_ops/* ~/.openclaw/workspace/agents/nova_ops/

# nova_developer agent
mkdir -p ~/.openclaw/workspace/agents/nova_developer
cp agents/nova_developer/* ~/.openclaw/workspace/agents/nova_developer/

# Repeat for other agents (nova_vision, nova_design, etc.)
```

## Gateway Service

The gateway runs as a systemd user service:

```bash
# Status
systemctl --user status openclaw-gateway.service

# Start
systemctl --user start openclaw-gateway.service

# Enable on boot
systemctl --user enable openclaw-gateway.service

# Restart
systemctl --user restart openclaw-gateway.service
```

## Running OpenClaw CLI

Always use nvm to ensure correct Node version:

```bash
source ~/.nvm/nvm.sh && nvm use 24 2>/dev/null

# Status
openclaw status --json

# List agents
openclaw agents list

# Gateway status
openclaw gateway status --json

# Config read/write
openclaw config get agents.defaults.model.primary
openclaw config set agents.defaults.model.primary "anthropic/claude-sonnet-4-6"

# Memory
openclaw memory index           # reindex vector memory
openclaw memory search "stock"  # search

# Sessions
openclaw sessions list

# Heartbeat
openclaw system heartbeat last
```

## PM2 Processes (companion apps)

| PM2 Name | ID | Description |
|---|---|---|
| `nova-dashboard` | — | NLW Dash (nova_developer manages this) |
| `nlw-dash` | 2 | Production NLW Dash |
| `nlw-dash-dev` | 6 | Dev NLW Dash |
| `nlw-planner-backend` | 3 | Production Planner |
| `nlw-planner-backend-dev` | 7 | Dev Planner |
| `nlw-stock-backend` | 0 | Production Stock backend |
| `nlw-stock-frontend` | 1 | Production Stock frontend |
| `nlw-stock-backend-dev` | 4 | Dev Stock backend |
| `nlw-stock-frontend-dev` | 5 | Dev Stock frontend |
| `nlw-portal` | 8 | Production Portal |
| `nlw-portal-dev` | 9 | Dev Portal |

## Troubleshooting

### Gateway won't start
```bash
journalctl --user -u openclaw-gateway.service -n 50
```

### Agent not responding
```bash
openclaw gateway status --json
openclaw sessions list
```

### Memory not finding recent context
```bash
openclaw memory index
```

### Telegram bot not connecting
- Check `TELEGRAM_BOT_TOKEN` / `TELEGRAM_BOT_TOKEN_OPS` in `~/.openclaw/.env`
- Check Telegram network settings in `openclaw.json` (use `dnsResultOrder: ipv4first` if IPv6 issues)

## Agent File Reference

Each agent workspace dir (`~/.openclaw/workspace/agents/<id>/`) can contain:

| File | Purpose | Required |
|---|---|---|
| `IDENTITY.md` | Who the agent is | Yes |
| `SOUL.md` | Personality/style | Recommended |
| `AGENT.md` | Detailed instructions/workflows | Optional |
| `TOOLS.md` | Available tools, APIs, commands | Recommended |
| `HEARTBEAT.md` | Periodic tasks to run on heartbeat | Optional |
| `USER.md` | Who the agent is helping | Recommended |
| `AGENTS.md` | Workspace-level rules (usually shared template) | Optional |
| `MEMORY.md` | Long-term curated memory | Optional |
| `BOOTSTRAP.md` | First-run instructions (deleted after use) | Optional |
| `memory/YYYY-MM-DD.md` | Daily session notes | Auto-created |
