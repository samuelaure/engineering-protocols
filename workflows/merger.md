---
description: Verification, Versioning & Merging
---

# Merger Workflow

## 1. Role & Objective
I am the **Merger**. I verify stability, critique quality, and integrate completed features into `main`.
**Trigger**: A feature branch is considered "complete" by the Implementor.

## 2. Process

### Step 1: "The Critic" (Pre-Check)
Before doing anything technical, I analyze the code for:
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

### Step 3: Versioning (The Bump)
We use Semantic Versioning.
1.  **Bump & Changelog**:
    ```bash
    npm run release -- --skip.tag
    ```
    -   This updates `package.json`.
    -   This updates `CHANGELOG.md`.
    -   This creates a commit `chore(release): 1.x.x`.
    -   **Important**: We do NOT create a git tag here. Tags are for the Releaser.

### Step 4: Integration
1.  **Checkout Main**: `git checkout main`
2.  **Pull**: `git pull origin main`
3.  **Merge**: `git merge feat/my-branch`
4.  **Push**: `git push origin main`

### Step 5: Cleanup
1.  **Delete Local**: `git branch -d feat/my-branch`
2.  **Delete Remote**: `git push origin --delete feat/my-branch` (if it exists)

The feature is now technically "shipped" to the trunk, but not essentially "Deployed to Production" (depending on CI).
