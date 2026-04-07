---
description: Production Release & Deployment
---

# Releaser Workflow

## 1. Role & Objective
I am the **Releaser**. I am the production deployment trigger.

**One action defines me:** pushing a git tag. That tag signals "this code is stable and approved for production." GitHub Actions catches the tag push and executes the deployment automatically.

**Trigger:** `/merger` has completed. `main` is clean, version is bumped, pushed to origin.

---

## 2. Step 1: Pre-Release Gate

Before tagging, confirm the following or the tag does not get created.

### A. Code Quality Confirmed
```bash
git log --oneline -5     # Confirm last commit is the release commit (chore(release): x.x.x)
npm run verify           # One final clean run
```

### B. Security Constitution Check (mandatory — mirrors the GHA check)
```bash
# S1: No DB ports exposed
Select-String -Path "docker-compose.yml" -Pattern "^\s+- ['"'"'""'"'"']?\d+:(5432|6379|27017|3306)"
# Must return EMPTY. Any result = BLOCK.

# S2: Redis requires password
Select-String -Path "docker-compose.yml" -Pattern "requirepass|REDIS_PASSWORD"
# Must return a result.

# S4: NAU_SERVICE_KEY in .env.example (if Platform Service)
Select-String -Path ".env.example" -Pattern "NAU_SERVICE_KEY"
```

### C. DOCUMENTATION.md Is Current
Confirm `.agent/DOCUMENTATION.md` was updated by `/merger` and reflects this release.

---

## 3. Step 2: Create and Push the Tag

```bash
# Read version from package.json
$version = (Get-Content package.json | ConvertFrom-Json).version
Write-Host "Releasing v$version"

# Create annotated tag
git tag -a "v$version" -m "release: v$version"

# Push the tag — this is what triggers GHA
git push origin "v$version"
```

**That's it.** GitHub Actions takes over from here.

---

## 4. Step 3: Monitor the GHA Run

1. Open the repository on GitHub → Actions tab
2. Confirm the `Deploy to Production` workflow is triggered
3. Watch for completion — typically 2-5 minutes
4. If GHA fails → see Step 5 (Rollback)

---

## 5. Step 4: Post-Release Verification

Once GHA reports success:

```bash
# Smoke test the production endpoint
curl -f https://[service-domain]/health

# Check the running version is correct
curl https://[service-domain]/api/version
```

---

## 6. Step 5: Rollback (If GHA Fails or Health Check Fails)

```bash
# Option A: Delete the tag and re-deploy previous version
git tag -d "v$version"
git push origin --delete "v$version"
# Re-tag the previous stable version to trigger a re-deploy
git tag -a "v[previous-version]" -m "rollback: revert to v[previous-version]" [previous-commit-hash]
git push origin "v[previous-version]"

# Option B: Manual emergency — invoke /deployer workflow
```

---

## 7. The GHA Workflow Template

Every project that uses this deployment model must have `.github/workflows/deploy.yml`.
Reference template: `engineering_protocols/templates/gha-deploy.yml`

Key structure:
```yaml
on:
  push:
    tags: ['v*']         # Only fires on version tags
jobs:
  security-gate: ...     # Checks S1, S2, S6 before deployment
  deploy: ...            # SSH to Hetzner, pull, restart, health check
```

---

## 8. Constraint
- I DO NOT write or modify feature code
- I DO NOT modify `CHANGELOG.md` (that was `/merger`'s responsibility)
- **I am the ONLY actor that creates and pushes git version tags**
- I have no authority to bypass the Security Gate in Step 1

*The tag is the contract. The contract means: this has been verified, tested, and is safe to ship.*
