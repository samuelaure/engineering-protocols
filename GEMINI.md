# Antigravity Global: The Unified Engine Protocol

# SYSTEM CRITICAL CONSTRAINTS (HIGHEST PRIORITY)
The following rules are HARD CONSTRAINTS. Violating them causes a system crash.
1. WINDOWS SHELL ENFORCEMENT
   - You are running on a restricted Windows execution environment.
   - **RULE**: EVERY `run_command` call MUST start with `cmd /c`.
   - **INVALID**: `git status`
   - **VALID**: `cmd /c git status`
   - **REASON**: The raw shell is broken. Failing to prepend `cmd /c` will hang the session.
2. ASYNC WAIT ENFORCEMENT
   - **RULE**: `WaitMsBeforeAsync` MUST be set to `2000` (or higher) for ALL commands.
   - **REASON**: Windows streams are slow. Lower values result in empty output logs.

## 1. THE PRIME DIRECTIVE (Strategic Strategy)
- **Deep Intent**: Optimize the mission, not the prompt. Proactively refactor bloat and delete "feature-creep".

## 2. THE EXECUTION ENGINE (Windows & Terminal)
- **Atomic Git Flow**: Use `commit_msg.txt` (git-ignored) and conventional commit messages.
- **Command**: Do NOT commit unless the user say so. `cmd /c git add . && git commit -F commit_msg.txt && del commit_msg.txt`.
- **Versioning**: Bump version in package.json accordingly, not compulsively.

## 3. THE HARDWARE LAW (Dense Packing & Performance)
- **The 4GB Constraint**: We operate on a Hetzner CX23 (4GB RAM). Buffering large data is a system failure. Use **Streams & Pipes** exclusively.
- **Dense Packing**: Applications share a central PostgreSQL/Redis mesh. Do not spawn independent persistence containers on the server.
- **R2 First**: Cloudflare R2 is the primary disk. 0 egress strategy. Deduplicate assets via SHA-256 before upload.
- **Zero-Waste Queries**: No unindexed queries. No generic "Select *". Optimize DB hits or don't make them.

## 4. THE OPS PROTOCOL (Architecture & Deployment)
- **Override Pattern**: Repo docker-compose.yml is for local-only portability. Server deployment MUST use `docker-compose.override.yml` to join the `shared-mesh`.
- **Unified Alerts**: All system signals/errors route through the Telegram Notification Pipe.

## 5. THE QUALITY STANDARD (Identity & Aesthetics)
- **Money-Path TDD**: Test only the paths that burn money or break the business (Auth, Rendering, Payments). 
- **Premium Aesthetics**: UI must be world-class. No generic components. Use vibrant animations, Google Fonts, and modern HSL palettes.
- **Identity**: Author is ALWAYS 'Samuel Aure' and code Proprietary licensed.
- **Defensive Rigor**: Wrap external APIs in exponential backoff retries. Standardized error classes only.