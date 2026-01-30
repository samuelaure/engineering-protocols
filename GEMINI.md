# Global Engineering Protocol  
**Lean systems. Elite quality. No excess.**

---

## 0. CORE PHILOSOPHY

Build like a top-tier engineer operating under startup constraints.

Every decision must balance:
- **Speed of delivery**
- **Long-term maintainability**
- **Performance efficiency**
- **Architectural clarity**

Avoid:
- Over-engineering
- Premature abstraction
- Decorative complexity

Prefer:
- Simple structures that scale
- Explicitness over magic
- Fewer moving parts, better placed

When in doubt → **simplify**.

---

## 1. EXECUTION ENVIRONMENT (HARD CONSTRAINTS)

### 1.1 Windows Shell Enforcement

All terminal commands MUST run through:

cmd /c <command>


Failure to do this may hang the session.

---

### 1.2 Async Output Stability

WaitMsBeforeAsync >= 2000


Prevents empty or truncated logs in Windows environments.

---

## 2. TECHNOLOGY SELECTION RULES

The **Preferred Stack** is the default for almost all projects.

Deviate only when justified by one of these rules:

### Rule 1 — The Project Can Be Lighter
If the scope is small, short-lived, or does not benefit from a full framework, use the **Lighter Stack**.

### Rule 2 — The Project Clearly Demands Another Tool
If the project type strongly benefits from a different technology (real-time engine, data processing stack, media pipeline, etc.), choose the better tool without violating core engineering standards.

No random stack switching. Changes must be justified by **simplicity or clear technical advantage**.

---

## 3. PREFERRED CORE STACK (DEFAULT)

Use for serious products, platforms, and scalable systems.

### Language & Runtime
- **TypeScript (strict mode)**
- **Node.js LTS**

### Backend
- **Framework:** NestJS (Express acceptable when structure is minimal)
- **API Style:** REST, modular, resource-oriented
- **Validation:** Zod *or* class-validator (standardize per project)
- **ORM:** Prisma
- **Auth:** JWT (access + refresh), HTTP-only cookies when possible
- **Logging:** Pino
- **Cache/Queues (if needed):** Redis

### Frontend
- **Framework:** Next.js (App Router)
- **UI:** React
- **Styling:** TailwindCSS
- **Data Fetching:** TanStack Query
- **Icons:** Lucide
- **Animation:** Framer Motion (only when it adds value)

### Database & Storage
- **Primary DB:** PostgreSQL (shared instance)
- **Object Storage:** Cloudflare R2 (S3-compatible)
- Deduplicate uploads using SHA-256 before storing.

### Infrastructure
- Docker
- docker-compose
- Shared PostgreSQL container
- Shared Redis container (only when needed)

### Testing
- Jest or Vitest (backend)
- Testing Library (frontend)
- Cypress only when true E2E value exists

---

## 4. PREFERRED LIGHTER STACK

Use when full frameworks add unnecessary weight.

### Backend (Light Mode)
- Express or Fastify
- Zod validation
- Prisma (if DB needed) or direct SQL for very small tools
- Pino logger

### Frontend (Light Mode)
- Vite + React
- TailwindCSS
- Minimal client state (avoid global stores unless required)

### Scripts / Automation
- Node.js + TypeScript
- No framework unless structure truly demands it

Lighter does **not** mean messy. All architecture and quality rules still apply.

---

## 5. SHARED INFRASTRUCTURE STRATEGY

### Container Philosophy
- Keep container count low
- Share services across projects
- Never duplicate DB/Redis containers unnecessarily

### docker-compose.yml (Public / Dev)
Must allow anyone to run the project easily.

Typical services:
- `app`
- `postgres` (shared)
- `redis` (optional, shared)

### docker-compose.override.yml (Server)
- Connect to shared infra network (e.g. `shared-mesh`)
- Reuse existing PostgreSQL/Redis if already running
- Avoid redundant containers

---

## 6. PERFORMANCE PRINCIPLES

We build efficient systems by discipline, not fear of limits.

Rules:
- Always paginate list endpoints
- Never run unbounded queries
- Select only required DB fields
- Add indexes for real query paths
- Stream large files (never load fully into memory blindly)

Systems should run smoothly even on 4GB RAM without being designed around scarcity.

---

## 7. BASE PROJECT TOOLING (ALL PROJECTS)

These dependencies and standards apply to **every repository**.

### Core Dev Dependencies
- TypeScript
- ESLint
- Prettier
- @types/node

### Backend Defaults
- ts-node or tsx
- jest or vitest

### Frontend Defaults
- eslint-plugin-react
- eslint-config-prettier

---

### Standard Scripts (Apply Everywhere)

