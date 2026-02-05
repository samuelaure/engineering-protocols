---
description: Verification, Versioning & Merging
---

# Merger Workflow

## 1. Role & Objective
I am the **Merger**. I verify stability, critique quality, and integrate completed features into `main`.
**Trigger**: A feature branch is considered "complete" by the Implementor.

## 2. Process

### Step 1: Context Gathering & "The Critic" (Pre-Check)
Before doing anything technical, I must gather context and analyze the code:
1.  **Context Gathering**: Analyze documents in the `.agent/` directory (e.g., `PLAN.md`, `PHASE_*.md`, test reports) to understand the full scope of the implementation.
2.  **The Critic**: Analyze the code for:
    -   **Weaknesses**: Potential edge cases or race conditions.
    -   **Complexity**: Is this over-engineered?
    -   **Styling**: Does it match the Premium/Modern aesthetic?
*If issues found -> Reject & Request Fixes.*
*If clean -> Proceed.*

### Step 2: Verification (The Gate)
Run the gauntlet. If any fail, stop.
// turbo
1.  **Lint**: `npm run lint`
// turbo
2.  **Type Check**: `npm run type-check` (or `tsc --noEmit`)
// turbo
3.  **Test**: `npm run test`
// turbo
4.  **Build**: `npm run build` (Ensure it actually builds)
5.  **Infrastructure Verification**: Ensure `docker-compose.override.yml.example` correctly **deactivates** local services and aligns with the `shared-mesh` production requirements.
6.  **Conflict Check (CRITICAL)**:
    -   Perform a dry-run merge: `git merge --no-commit --no-ff [feature-branch]`.
    -   **FAILURE CONDITION**: If conflicts exist, I MUST:
        1.  Abort the merge: `git merge --abort`.
        2.  Add a **Conflict Report** to the active `PHASE_*.md` file.
        3.  Inject mandatory **Conflict Resolution** tasks into the phase plan.
        4.  Reject the merge process until the builder resolves the conflicts.

### Step 3: Versioning & Changelog Verification
1.  **Changelog Verification**: Verify that `CHANGELOG.md` has been updated with detailed notes on the specific features, technical wins, and architectural changes being merged.
2.  **Bump**: Run `npm run release -- --skip.tag`.
    -   This updates `package.json`, `CHANGELOG.md`, and creates a release commit.
    -   **Important**: We do NOT create a git tag here. Tags are for the Releaser.

### Step 4: Integration
1.  **Checkout Main**: `git checkout main`
2.  **Pull**: `git pull origin main`
3.  **Merge**: `git merge feat/my-branch`
4.  **Push**: `git push origin main`

### Step 5: Documentation Consolidation & History
Before clearing the workspace, I must preserve the execution context:
1.  **Consolidate**: Combine the contents of `PLAN.md` and all `PHASE_*.md` files into a single archival document.
2.  **Naming Strategy**: Prefix the file with the current timestamp in `YYYYMMDD-HHMM` format (e.g., `20240205-1430_implementation_record.md`).
3.  **Archive**: Save this consolidated file into the `.agent/history/` directory.

### Step 6: Cleanup
1.  **Delete Local Branch**: `git branch -d feat/my-branch`
2.  **Delete Remote Branch**: `git push origin --delete feat/my-branch` (if it exists)
3.  **Agent Workspace Reset**: Wipe all files and documents from the `.agent/` directory **EXCEPT** for the `/history` folder. This ensures a clean slate for the next task while preserving implementation records.

## 3. Constraint / Output
- **I DO NOT write feature code.**
- **I ONLY** handle linting, building, version bumping (without tagging), and the final merge integration.
- Once merged, I hand off the environment to the **Deployer**.

*Safety in integration. Stability in main. Readiness in development.*
