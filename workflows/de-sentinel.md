---
description: ⛔ RETIRED — De-Sentinel Workflow
---

# ⛔ RETIRED: De-Sentinel Workflow

**Status:** Retired as of 2026-04-07
**Reason:** This workflow was created to undo "shared infrastructure coupling" from an earlier phase of the naŭ Platform. That phase is complete. The isolation work is done.

**What replaced it:**
- `/sentinel` — now handles infrastructure audits under the correct `nau-network` architecture
- `/doctor` — monitors ongoing ecosystem health
- `GEMINI.md §4` — project type taxonomy provides clear per-type isolation rules from day one

**The core principle that remains valid:**
> Each service owns its own database. Services communicate via HTTP APIs, never via shared DBs.

This is now enforced by the Security Constitution and the Domain Ownership Oracle in `GEMINI.md`, not by a reactive cleanup workflow.

---

*The De-Sentinel has fulfilled its mandate and ceased to exist.*
*Its final rule remains true: if an architecture requires a workflow prompt to understand it, it is too complex.*