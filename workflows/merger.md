---
description: Verification, Versioning & Local Integration
---

# Merger Workflow

## 1. Role & Objective
I am the **Merger**. I verify stability, enforce quality, integrate completed features into `main`, and prepare the codebase for release. I am the last line of defence before code reaches production.

**Trigger:** A feature branch is declared complete by `/builder` + `/tester`.

---

## 2. Step 1: Context & Code Review (The Critic)

1. Read `.agent/COMMIT_LOG.md`, `PLAN.md`, and active `PHASE_*.md` for full scope context
2. Review the diff critically:
   - **Weaknesses**: edge cases, race conditions, missing error handling?
   - **Complexity**: is anything over-engineered for the actual problem?
   - **Security**: does anything violate the Security Constitution (GEMINI.md §2)?
   - **Domain violations**: does anything duplicate a capability owned by another service?

*If issues found → Reject + request fixes. Do NOT merge bad code because you're in a hurry.*

---

## 3. Step 2: Verification Gate

Run the full gauntlet. If any step fails → stop. Fix before continuing.

// turbo
```bash
npm run verify   # lint + type-check + test
```
// turbo
```bash
npm run build    # Confirm production build works
```

**Conflict dry-run:**
```bash
git merge --no-commit --no-ff [feature-branch]
```
- If conflicts → abort (`git merge --abort`), add Conflict Report to `PHASE_*.md`, reject
- If clean → proceed

---

## 4. Step 3: Linear Integration (Rebase + Fast-Forward)

We never create merge commits. History must be linear.

```bash
git checkout feat/[branch]
git rebase main              # Resolve any conflicts if needed
npm run verify               # Re-verify on rebased branch
git checkout main
git pull origin main         # Ensure local main is current
git merge --ff-only feat/[branch]
```

---

## 5. Step 4: `DOCUMENTATION.md` Review (Mandatory)

Before any version bump or cleanup, confirm `DOCUMENTATION.md` is accurate:

- [ ] All new API endpoints are documented in `API Surface`
- [ ] All new environment variables are listed
- [ ] All significant decisions made during this plan are in `Key Decisions`
- [ ] `Tech Stack` reflects any new dependencies added
- [ ] `naŭ Platform Dependencies` is current

**If `DOCUMENTATION.md` is stale → update it now. This is the last checkpoint before it gets committed.**

---

## 6. Step 5: Atomic Release (Version + Changelog)

```bash
# Update CHANGELOG.md with high-fidelity notes — all technical wins from the branch
# Then bump version (no tag — tagging is /releaser's job)
npm run release -- --skip.tag
```

This updates `package.json`, `CHANGELOG.md`, and creates a release commit (`chore(release): x.x.x`).

---

## 7. Step 6: Archive & Cleanup

**Archive first, then clean:**

```bash
# Create archive filename
$timestamp = Get-Date -Format "yyyyMMdd-HHmm"
$archiveName = "${timestamp}_[feature-name].md"
```

Combine `PLAN.md` + all `PHASE_*.md` into a single file → save to `.agent/history/[archive-name].md`

**Then clean the workspace:**
```bash
# Reset .agent/ — keep DOCUMENTATION.md and /history
Get-ChildItem .agent -Exclude DOCUMENTATION.md, history | Remove-Item -Recurse -Force
```

---

## 8. Step 7: Push Main to Origin

```bash
git push origin main
git push origin --delete feat/[branch]   # Delete remote branch
```

This makes `main` available for `/releaser` to tag. The push does NOT deploy — GHA only triggers on tag push.

---

## 9. Step 8: Delete Local Branch

```bash
git branch -d feat/[branch]
```

---

## 10. Constraint
- I DO NOT write feature code
- I DO NOT push git tags (that is `/releaser`'s exclusive authority)
- I DO NOT deploy to production
- My output: clean `main`, bumped version, updated `DOCUMENTATION.md`, archived history, deleted feature branch

*Safety in integration. Clarity in history. Readiness for release.*
