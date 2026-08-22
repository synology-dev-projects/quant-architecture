---
trigger: always_on
---

# 🛡️ Bug Triage & Remediation Protocol (L8 Standard)

When resolving bugs, errors, or code review findings across the Quant System, all agents MUST follow this 4-step procedure:

---

## 1. Triage by Blast Radius (Fix Hierarchy)
Never attempt to resolve all findings in a single unstructured pass. Triage and sequence fixes according to this severity hierarchy:

1. **Tier 1: 🔴 Critical Blockers (Immediate Priority)**
   - Silent data loss (e.g. scraper cutoff drops).
   - Database race conditions & table locking (e.g. static temp table collisions).
   - Destructive test side effects (e.g. tests mutating production DB tables).
   - Plaintext credentials or session leaks in source code.

2. **Tier 2: 🟠 High-Impact Correctness**
   - Mathematical / Greek calculation bugs (e.g. DTE operator precedence).
   - Socket & connection leaks (e.g. missing `finally: ib.disconnect()`).
   - Regex boundaries & format truncation (e.g. 4-digit constraints).
   - Unauthenticated API routes or timing attacks.

3. **Tier 3: 🟡 Reliability & UI Glitches**
   - DOM ID mismatches and chart lightbox failures.
   - Message length overflow (e.g. Discord 2000-character limits).
   - Missing repository stubs and incomplete pipeline skeletons.

4. **Tier 4: 🟢 Polish & Accessibility**
   - ARIA labels, comment hygiene, and typing aesthetics.

---

## 2. The 4-Step Remediation Loop

```mermaid
graph TD
    A[Step 1: Isolate 1-2 Related Bugs] --> B[Step 2: Write Failing Test / TDD]
    B --> C[Step 3: Smallest Viable Diff]
    C --> D[Step 4: Verify Locally & Lock In Rule]
```

### Step 1: Micro-Plan Scope & Backlog Ingestion
- **Zero-Lost Bugs Invariant (MANDATORY):** Any bug, edge case, or defect discovered during development, testing, or review that is NOT immediately patched in the active turn MUST be logged immediately into `implementation_plans/00_ACTIVE_BACKLOG.md` with priority, component, and blast radius.
- Scope each active remediation task to **1 to 3 closely related bugs** within a single subsystem or dependency boundary.

### Step 2: Test-First Reproduction (TDD)
- The Captain dispatches the appropriate crew subagent (`backend_agent`, `frontend_agent`, or `pipeline_agent`).
- The subagent writes an isolated regression test in `tests/` that reproduces the bug (RED).

### Step 3: Smallest Viable Diff (Strict YAGNI)
- The subagent patches ONLY the lines necessary to fix the root cause with zero collateral churn.

### Step 4: Multi-Agent Verification, Review & Lock-In
- The subagent runs local `pytest` to verify the fix (GREEN).
- The Captain dispatches the `no-mistakes-reviewer` subagent for pre-merge adversarial review (Phase 5a).
- For architectural changes, dispatch `architecture_review_agent` (Phase 5b).
- The Captain deploys to `develop2` staging (`8091`/`8096`) for human validation.
- Upon approval, promote to `master`, move the task from `00_ACTIVE_BACKLOG.md` to `completed_archive/`, and synchronize living documentation in `docs/`.
