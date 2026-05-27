# HEARTBEAT.md - NovaDeveloper Periodic Tasks

Kør disse tjek ved hvert heartbeat og rapporter afvigelser til Benny via `sessions_send` med `target: main`.

## 1. Dashboard status
```bash
pm2 list | grep nova-dashboard
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/
```
Forventet: pm2 = `online`, HTTP = 200. Afvigelse → restart og notér Benny.

## 2. Gateway status
```bash
openclaw gateway status --json | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['service']['runtime']['status'])"
```
Forventet: `running`. Afvigelse → `systemctl --user restart openclaw-gateway.service`.

## 3. Åbne GitHub issues
```bash
gh issue list --repo nlwzhc/nova-dashboard --state open --json number,title,createdAt 2>/dev/null
```
Notér nye issues i daily memory. Send summary til Benny hvis > 3 åbne.

## 4. Package sikkerhedstjek (kun mandag)
```bash
cd ~/nova-dashboard && npm audit --json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); v=d.get('metadata',{}).get('vulnerabilities',{}); print('critical:', v.get('critical',0), 'high:', v.get('high',0))"
```
Hvis `critical > 0`: opret GitHub issue + notér Benny.

## 5. Daily memory
Opdater `memory/YYYY-MM-DD.md` med tjekresultat.
