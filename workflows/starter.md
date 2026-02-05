---
description: Project Initialization & Scaffolding
---

# Starter Workflow

## 1. Role
I am the **Executor**. I do not think; I obey the `.agent/PLAN.md` and `.agent/PHASE_*.md` files.
My job is to go from "Empty Folder" to "Base Architecture Committed".

## 2. Execution Sequence

### Step 1: Git Foundations (The Triple Start)
I MUST initialize the repository with exactly these three atomic commits in order:
1.  **Commit 1: Start**
    -   Create `README.md` containing ONLY `# [project-name]`.
    -   `git init`
    -   `git add README.md && git commit -m "start"`
2.  **Commit 2: Professional Docs**
    -   Update `README.md` to be a high-fidelity, industry-standard document.
    -   Include: Description, Stack, Quick Start, and Architecture overview based on the `PLAN.md`.
    -   `git add README.md && git commit -m "docs: update README.md"`
3.  **Commit 3: Workspace Hygiene**
    -   Create a comprehensive `.gitignore`. It MUST include:
        -   Environment files (`.env`).
        -   System files (Mac/Windows).
        -   Node modules (`node_modules`).
        -   **The `.agent/` folder** (Crucial: agent-specific documents are for local orchestration only and MUST NOT be committed).
    -   `git add .gitignore && git commit -m "chore: initial .gitignore"`

4.  **Remote**: (If provided) `git remote add origin [URL]`.

### Step 2: The Contract (package.json)
I create a `package.json` with the **Professional Standard** metadata (as defined in `GEMINI.md`), but with the **Dependencies** specified in the `.agent/PLAN.md`.

**Mandatory Metadata:**
-   `name`: `[project-name]`
-   `version`: `0.0.0`
-   `author`: `{ "name": "Samuel Aure", "url": "https://www.samuelaure.com" }`
-   `license`: `Proprietary`
-   `scripts`: `dev`, `build`, `start`, `lint`, `format`, `test`, `release` (plus any others from the Plan).
-   `engines`: `{ "node": ">=20.0.0" }`

**Install**: Run `npm install` for the specific dependencies listed in the **Starter Instructions** section of the `PLAN.md`.

### Step 3: Base Files & Structure
I read the **"Folder Structure"** and **"Files to Create"** sections of the `PLAN.md` and generate them.
-   **Configs**: `tsconfig.json`, `.eslintrc`, `.prettierrc` (Standardized).
-   **Commit**: `git add . && git commit -m "feat: scaffold base configurations"` (Exclude source code files for now).

### Step 4: Infrastructure Connection (If applicable)
If the `PLAN.md` specifies a Database:
1.  **Env**: Create `.env` and `.env.example`.
2.  **Connection**: Set `DATABASE_URL` to point to the Shared Infrastructure (e.g., `postgresql://user:pass@localhost:5432/db_name` for local).
3.  **Docker**: 
    -   Create `docker-compose.yml` containing the App and all required auxiliary services (Postgres, Redis, etc.) so it's fully functional for any developer.
    -   Create `docker-compose.override.yml.example` which **deactivates** the auxiliary services (e.g., using `entrypoint: ["echo", "service deactivated"]` or `profiles`) and configures the App to connect to the `shared-mesh` network.
4.  **Local Override**: Create a local `docker-compose.override.yml` for development if needed, but ensure it is **Git Ignored**.

### Step 5: Final Lockdown & Runtime Verification
// turbo
1.  **Static Check**: Run `npm run lint` and `npm run build` (if applicable) to ensure the skeleton is valid.
// turbo
2.  **Runtime Verification (CRITICAL)**: 
    -   Launch the application (e.g., `npm run dev` or `npm start`).
    -   Keep the process running for **~10 seconds**.
    -   Observe logs and ensure no crashes or immediate errors occur.
    -   This confirms the scaffold is functionally sound.
3.  **Commit Architecture**: `git add . && git commit -m "feat: complete project architecture scaffolding"`

## 3. Handoff
Project is now ready for the **Builder**.
