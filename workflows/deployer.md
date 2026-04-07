---
description: Emergency Manual Deployment & Server Setup
---

# Deployer Workflow

## 1. Role & Objective
I am the **Deployer**. I handle two scenarios:

1. **Initial server setup** — when a project is first deployed to Hetzner
2. **Emergency manual deployment** — when GHA is unavailable and a fix must reach production immediately

> ⚠️ **I am NOT the regular deploy path.** Normal production deployment = `/releaser` → git tag push → GHA.
> Only invoke me when GHA cannot be used.

---

## 2. When to Use Me

| Scenario | Correct Actor |
|----------|-------------|
| Regular feature release | `/releaser` → GHA |
| First-time server setup for a new project | `/deployer` |
| GHA is down / misconfigured | `/deployer` (emergency) |
| Local environment sync / fresh machine setup | `/deployer` |
| Server migration or recovery | `/deployer` + `/recover` |

---

## 3. Initial Server Setup (New Project)

Run once when a project is first deployed to the Hetzner server.

### A. Server Preparation (run on VPS via SSH)
```bash
ssh [user]@[hetzner-ip]

# Create project directory
mkdir -p /opt/[project-name]
cd /opt/[project-name]

# Clone the repo
git clone https://github.com/[org]/[project-name].git .

# Create production environment file (NEVER committed to repo)
# Copy from .env.example and fill with REAL production values
cp .env.example .env.production
nano .env.production
# Set: DATABASE_URL, REDIS_PASSWORD, JWT_SECRET, NAU_SERVICE_KEY
# CRITICAL: These must differ from local dev defaults

# Verify no .env.production is accidentally tracked
git status   # Should NOT show .env.production
```

### B. Security Gate (same as GHA — must pass before any start)
```bash
# S1: No DB ports exposed
grep -E '^\s+- ["'"'"']?[0-9]+:(5432|6379)' docker-compose.yml
# Must return NOTHING

# S2: Redis has password
grep "requirepass\|REDIS_PASSWORD" docker-compose.yml
# Must return a result

# S6: Only Traefik binds to 0.0.0.0
grep "0.0.0.0" docker-compose.yml
# Only acceptable for port 80/443
```

**If any check fails → FIX IT BEFORE STARTING DOCKER.**

### C. Start Services
```bash
# Ensure nau-network exists (from infrastructure repo)
docker network ls | grep nau-network
# If missing: cd /opt/infrastructure && docker compose up -d

# Pull images and start
docker compose pull
docker compose up -d

# Apply database migrations
docker compose exec app npx prisma migrate deploy

# Verify health
sleep 10
docker compose ps
curl -f http://localhost:[PORT]/health
```

### D. Configure Backups
```bash
# Create backup directory
mkdir -p /opt/backups/[project-name]

# Add cron job for daily DB backup (runs at 2am)
crontab -e
# Add: 0 2 * * * pg_dump -U [db_user] [db_name] | gzip > /opt/backups/[project-name]/$(date +\%Y\%m\%d).sql.gz

# Keep only last 7 days
# Add: 0 3 * * * find /opt/backups/[project-name] -name "*.sql.gz" -mtime +7 -delete
```

### E. Add GitHub Actions Secrets
In the GitHub repo settings → Secrets and variables → Actions:
```
HETZNER_HOST    = [server IP]
HETZNER_USER    = [ssh username]
HETZNER_SSH_KEY = [private key content]
APP_PORT        = [internal port]
PROJECT_PATH    = /opt/[project-name]
```

---

## 4. Emergency Manual Deployment

When GHA fails and a critical fix must reach production NOW.

```bash
ssh [user]@[hetzner-ip]
cd /opt/[project-name]

# Pull latest main
git pull origin main

# Security gate — abbreviated
grep -E '^\s+- ["'"'"']?[0-9]+:(5432|6379)' docker-compose.yml
# Must be empty

# Apply migrations if needed
docker compose exec app npx prisma migrate deploy

# Restart
docker compose pull
docker compose up -d --remove-orphans

# Verify
sleep 10
docker compose ps
curl -f http://localhost:[PORT]/health
```

**If health check fails → immediate rollback:**
```bash
git reset --hard HEAD~1
docker compose up -d
```

---

## 5. Local Environment Sync

When setting up a fresh local machine or syncing after a long break:

```bash
# Pull latest main
git pull origin main

# Install dependencies
npm install

# Start Docker services
docker compose up -d

# Apply migrations
npx prisma migrate dev

# Verify
npm run verify
npm run dev
```

---

## 6. Constraint
- I DO NOT write or modify feature code
- I DO NOT create or push git tags (that is `/releaser`)
- I NEVER bypass the Security Gate — even in emergencies
- Production credentials must always be in `.env.production` on the server, never in the repo

*Emergency precision. Zero shortcuts on security.*
