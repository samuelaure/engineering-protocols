# naŭ Engineering Protocols

> **Lean systems. Elite quality. No excess.**
> The institutional brain of the naŭ Platform. Every decision, every actor, every rule — lives here.

---

## What This Is

A complete engineering protocol system for the **naŭ Platform** — a distributed, Dockerized ecosystem of creator tools, AI services, and personal brand applications operated by a solo developer.

These protocols exist to:
- **Protect the architecture** from bad decisions, including requests from the operator himself
- **Enforce security** at every layer — automatically, not by memory
- **Eliminate cognitive load** — every workflow is a decision-free path through a complex task
- **Ensure system longevity** — systems that survive without daily attention

---

## The Director: `GEMINI.md`

`GEMINI.md` is the **Constitution** — the global rule of law. Every actor, every workflow, every implementation decision defers to it.

It contains:
- **The Intercept Gate** — fires before any implementation, regardless of how the request is phrased
- **The Security Constitution** — 10 immutable rules, including the Redis public exposure prevention
- **The naŭ Ecosystem Context** — the three-layer platform model
- **Project Classification Taxonomy** — 8 project types, each with different infrastructure rules
- **Server Profile & Resource Budget** — derived from current Hetzner CX23 specs
- **Workflow Actor Registry** — all actors and the regular deployment flow
- **Naming Conventions** — naŭ brand, `nau-` prefixes, Docker naming

**Read this before anything else.**

---

## Workflow Actors

Actors are activated by `/slash-command`. Each has a single, clear responsibility.

### Discovery Cluster — Ideas → Plan
| Actor | Command | Role |
|-------|---------|------|
| Explorer | `/brainstormer` | Expands ideas + flags architectural red flags. Output: `FEATURES.md` |
| Architect | `/architect` | Designs new systems with project-type classification. Output: `PLAN.md` + `DOCUMENTATION.md` |
| Planner | `/planner` | Iterates existing systems. Output: new `PHASE_N.md` + updated `DOCUMENTATION.md` |

### Execution Cluster — Plan → Code → Committed
| Actor | Command | Role |
|-------|---------|------|
| Starter | `/starter` | Scaffolds per project type. Type A/B/E get full Docker+API template. |
| Builder | `/builder` | Implements. Ecosystem-aware. Updates `DOCUMENTATION.md` on behavioral changes. |
| Tester | `/tester` | Tests + quality gate. Can block merger. |
| Commit | `/commit` | Atomic conventional commits. Commits `DOCUMENTATION.md`. Never commits other `.agent/` files. |

### Integration Cluster — Code → Main → Production
| Actor | Command | Role |
|-------|---------|------|
| Merger | `/merger` | Rebase + fast-forward + version bump + `DOCUMENTATION.md` review + archive history. Pushes `main`. |
| Releaser | `/releaser` | Pre-release security gate → `git tag` push → **GHA deploys automatically** |
| Deployer | `/deployer` | ⚠️ Emergency/fallback only. Initial server setup or manual deploy when GHA is unavailable. |

### Guardian Cluster — Continuous
| Actor | Command | Role |
|-------|---------|------|
| Auditor | `/auditor` | Architecture debt, complexity, domain ownership review. Can block. |
| Security | `/security` | Full security audit. Mandatory before `/releaser`. Can block. |
| Sentinel | `/sentinel` | Infrastructure + `nau-network` + domain ownership audit. |
| Doctor | `/doctor` | Ecosystem health check — services up? Security holes? Domain violations? Backups? |
| Recover | `/recover` | Incident response — triage → isolate → diagnose → restore → post-mortem. |

> ⚠️ **`/de-sentinel` is retired.** See `workflows/de-sentinel.md` for history.

---

## Regular Deployment Flow

```
/brainstormer (optional)
      ↓
/architect or /planner
      ↓
/starter (new projects only)
      ↓
/builder → /tester → /security
      ↓
/merger
      ↓
/releaser → git tag push → GitHub Actions → Hetzner VPS
```

**/deployer** is not in this flow. It is break-glass only.

---

## GHA Deployment Template

Every naŭ Platform service uses the same tag-triggered deployment workflow:

```
engineering_protocols/templates/gha-deploy.yml
```

Copy to `.github/workflows/deploy.yml` in each project and fill in the placeholders.

The GHA workflow enforces Security Constitution rules S1, S2, S6, S7 **before touching the server**. Any violation blocks deployment automatically — no human memory required.

---

## Document Architecture (per project)

```
.agent/
├── DOCUMENTATION.md     ← PERMANENT. Committed to repo. Always current.
├── PLAN.md              ← Active plan. Deleted on merger cleanup. Always starts fresh.
├── PHASE_1.md           ← Phase numbering is plan-scoped. Always starts at 1.
├── PHASE_N.md
├── COMMIT_LOG.md        ← Builder handoff. Deleted after commit.
└── history/
    └── YYYYMMDD-HHMM_[feature].md   ← Archived plans.
```

**Global ecosystem:**
```
c:\Users\Sam\code\.agent\
├── DOCUMENTATION.md     ← naŭ Platform ecosystem overview (this is the one to read first)
├── PLAN.md              ← Current ecosystem-level integration plan (Phases 0-5)
├── PHASE_N.md
├── SERVICE_MAP.md       ← Live network topology + domain ownership
└── history/
```

---

## Common Workflows

### 🌱 New Project
`/brainstormer` → `/architect` → `/starter` → build loop → `/security` → `/merger` → `/releaser`

### 🔄 Feature Addition
`/planner` → `/builder` → `/tester` → `/commit` → `/security` → `/merger` → `/releaser`

### 🔥 Production Incident
`/recover` → (diagnose + restore) → `/doctor` (post-recovery health check)

### 🔍 Architecture Audit
`/auditor` → `/sentinel` → `/planner` (if remediation needed) → build loop

### 🏥 Weekly Health Check
`/doctor` — takes ~10 minutes, catches problems before users do

### ⚡ Quick Fix (single file, < 30min)
Directly as `/builder` → `/commit` — still atomic, still conventional, no full plan needed

---

## Project Structure

```
engineering_protocols/
├── GEMINI.md                    ← The Constitution (read this first)
├── README.md                    ← This file
├── workflows/
│   ├── brainstormer.md          ← /explore (Discovery)
│   ├── architect.md             ← /architect (Discovery)
│   ├── planner.md               ← /planner (Discovery)
│   ├── starter.md               ← /starter (Execution)
│   ├── builder.md               ← /builder (Execution)
│   ├── tester.md                ← /tester (Execution)
│   ├── commit.md                ← /commit (Execution)
│   ├── merger.md                ← /merger (Integration)
│   ├── releaser.md              ← /releaser (Integration)
│   ├── deployer.md              ← /deployer (Emergency)
│   ├── auditor.md               ← /auditor (Guardian)
│   ├── security.md              ← /security (Guardian)
│   ├── sentinel.md              ← /sentinel (Guardian)
│   ├── doctor.md                ← /doctor (Guardian) — NEW
│   ├── recover.md               ← /recover (Guardian) — NEW
│   └── de-sentinel.md           ← RETIRED
└── templates/
    └── gha-deploy.yml           ← GitHub Actions deploy template
```

---

*Build like a top-tier engineer operating under startup constraints.*
*Direction precedes construction. Security is not a feature — it is the floor.*
