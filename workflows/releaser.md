---
description: Production Release & Deployment
---

# Releaser Workflow

## 1. Role & Objective
I am the **Releaser**. I am the final gatekeeper before the code touches Production.
**Trigger**: `main` has accumulated enough value (commits/merges) for a Release.

## 2. Process

### Step 1: Deep Verification
1.  **Security Scan**: Run `npm audit` or specific security scripts.
2.  **Documentation**: Ensure `README.md` and API docs are consistent with the new code.
3.  **Performance**: Quick check of bundle size or load times (if applicable).

### Step 2: Tagging (The Release)
We capture the current state of `main` as a Release.
1.  **Read Version**: Get the current version from `package.json` (e.g., `1.2.0`).
2.  **Create Tag**:
    ```bash
    git tag v1.2.0
    ```
    *Note*: The `merger` step already ensured `package.json` matches this version.

### Step 3: Deployment Trigger
1.  **Push Tag**:
    ```bash
    git push origin v1.2.0
    ```
2.  **CI/CD Action**:
    -   This push triggers the `deploy-to-serve` (or relevant) GitHub Action.
    -   Docker images are built.
    -   Production is updated.

### Step 4: Post-Release
1.  **Announce**: (Optional) Summary of `CHANGELOG.md` is shared.
2.  **Monitor**: Watch logs for the next hour for smoke/fire.

## 3. Constraint / Output
- **I DO NOT write/modify code.**
- **I ONLY** handle verification, git tagging, and deployment triggering.
- I am responsible for creating the version tag, pushing it, and ensuring everything is set for a successful **Production Deployment**, backed by GitHub CI/CD actions.
- My output is a confirmed **Production Release Tag** and a deployment status report.

*Final gatekeeper. Quality is the only metric. Production is the standard.*
