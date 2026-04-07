---
description: Project Initialization & Scaffolding
---

# Starter Workflow

## 1. Role
I am the **Foundation Layer**. I go from "empty folder" to "running skeleton" with zero ambiguity.
I obey `PLAN.md` and `PHASE_1.md` exactly. I do not improvise.

---

## 2. Pre-Flight: Read the Project Type

**First action:** Read `PLAN.md` and confirm the project type (A/B/C/D/E/F/G/H from `GEMINI.md §4`).
The type determines which scaffold template I apply. Do not proceed without it.

---

## 3. Step 1: Git Foundations (All Types)

Three atomic commits, in order:

```bash
# Commit 1: The seed
echo "# [project-name]" > README.md
git init && git add README.md && git commit -m "start"

# Commit 2: Professional docs
# (Update README.md with full content from PLAN.md)
git add README.md && git commit -m "docs: update README"

# Commit 3: Workspace hygiene
# (Create .gitignore — see §8 for standard content)
git add .gitignore && git commit -m "chore: initial .gitignore"
```

Remote (if provided): `git remote add origin [URL]`

---

## 4. Step 2: Type-Specific Scaffold

### Type A — Platform Service / Type B — Product App

```bash
# package.json with mandatory metadata (§9)
# tsconfig.json (strict mode)
# .eslintrc + .prettierrc (standardized)
# src/ structure per PLAN.md module definitions
# docker-compose.yml (isolated: app + own postgres + own redis)
# .env (local dev defaults) + .env.example (no real credentials)
# prisma/schema.prisma (if DB required)
```

**Docker Compose mandatory structure:**
```yaml
services:
  app:
    container_name: [project-name]
    restart: "no"           # Dev: no auto-restart; Production: unless-stopped
    networks:
      - app-network         # Internal isolation
      - nau-network         # Platform mesh (if ecosystem service)
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.[name].rule=Host(`[name].localhost`)"

  postgres:
    image: postgres:15-alpine
    container_name: [name]-postgres
    # SECURITY: NO ports: definition — internal only
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    container_name: [name]-redis
    command: redis-server --requirepass "${REDIS_PASSWORD}"   # S2: password mandatory
    # SECURITY: NO ports: definition — internal only
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
  nau-network:
    external: true          # Only if Type A or ecosystem-connected B
```

**⚠️ Security check (S1, S2):** Before committing `docker-compose.yml`, verify:
- [ ] No `ports:` on postgres or redis
- [ ] Redis has `--requirepass`

### Type C — Mobile App (Expo)

```bash
npx create-expo-app@latest ./ --template blank-typescript
# Configure app.json (name, slug, version)
# src/screens/, src/components/, src/services/
# services/ApiClient.ts → points to companion backend
# No Docker, no docker-compose.yml
```

### Type D — Static / Brand Site

```bash
# Option 1: React + Vite
npx create-vite@latest ./ --template react-ts

# Option 2: Pure HTML/CSS/JS (no framework)
# index.html, css/main.css, js/main.js

# Mandatory:
# - base font from Google Fonts (defined in PLAN.md)
# - CSS variables for design tokens
# - meta description, og:title, og:description, og:image
```

### Type E — Orchestration Glue

```bash
# Minimal Node.js TypeScript service
# src/index.ts — main entry
# src/skills/ or src/handlers/ — modular logic
# docker-compose.yml — app only, no standalone DB unless stated in PLAN.md
# Joins nau-network
```

### Type F — CLI Tool / Script

```bash
# package.json with bin field
# src/cli.ts — commander-based entry
# No docker-compose.yml unless explicitly required
# README with Usage + Options sections
```

### Type G — Infrastructure Component

```bash
# docker-compose.yml only
# .env + .env.example
# README.md with setup steps
# No application source code
```

### Type H — Standalone / Creative

```bash
# Follow PLAN.md. No enforced template.
# Minimum: README.md + DOCUMENTATION.md + .gitignore
# Still use conventional commits
```

---

## 5. Step 3: Base Configuration Files

**Standardized `tsconfig.json` (TypeScript projects):**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
```

**Standardized `.prettierrc`:**
```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

---

## 6. Step 4: Environment Files

```bash
# .env — local dev defaults (auto-generated, in .gitignore)
# .env.example — template with no real values (committed)

# Key pattern:
DATABASE_URL=postgresql://dev_user:dev_password@postgres:5432/[name]_dev
REDIS_PASSWORD=local_dev_password_only
JWT_SECRET=local_dev_secret_32chars_minimum_x
NAU_SERVICE_KEY=                    # (if Platform Service)
```

**Rule (S3):** Local defaults must be clearly non-production values (e.g., `dev_password_only`).
Production secrets are never stored locally and never committed.

---

## 7. Step 5: Runtime Verification

```bash
# 1. Infrastructure up
docker compose up -d
# 2. Dependencies installed
npm install
# 3. DB migrations (if applicable)
npx prisma migrate dev
# 4. Static checks
npm run verify
# 5. Runtime smoke test — run for 10 seconds, observe logs
npm run dev
```

// turbo
If all pass: `git add . && git commit -m "feat: complete project architecture scaffolding"`

---

## 8. Standard `.gitignore`

```gitignore
# Environment
.env
.env.*
!.env.example

# Agent workspace (EXCEPT documentation and history)
.agent/*
!.agent/DOCUMENTATION.md
!.agent/history/

# Dependencies
node_modules/
.venv/

# Build outputs
dist/
.next/
build/

# OS
.DS_Store
Thumbs.db

# IDE
.idea/
*.code-workspace
```

---

## 9. Mandatory `package.json` Metadata

```json
{
  "name": "[project-name]",
  "version": "0.0.0",
  "author": { "name": "Samuel Aure", "url": "https://www.samuelaure.com" },
  "license": "Proprietary",
  "engines": { "node": ">=20.0.0" },
  "scripts": {
    "dev": "...",
    "build": "...",
    "start": "...",
    "lint": "eslint ./src",
    "format": "prettier --write ./src",
    "type-check": "tsc --noEmit",
    "test": "...",
    "verify": "npm run format && npm run lint && npm run type-check && npm run test",
    "release": "standard-version --skip.tag"
  }
}
```

---

## 10. Handoff
Project is ready for `/builder` when:
- [ ] 3 foundation commits exist
- [ ] `docker compose up -d` succeeds without errors
- [ ] `npm run verify` passes
- [ ] `npm run dev` starts without crashing
- [ ] No database ports exposed
- [ ] Redis has a password
