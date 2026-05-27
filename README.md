# NLWOpenclaw — OpenClaw Configuration & Documentation

This repository documents the **OpenClaw** AI agent runtime deployed at Nova Light Works.
It mirrors the live configuration from the server and serves as the source of truth for agent setup, identities, and tools.

## What is OpenClaw?

OpenClaw is a self-hosted AI agent runtime (v2026.3.24). It runs on `moltbot@192.168.88.254` and hosts all Nova Light Works AI agents. It handles:

- Multi-agent orchestration
- Telegram channel integration (two bots: `default` / CEO-Benny, `nova_ops`)
- Heartbeat-driven autonomous operation
- Vector memory search (local SQLite + embeddings)
- Tool/skill execution via a local gateway

## Architecture

```
OpenClaw Gateway  (port: local LAN)
├── main agent    → CEO-Benny   (claude-sonnet-4-6)  — primary Telegram bot
├── nova_ops      → NovaOps     (claude-haiku-4-5)   — stock operations (own Telegram bot)
├── nova_developer → NovaDeveloper (claude-sonnet-4-6)
├── nova_vision   → NovaVision  (claude-sonnet-4-6)
├── nova_design   → NovaDesign  (claude-sonnet-4-6)
├── nova_produkt  → NovaProdukt (claude-sonnet-4-6)
├── nova_salg     → NovaSalg    (claude-sonnet-4-6)
├── nova_marketing → NovaMarketing (claude-sonnet-4-6)
├── nova_intelligence → NovaIntelligence (claude-sonnet-4-6)
├── nova_finance  → NovaFinance (claude-sonnet-4-6)
└── nova_quality  → NovaQuality (claude-sonnet-4-6)
```

## Server

- **Host:** `moltbot@192.168.88.254`
- **Binary:** `/home/moltbot/.nvm/versions/node/v24.14.0/bin/openclaw`
- **Config:** `~/.openclaw/openclaw.json`
- **Workspace:** `~/.openclaw/workspace/`
- **Agent dirs:** `~/.openclaw/agents/<id>/agent/` (auth, models) + `~/.openclaw/workspace/agents/<id>/` (identity files)
- **Node version required:** v22+ (run via `source ~/.nvm/nvm.sh && nvm use 24`)

## Repository Structure

```
NLWOpenclaw/
  README.md                   ← this file
  setup/
    openclaw.json              ← live config (gateway token redacted)
    install.md                 ← install + operation guide
  workspace/
    IDENTITY.md                ← CEO-Benny persona (main agent)
    AGENTS.md                  ← main workspace instructions
    lib/
      company/
        IDENTITY.md            ← Nova Light Works company facts (auto-loaded)
      ops/
        TOOLS.md               ← operational processes (auto-loaded)
  agents/
    nova_ops/                  ← Stock operations agent
      IDENTITY.md
      SOUL.md
      TOOLS.md
      HEARTBEAT.md
      AGENTS.md
      USER.md
      MEMORY.md
    nova_developer/            ← Software developer agent
      IDENTITY.md
      AGENT.md
      SOUL.md
      TOOLS.md
      HEARTBEAT.md
      USER.md
```

## Quick Reference

### SSH to server
```powershell
ssh moltbot@192.168.88.254
# or via Posh-SSH:
$cred = New-Object PSCredential("moltbot", (ConvertTo-SecureString "..." -AsPlainText -Force))
$s = New-SSHSession -ComputerName "192.168.88.254" -Port 22 -Credential $cred -AcceptKey
```

### OpenClaw CLI (must use nvm)
```bash
source ~/.nvm/nvm.sh && nvm use 24 2>/dev/null
openclaw status --json
openclaw agents list
openclaw gateway status --json
openclaw memory index
```

### Restart gateway
```bash
systemctl --user restart openclaw-gateway.service
```

### Update agent files on server
All agent workspace files live in `~/.openclaw/workspace/agents/<id>/`. Edit directly on the server or push from this repo.

## Secrets

Secrets are NOT stored here. They live in `~/.openclaw/.env` on the server:
- `TELEGRAM_BOT_TOKEN` — main/CEO-Benny Telegram bot
- `TELEGRAM_BOT_TOKEN_OPS` — nova_ops Telegram bot
- `SHOPIFY_ADMIN_TOKEN` — Shopify Admin API
- Anthropic API key (set via env or openclaw config)