| Script | Purpose |
|--------|---------|
| `dev` | Start development mode |
| `build` | Production build |
| `start` | Run production app/server |
| `lint` | Run ESLint |
| `format` | Run Prettier |
| `type-check` | TypeScript validation |

Backend projects should also include:

| Script | Purpose |
|--------|---------|
| `test` | Run test suite |
| `test:watch` | Watch mode testing |

Prisma projects should include:

| Script | Purpose |
|--------|---------|
| `prisma:generate` | Generate Prisma client |
| `prisma:migrate` | Run migrations |
| `prisma:seed` | Seed database |

---

## 8. ARCHITECTURE PRINCIPLES

- Modular monolith by default
- Structure by **domain**, not by technical layer
- Clear separation of business logic and infrastructure
- Composition over inheritance
- No god classes or mega-files

Microservices only when there is a **real scaling or deployment boundary**, not anticipation.

---

## 9. CODE QUALITY RULES

- Functions do one thing
- Prefer early returns over deep nesting
- No magic numbers
- No silent failures
- Explicit names over clever names
- Use typed, meaningful error classes
- Delete dead code immediately

### Error Handling
All external API calls must include:
- Timeout
- Retry with exponential backoff
- Graceful failure handling

---

## 10. UI / PRODUCT QUALITY STANDARD

Interfaces must feel:
- Modern
- Intentional
- Lightweight

Rules:
- No default-looking UIs
- Consistent spacing system
- Limited color palette
- Subtle animation > loud effects

Premium does **not** mean heavy.

---

## 11. ANTI-BLOAT DIRECTIVE

Before adding any feature, ask:

1. Does this directly serve the core user outcome?
2. Can this be solved by simplifying instead of adding?
3. Is this abstraction needed now, or imagined for later?

If unsure → **do less**.

---

## 12. TESTING PHILOSOPHY

Test impact, not trivia.

Always test:
- Authentication & authorization
- Payments / billing logic
- Critical data transformations
- Any logic that could cause financial or data loss

Avoid testing framework internals or trivial mappings.

Goal: **maximum protection with minimum test weight**.

---

## 13. GIT & VERSIONING

- Do not commit unless explicitly instructed
- Use conventional commits
- Atomic commits (one purpose per commit)
- Use semver intentionally

Command pattern:

cmd /c git add . && git commit -F commit_msg.txt && del commit_msg.txt


---

## 14. DEFINITION OF DONE

Done means:
- Code is clean and understandable
- No obvious performance waste
- Errors handled predictably
- Logs are meaningful, not noisy
- Structure follows project standards

Not done when:
- Every theoretical edge case is covered
- Imaginary scale problems are solved

---

## PACKAGE.JSON PROFESSIONAL STANDARD

Every project must treat `package.json` as a **technical contract**, not just a dependency list.  
It must clearly communicate ownership, runtime expectations, tooling, and ecosystem integration.

### 1. Identity & Ownership (Required)
Define clear authorship and project traceability.

- `name`
- `version`
- `description`
- `author` (object with Samuel Aure + https://www.samuelaure.com)
- `license` (Proprietary)
- `repository`
- `homepage`
- `bugs`

### 2. Runtime & Environment Clarity (Required)

Make runtime expectations explicit to prevent environment drift.

- `engines.node` must specify minimum LTS version
- Use `"type": "module"` or be explicit about CommonJS
- Use `"private": true` for apps, `false` only for published packages

### 3. Script Standardization (Required)

Every project must expose a consistent command surface.

Mandatory scripts:
- `dev`
- `build`
- `start`
- `lint`
- `format`
- `type-check`

Backend projects must also include:
- `test`
- `test:watch`

Prisma projects must include:
- `prisma:generate`
- `prisma:migrate`
- `prisma:seed`

Scripts must be minimal, predictable, and aligned with the stack.

### 4. Tooling Signals (Required)

Quality tooling must be visible at the package level.

- ESLint configuration (or reference to config file)
- Prettier configuration (or reference)
- TypeScript as a dependency in TS projects

### 5. Dependency Discipline (Recommended)

- Avoid unnecessary dependencies
- Prefer stable, widely adopted libraries
- Remove unused packages immediately
- Use `sideEffects: false` for libraries when safe

### 6. Distribution Clarity (When Applicable)

For libraries or shared packages:

- `main`
- `types`
- `exports`
- `files` (only ship what is needed)

For CLI tools:
- `bin` field properly defined

### 7. Automation Hooks (Optional but High-Quality)

When commit discipline matters:

- `lint-staged`
- `husky` pre-commit hooks

These enforce quality without manual policing.

---

A professional `package.json` should allow another engineer to understand:
- What the project is
- How to run it
- What environment it expects
- How quality is enforced

If this is unclear, the configuration is incomplete.


---

**Build fast. Keep it light. Deliver systems a senior engineer would respect.**
