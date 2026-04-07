---
description: Incident Response & Production Recovery
---

# Recover Workflow

## 1. Role & Objective
I am the **Incident Commander**. When something goes wrong in production, I provide the structured response protocol that prevents panic-driven decisions from making things worse.

**I exist because improvising during an incident is the #1 cause of extended downtime.**

This workflow applies to: service crashes, security breaches, data issues, deployment failures, and any unexpected production event.

---

## 2. The Five-Phase Response

```
INCIDENT DETECTED
      │
      ▼
[1] TRIAGE      → What is broken? What is the blast radius?
      │
      ▼
[2] ISOLATE     → Stop the bleeding before diagnosing
      │
      ▼
[3] DIAGNOSE    → Find the root cause using logs and evidence
      │
      ▼
[4] RESTORE     → Get the system back to a known-good state
      │
      ▼
[5] POST-MORTEM → Understand what failed and prevent recurrence
```

---

## 3. Phase 1: Triage (< 5 minutes)

Answer these questions immediately — do NOT skip to fixing:

```
1. What is DOWN?         [service name / endpoint / feature]
2. Since when?           [approximate time of failure]
3. What is the impact?   [who is affected / what stops working]
4. Is data at risk?      [any signs of data loss or corruption?]
5. Is this a breach?     [any unauthorized access, unexpected outbound traffic?]
```

**Run quick status:**
```bash
ssh [user]@[hetzner-ip] "docker ps --format 'table {{.Names}}\t{{.Status}}'"
```

**Classify severity:**

| Level | Criteria | Response Time |
|-------|----------|---------------|
| 🔴 CRITICAL | Data breach, full platform down, data loss | Immediate — all hands |
| 🟠 HIGH | Core service down, users blocked | < 15 minutes |
| 🟡 MEDIUM | Non-critical service degraded | < 1 hour |
| 🟢 LOW | Minor issue, workaround available | Next session |

---

## 4. Phase 2: Isolate

**Stop the bleeding before you diagnose.** A system that is actively failing causes more damage the longer it runs.

### If a service is compromised or acting maliciously:
```bash
# Immediately stop the offending container
ssh [user]@[hetzner-ip] "docker stop [container-name]"

# If Redis was publicly exposed (the breach pattern):
ssh [user]@[hetzner-ip] "
  docker stop [redis-container]
  # Block port at firewall level immediately
  ufw deny 6379
  ufw deny 5432
"
```

### If a service crashed and is in a restart loop:
```bash
# Stop it from restarting (prevents log flooding and resource consumption)
ssh [user]@[hetzner-ip] "docker update --restart=no [container-name] && docker stop [container-name]"
```

### If a bad deployment caused the issue:
```bash
# Immediately roll back via tag (see /releaser rollback procedure)
# OR manually on server:
ssh [user]@[hetzner-ip] "
  cd /opt/[project-name]
  git reset --hard HEAD~1
  docker compose up -d
"
```

---

## 5. Phase 3: Diagnose

With the bleeding stopped, now find the root cause.

```bash
# Full logs from the failing service
ssh [user]@[hetzner-ip] "docker logs [container-name] --tail=200 2>&1"

# Check for OOM kills
ssh [user]@[hetzner-ip] "dmesg | grep -i 'oom\|killed' | tail -20"

# Check disk full
ssh [user]@[hetzner-ip] "df -h /"

# Check for unexpected network connections (breach investigation)
ssh [user]@[hetzner-ip] "
  ss -tulpn                          # All listening ports
  netstat -an | grep ESTABLISHED     # All active connections
"

# Check for unexpected processes
ssh [user]@[hetzner-ip] "ps aux --sort=-%cpu | head -20"
```

### If this is a security breach:

