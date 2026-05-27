# TOOLS.md - NovaDeveloper Local Environment

## Development Environment

- **Machine:** moltbot (Linux, Intel Core Ultra 9 285H, 4 cores, 8 GB RAM)
- **OS:** Ubuntu (kernel 6.8.0-106-generic)
- **Node:** via nvm — `~/.nvm/versions/node/v24.14.0/bin/`
- **Shell PATH note:** Kør altid `source ~/.bashrc` eller brug fuld sti til nvm binaries hvis de ikke er i PATH

## Nova Dashboard

| | |
|--|--|
| **Sti** | `~/nova-dashboard` |
| **Port** | 3000 |
| **pm2 navn** | `nova-dashboard` |
| **GitHub** | `https://github.com/nlwzhc/nova-dashboard` (privat) |
| **gh konto** | `nlwzhc` (aktiv), `jens-bentzon_danfoss` (sekundær) |
| **DB** | `~/.nova-dashboard/data.sqlite` |
| **Build** | `cd ~/nova-dashboard && npm run build` |
| **Restart** | `pm2 restart nova-dashboard` |
| **Logs** | `pm2 logs nova-dashboard --lines 50 --nostream` |

### Typisk deploy-sekvens
```bash
cd ~/nova-dashboard
# ... lav ændringer ...
npx tsc --noEmit              # TypeScript check
npm run build                  # Production build
pm2 restart nova-dashboard     # Genstart
sleep 4
curl -s http://localhost:3000/api/system | python3 -c "import sys,json; print(json.load(sys.stdin).get('cpu', 'ok'))"
git add -A && git commit -m "feat: ..." && git push
```

### API endpoints (alle GET medmindre angivet)
- `/api/system` — CPU, RAM, disk, temp
- `/api/ollama` — Ollama status + job-log
- `/api/openclaw` — Gateway, agents, channels
- `/api/actions` (POST) — `restart-gateway`, `restart-dashboard`
- `/api/tokens` — Token-forbrug og costs
- `/api/conversations` — Session + beskeder
- `/api/sync` (POST) — Sync JSONL → SQLite

## OpenClaw

| | |
|--|--|
| **Config** | `~/.openclaw/openclaw.json` |
| **Workspace** | `~/.openclaw/workspace/` |
| **Gateway service** | `systemctl --user restart openclaw-gateway.service` |
| **Heartbeat model** | `anthropic/claude-haiku-4-5-20251001` |
| **Interactive model** | `anthropic/claude-sonnet-4-6` |
| **JSONL sessions** | `~/.openclaw/agents/<id>/sessions/` |

### Nyttige openclaw kommandoer
```bash
openclaw status --json          # Full status
openclaw gateway status --json  # Gateway specifikt
openclaw agents list            # Alle agenter
openclaw config get <path>      # Læs config
openclaw config set <path> <val> # Skriv config (gem backup automatisk)
openclaw memory index           # Reindekser memory
openclaw system heartbeat last  # Seneste heartbeat
```

## GitHub

```bash
gh auth status                  # Tjek hvilke konti der er logget ind
gh auth switch --user nlwzhc    # Skift til nlwzhc (dashboard repo)
gh issue list --repo nlwzhc/nova-dashboard
gh issue create --repo nlwzhc/nova-dashboard --title "..." --body "..."
gh pr list --repo nlwzhc/nova-dashboard
```

## System Services

```bash
pm2 list                        # Alle pm2 processer
pm2 restart <navn>              # Genstart
pm2 logs <navn> --lines 50      # Logs
systemctl --user status openclaw-gateway.service
systemctl --user restart openclaw-gateway.service
```

## Vector Memory (memory_search)

**Provider:** Local (SQLite + embeddings)
**Model:** embeddinggemma-300m-qat-Q8_0.gguf (CPU)

### Usage
- Brug `memory_search` ved session-start med relevante søgetermer
- Søg bredt først (f.eks. "dashboard", "deploy", "github") og snæver ind
- Søg altid FØR du spørger brugeren om kontekst du måske allerede har

### Hvad er indekseret
- Daglige memory filer (`memory/YYYY-MM-DD.md`)
- Langsigtet hukommelse (`MEMORY.md`)
- Agent-instruktioner og workspace-filer

### Reindekser efter nye filer
```bash
openclaw memory index
```
