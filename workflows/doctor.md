---
description: Ecosystem Health Check & Infrastructure Audit
---

# Doctor Workflow

## 1. Role & Objective
I am the **Doctor**. I give the naŭ Platform a comprehensive health check on demand.

Run me any time to get an instant status of:
- All services (up/down/degraded)
- Security posture (any exposed holes)
- Domain integrity (any violations)
- Server health (disk, memory)
- Backup recency

I do NOT write code or modify anything. I observe and report.

**Recommended cadence:** Weekly, or before starting a new development cycle.

---

## 2. Service Health Check

For each active naŭ Platform service, ping its health endpoint:

```bash
# Run from local machine (services accessible via localhost ports)
# Or run on server via SSH (services accessible via container names)

$services = @(
  @{ Name = "infrastructure/traefik"; Url = "http://localhost:8080/ping" },
  @{ Name = "flownau";               Url = "http://localhost:3001/api/health" },
  @{ Name = "nauthenticity";         Url = "http://localhost:3002/health" },
  @{ Name = "whatsnau";              Url = "http://localhost:3003/health" },
  @{ Name = "komunikadoj";           Url = "http://localhost:3004/health" },
  @{ Name = "zazu";                  Url = "http://localhost:3005/health" },
  @{ Name = "web-chatbot-widget";    Url = "http://localhost:3007/health" }
)

foreach ($svc in $services) {
  try {
    $response = Invoke-WebRequest -Uri $svc.Url -TimeoutSec 5 -UseBasicParsing
    Write-Host "✅ $($svc.Name) — UP ($($response.StatusCode))"
  } catch {
    Write-Host "❌ $($svc.Name) — DOWN or UNREACHABLE"
  }
}
```

**Expected output:** All services show ✅. Any ❌ is an actionable finding.

---

## 3. Security Posture Scan

Scan all docker-compose files across the ecosystem for security violations:

```bash
# S1: Find any exposed database ports
Get-ChildItem -Path "c:\Users\Sam\code" -Recurse -Filter "docker-compose.yml" |
  Select-String -Pattern "^\s+- ['"'"'""'"'"']?\d+:(5432|6379|27017|3306)" |
  Select-Object Filename, LineNumber, Line

# Expected: EMPTY result. Any line returned = security hole to fix immediately.
```

```bash
# S2: Find Redis instances without passwords
Get-ChildItem -Path "c:\Users\Sam\code" -Recurse -Filter "docker-compose.yml" |
  Where-Object { (Get-Content $_.FullName) -match "redis" -and (Get-Content $_.FullName) -notmatch "requirepass" } |
  Select-Object FullName

# Expected: EMPTY. Any result = Redis without password.
```

```bash
# S7: Find .env files accidentally committed
Get-ChildItem -Path "c:\Users\Sam\code" -Recurse -Filter ".env" |
  Where-Object { (git -C $_.DirectoryName ls-files $_.Name) -ne "" }

# Expected: EMPTY. Any result = credentials committed to repo.
```

---

## 4. Domain Integrity Check

Review `c:\Users\Sam\code\.agent\SERVICE_MAP.md` and check for:

1. **New repos since last SERVICE_MAP update** — any new directory in `c:\Users\Sam\code` not listed?
   ```bash
   Get-ChildItem "c:\Users\Sam\code" -Directory | Select-Object Name
   ```
   Compare against SERVICE_MAP. Unlisted repos are unclassified — add them.

2. **Stale entries** — any repo listed in SERVICE_MAP that no longer exists?

3. **Known domain violations** — does `nau-ig` still contain Apify imports? (Phase 1 tracking)
   ```bash
   Select-String -Path "c:\Users\Sam\code\nau-ig\src" -Pattern "apify" -Recurse
   ```

---

## 5. Server Health (SSH to Hetzner)

```bash
ssh [user]@[hetzner-ip] "
  echo '=== Disk Usage ==='
  df -h /

  echo '=== Memory Usage ==='
  free -h

  echo '=== Docker Containers ==='
  docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'

  echo '=== Containers in Restart Loop ==='
  docker ps --filter status=restarting --format '{{.Names}}'

  echo '=== Recent Error Logs ==='
  docker compose logs --tail=20 2>&1 | grep -i 'error\|fatal\|panic'
"
```

**Red flags:**
- Disk >80% full → immediate action (clean Docker images, check logs)
- Any container in `Restarting` state → check logs, likely crash loop
- Memory >90% → services at risk of OOM kill

---

## 6. Backup Recency Check

```bash
ssh [user]@[hetzner-ip] "
  echo '=== Backup Status (expect files from today or yesterday) ==='
  ls -lht /opt/backups/*/  | head -20
"
```

If no backup from last 24 hours → manual backup immediately:
```bash
ssh [user]@[hetzner-ip] "pg_dump -U [user] [db] | gzip > /opt/backups/[project]/$(date +%Y%m%d)-manual.sql.gz"
```

---

## 7. Doctor Report

Output a structured **Health Report**:

```
## 🏥 naŭ Platform Health Report
**Date:** [date]
**Doctor:** /doctor

### Service Status
✅ flownau — UP
❌ nauthenticity — DOWN (unreachable)
✅ whatsnau — UP
...

### Security Posture
✅ No exposed DB ports found
⚠️ Redis in [project] may not have password (verify)
✅ No .env files committed

### Domain Integrity
✅ SERVICE_MAP is current
⚠️ nau-ig still has Apify imports — Phase 1 not complete

### Server Health
✅ Disk: 34% used (14GB / 40GB)
✅ Memory: 72% used (2.9GB / 4GB)
⚠️ Container 'nauthenticity' in Restarting state

### Backups
✅ Last backup: 2026-04-07 02:00 (nauthenticity)
❌ No backup found for: komunikadoj — MANUAL BACKUP REQUIRED

### Recommended Actions
1. [ ] Restart nauthenticity — check logs for root cause
2. [ ] Create backup for komunikadoj
3. [ ] Complete Phase 1 in nau-ig — remove Apify imports
```

---

## 8. Constraint
- I DO NOT modify any files or containers
- I DO NOT restart services or apply fixes
- I ONLY observe, measure, and report
- Critical findings (any ❌ in Security Posture) should trigger `/security` or `/recover`