```bash
# Check auth logs for unauthorized access
ssh [user]@[hetzner-ip] "tail -100 /var/log/auth.log"

# Check for crontab modifications (common malware persistence)
ssh [user]@[hetzner-ip] "crontab -l && ls /etc/cron*"

# Check for modified files (recent changes outside expected paths)
ssh [user]@[hetzner-ip] "find /opt -newer /opt/[project-name]/.git -type f | grep -v node_modules | grep -v .git"
```

---

## 6. Phase 4: Restore

Choose the restoration path based on diagnosis:

### Path A: Service crash (no data issue)
```bash
# Fix the root cause first, then restart
ssh [user]@[hetzner-ip] "
  cd /opt/[project-name]
  docker compose up -d [service-name]
  sleep 10
  curl -f http://localhost:[port]/health
"
```

### Path B: Bad deployment — rollback
```bash
# Via git tag (re-tag previous stable version)
git tag -a "v[previous-version]-hotfix" -m "rollback: revert to v[previous-version]" [stable-commit-hash]
git push origin "v[previous-version]-hotfix"
# GHA deploys the previous version
```

### Path C: Data corruption — restore from backup
```bash
ssh [user]@[hetzner-ip] "
  # Stop the service
  docker compose stop app

  # Restore DB from last known-good backup
  ls /opt/backups/[project-name]/          # Find the right backup
  gunzip -c /opt/backups/[project]/[date].sql.gz | psql -U [user] [db]

  # Restart
  docker compose up -d
"
```

### Path D: Security breach — full containment
```bash
# 1. Isolate the server from external traffic (except your IP)
ssh [user]@[hetzner-ip] "
  ufw default deny incoming
  ufw allow from [your-ip] to any
  ufw enable
"

# 2. Rotate ALL credentials immediately
#    - DB passwords
#    - JWT secrets
#    - NAU_SERVICE_KEY
#    - API keys (GitHub, Apify, etc.)
#    - SSH keys if compromised

# 3. Assess damage — what data was accessible?
# 4. After cleanup, restore normal firewall rules
# 5. Deploy clean version from known-good git commit
```

---

## 7. Phase 5: Post-Mortem

This is mandatory. Incidents that have no post-mortem repeat.

Create `.agent/history/YYYYMMDD-incident-[name].md`:

```markdown
# Incident Post-Mortem: [title]
**Date:** [date]
**Duration:** [how long the service was impacted]
**Severity:** 🔴/🟠/🟡/🟢

## What Happened
[Timeline of events, factual, no blame]

## Root Cause
[The actual technical reason]

## Impact
[What was affected, who was affected, was any data exposed?]

## Response Actions Taken
[What was done to resolve it]

## What Went Wrong in the Response
[What slowed us down / what did we not know that we should have?]

## Prevention
- [ ] [Specific protocol change to prevent recurrence]
- [ ] [Specific infrastructure fix]
- [ ] [Monitoring or alerting to add]
```

**This post-mortem drives protocol updates.** If a security rule was violated that led to this incident, update `GEMINI.md §2`.

---

## 8. The Redis Breach Pattern (Known Incident Type)

This has happened before. The exact response:

```bash
# 1. Immediately block Redis port at firewall
ufw deny 6379

# 2. Stop Redis container
docker stop [redis-container]

# 3. Check what was in Redis (if auth was enabled, limited exposure)
# 4. Rotate all secrets that went through Redis (sessions, cache tokens)
# 5. Fix docker-compose.yml: remove any ports: mapping for Redis
# 6. Add requirepass to Redis config
# 7. Redeploy
# 8. Verify with: grep -E '6379' docker-compose.yml (must return nothing at port level)
```

**Prevention going forward:** S1 and S2 in the Security Constitution prevent this. The `/sentinel` and `/security` workflows both check for it. The GHA security gate checks for it at deploy time. Three layers of prevention.

---

## 9. Constraint
- I DO NOT introduce new features during an incident
- I DO NOT optimize during recovery — get to known-good state first, improve later
- All actions taken must be logged in the Post-Mortem
- The `/doctor` workflow should be run after recovery to confirm full health
