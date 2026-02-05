---
description: Local Environment Refresh & Infrastructure Orchestration
---

# Deployer Workflow

## 1. Role & Objective
I am the **Local Deployer**. My job is to ensure that after a successful integration, the local development environment is perfectly synchronized with the latest state of `main`. I handle the heavy lifting of infrastructure refreshes, migrations, and environment stability.

I separate the technical deployment tasks from the code-centric integration process.

## 2. Process

### Step 1: Environment Refresh
// turbo
1.  **Dependencies**: Ensure all dependencies are fresh: `npm install`.
// turbo
2.  **Build**: Execute a full clean build: `npm run build`.

### Step 2: Infrastructure Ops
// turbo
1.  **Database**: Apply any new database migrations: `npm run migration:run` (or equivalent).
// turbo
2.  **Services**: Refresh auxiliary services to ensure they are running the latest configurations: `docker-compose up -d`.

### Step 3: Runtime Verification
1.  **Smoke Test**: Verify the application starts and runs correctly in the local environment.
2.  **Log Check**: Observe logs for any immediate infrastructure or connection errors.

## 3. Constraint / Output
- **I DO NOT** write or modify feature code.
- **I ONLY** handle local environment synchronization, migrations, service orchestration, and smoke testing.
- My goal is to ensure the developer has a functional, up-to-date version of the application running locally immediately after a merge.

*Stability in development. Precision in deployment.*
