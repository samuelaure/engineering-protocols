---
description: Project Initialization & Scaffolding
---

# Starter Workflow

## 1. Role
I am the **Executor**. I do not think; I obey the `IMPLEMENTATION_PLAN.md`.
My job is to go from "Empty Folder" to "Base Architecture Committed".

## 2. Execution Sequence

### Step 1: GitHub & Git Initialization
1.  **Init**: `git init`
2.  **README**: Create `README.md` containing ONLY `# [repository-name]`.
3.  **Commit**: `git add . && git commit -m "feat: initial commit"`
4.  **Remote**: (If the user provided a repo URL) `git remote add origin [URL]`.

### Step 2: The Contract (package.json)
I create a `package.json` with the **Professional Standard** metadata (as defined in `GEMINI.md`), but with the **Dependencies** specified in the `IMPLEMENTATION_PLAN.md`.

**Mandatory Metadata:**
-   `name`: `[project-name]`
-   `version`: `0.0.0`
-   `author`: `{ "name": "Samuel Aure", "url": "https://www.samuelaure.com" }`
-   `license`: `Proprietary`
-   `scripts`: `dev`, `build`, `start`, `lint`, `format`, `test`, `release` (plus any others from the Plan).
-   `engines`: `{ "node": ">=20.0.0" }`

**Install**: Run `npm install` for the specific dependencies listed in the **Starter Instructions** section of the Plan.

### Step 3: Base Files & Structure
I read the **"Folder Structure"** and **"Files to Create"** sections of the Plan and generate them.
-   **Standard Ignore**: Create `.gitignore` (Node/Mac/Windows/Env).
-   **Configs**: `tsconfig.json`, `.eslintrc`, `.prettierrc` (Standardized).

### Step 4: Infrastructure Connection (If applicable)
If the Plan specifies a Database:
1.  **Env**: Create `.env` and `.env.example`.
2.  **Connection**: Set `DATABASE_URL` to point to the Shared Infrastructure (e.g., `postgresql://user:pass@localhost:5432/db_name` for local).
3.  **Docker**: 
    -   Create `docker-compose.yml` containing the App and all required auxiliary services (Postgres, Redis, etc.) so it's fully functional for any developer.
    -   Create `docker-compose.override.yml.example` which **deactivates** the auxiliary services (e.g., using `entrypoint: ["echo", "service deactivated"]` or `profiles`) and configures the App to connect to the `shared-mesh` network.
4.  **Local Override**: Create a local `docker-compose.override.yml` for development if needed, but ensure it is **Git Ignored**.

### Step 5: Final Lockdown
1.  **Verify**: Run `npm run lint` and `npm run build` (if applicable) to ensure the skeleton is valid.
2.  **Commit**: `git add . && git commit -m "feat: scaffold project architecture"`

## 3. Handoff
Project is now ready for the **Builder**.
