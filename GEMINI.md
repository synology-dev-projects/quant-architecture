# 🧭 Quant System: Master Agent Directives & Project Memory

> **System Standard:** L8 Principal Agentic Engineering Hierarchy  
> **Active Development Branch:** `develop2` (Protected Sandbox)

---

## 1. Core Operating Principles
- **Strict YAGNI (Anti-Bloat):** Implement ONLY what is explicitly requested. Do not introduce speculative wrappers, unneeded utility classes, or unsolicited styling churn.
- **Smallest Viable Diff:** Strive for clean, minimal diffs that solve the root cause with zero collateral complexity.
- **Mandatory Prior Plan Approval Gate (HARD INVARIANT):** NO code changes (modifications, creations, or deletions to source code) are allowed before an explicit `implementation_plan.md` has been presented and explicitly approved by the user. Zero code edits before prior plan approval.
- **Mandatory Bug vs. Feature Intake & Grill-Me Gate (HARD INVARIANT):** Whenever the user asks to add code, modify code, or points out an issue with the code/site, the agent **MUST ALWAYS explicitly ask the user whether the request is a Bug or a Feature**:
  - *If a Feature:* Initiate a deep-dive **Grill-Me interview** to clarify requirements, edge cases, UX flows, and API schemas, then produce an `implementation_plan.md` for review.
  - *If a Bug:* Initiate a targeted **Grill-Me interview** to clarify exact **expected vs. actual behavior**, execute the **In-Situ Reproduction Gate (RED)** in the target environment, triage blast radius (Tier 1–4), and produce an `implementation_plan.md` (root cause, reproduction proof, minimal fix) for review.
- **Mandatory Multi-Agent Workflow Selection & Complexity Benchmark Gate (HARD INVARIANT):** Whenever the user approves an implementation plan, the agent **MUST ALWAYS present a brief Benchmark & Complexity Estimate** to help the user gauge the change size:
  - *Estimated Diff Size:* (e.g. `~15–30 lines across 2 files`)
  - *Subsystems Affected:* (e.g. `Single domain: quant-pwa Gateway` vs. `Cross-repo: common-lib + gexdex-api + pwa`)
  - *Blast Radius & Risk:* (`Low`, `Medium`, or `High`)
  - *Recommended Fleet:* (with clear rationale based on complexity)
  The agent must explicitly ask the user to select the workflow (Full Fleet vs. Targeted Crew vs. Single Subagent/Pair) and wait for the user's response before dispatching subagents or executing any implementation.
- **Mandatory Chrome DevTools UI Verification Gate (HARD INVARIANT):** Whenever the frontend UI (HTML, CSS, JavaScript, components, views, modals, PWA assets) is modified or created, the agent **MUST use Chrome DevTools MCP tools** (`navigate_page`, `take_screenshot`, `evaluate_script`, `click`, `take_snapshot`, etc.) to visually inspect and functionally verify the live UI in a real browser session. This MUST happen for ALL workflows before declaring UI work complete.
- **Mandatory Staging Approval Gate Before Master Promotion (HARD INVARIANT):** NEVER merge or push code to `master` (Production) automatically. Code MUST first be deployed to `develop2` (Staging, port `8096`), verified live, and presented to the user. Promotion to `master` requires the user's explicit approval. Zero exceptions.
- **Mandatory Bug In-Situ Reproduction & Symmetric Verification Gate (HARD INVARIANT):** When resolving bugs, agents are **strictly forbidden from modifying application source code** until the assigned domain subagent has written a reproduction test (`tests/test_reproduce_<issue>.py`) and executed it in the active environment to produce a verified **RED (failing)** result. After applying the minimal fix, the agent must re-run that **exact same test in the exact same environment** to verify **GREEN (passing)** before running the universal `./verify.sh` / `pytest` regression suite. The `no-mistakes-reviewer` must automatically reject any bug-fix diff missing a reproduction test file.
- **Zero-Lost Bugs Invariant (MANDATORY):** Whenever a defect, bug, regression, or technical debt item is identified or mentioned during pair programming or review that is NOT immediately fixed in the current turn, the agent MUST immediately log it into `implementation_plans/00_ACTIVE_BACKLOG.md` (and generate a dedicated RFC spec in `p0_...`, `p1_...`, `p2_...`, or `p3_...` for high-impact items). Never let a mentioned bug remain only in ephemeral chat prose.
- **Feature Development Protocol:** When building new features, follow the 6-phase lifecycle (Architecture RFC -> Dependency sequencing -> Strict YAGNI implementation -> Local test gate -> No-Mistakes review -> PR summary).
- **Pre-Flight Read Gates (MANDATORY):** Before proposing code edits or creating implementation plans:
  - *API / Gateway / Telemetry:* MUST view `docs/APIS_AND_GATEWAYS.md` and `.agents/rules/telemetry-and-logging-standards.md`.
  - *Database / Pipelines / ETL:* MUST view `.agents/rules/oracle-concurrency.md` and `.agents/rules/pipeline-standards.md`.
  - *Frontend / Mobile PWA:* MUST view `docs/FRONTEND_AND_BOT_APPLICATIONS.md`.
- **Mandatory Local End-to-End Test Gate (Hard Rule):** NEVER commit or push code to git without first executing end-to-end local testing with real requests and automated test suites (`pytest`). For ANY new feature, bug fix, or hotfix, local in-situ verification MUST pass 100% before requesting approval or running `git commit`.
- **Self-Verification Gate:** Always execute `./verify.sh` or `pytest` locally inside the workspace before presenting code. Never declare a task complete without passing tests.

---

## 2. Golden Architectural Guardrails

### 🗄️ Database & Concurrency (Oracle)
- **UUID Staging Tables:** ALWAYS append a random UUID suffix to temporary staging tables (`f"TMP_{table_name[:12]}_{uuid.uuid4().hex[:8]}".upper()`) in `oracle.py` to prevent race conditions during concurrent cron runs.
- **Connection Pools:** Reuse `@lru_cache` SQLAlchemy engine singletons; NEVER call `engine.dispose()` inside individual query functions.
- **Test Safety:** Unit and integration tests MUST NEVER write to or delete from production tables (`QUANT_LVL_DATA_TE`). Use mock connectors or dedicated test tables (`*_TEST`).

### ⚙️ Data Pipelines & Scrapers
- **Incremental Cutoffs:** Always compare `item_date > cutoff_date` directly without adding day offsets (`+ timedelta(days=1)` is strictly forbidden).
- **Regex Precision:** Numerical strike/price regexes must support 2 to 6 digits (`\d{2,6}(?:\.\d+)?`) to accommodate stocks, ETFs, and indices.
- **Attachment Resilience:** If an attachment download fails, preserve the existing HTML body text instead of overwriting with `None`.
- **Non-Trading Days:** When no new posts or bars exist on weekends or holidays, exit cleanly with `code 0` (prevent false-alarm cron failures).

### ⚡ Backend & APIs (`gexdex-api` & Gateway)
- **Distributed Tracing (`tr-xxxxxx`):** Every chat request MUST generate a unique 6-character trace ID (`tr-{token_hex(3)}`) that prefixes all log lines and is returned in headers (`X-Trace-ID`) and `event: metrics` SSE payload.
- **In-Memory Diagnostic Logging:** The Gateway must maintain an in-memory `_RingBufferHandler` capturing the last 200 logs accessible via `GET /api/diagnostics/logs` for remote debugging without SSH.
- **Split-Model Routing Invariant:** Single-ticker lookups MUST route to Tier 1 (`gemini-3.5-flash-lite`, `budget=0`, omitting `thinking_config`), while multi-source macro synthesis routes to Tier 2 (`gemini-3.7-flash`, `budget=512`).
- **Asymmetric Payload Separation:** Large arrays (40KB strike distributions) must be pushed directly to client Canvas over SSE (`tool_ui` event), while returning a compact ~150-token quantitative summary to the LLM.
- **Rule 5 Strict Completeness:** Models MUST output an explicit breakdown row for EVERY requested ticker in batch queries without exception.
- **DTE / Gamma Math:** Ensure timezone localization handles naive and aware timestamps properly (`exp_date.tz_localize(timezone.utc) - now_dt`) before extracting `.days`.
- **Native Streaming:** Use native `chat.send_message_stream()` in the PWA Gateway for sub-300ms mobile Time-To-First-Token.
- **Authentication:** All microservice endpoints must enforce constant-time API key comparisons (`secrets.compare_digest`). Fail closed if `API_KEY` is missing.
- **CORS:** Explicitly whitelist trusted origins; never combine `allow_origins=["*"]` with `allow_credentials=True`.

### 📱 Clients & Bots
- **Discord Limits:** Always chunk long responses into $\le 1900$-character batches. Never leak internal microservice IPs or stack traces to chat.
- **DOM ID Sync:** In `quant-pwa`, keep `lightbox.js` element IDs strictly synchronized with `index.html` (`lightboxOverlay` and `lightboxImg`).

---

## 3. Subagent Fleet Roster

| Role | Name | Trigger / Purpose |
| :--- | :--- | :--- |
| **Orchestrator** | `QuantFleetCommander` | Cross-repo planning, dependency sequencing, and task dispatching. |
| **Backend & API Engineer** | `BackendSystemsEngineer` | FastAPI endpoints, SQLAlchemy pooling, DTE math, and SSE streaming. |
| **Data & Pipeline Engineer** | `DataPipelineEngineer` | Scrapers, ETL transformations, rate limits, and Oracle upsert loaders. |
| **Mobile & Frontend Engineer** | `MobileFrontendEngineer` | Quant PWA, HTML5 Canvas charting, SWR caching, and Chrome DevTools verification. |
| **Adversarial Quality Auditor** | `AdversarialQualityAuditor` | Adversarial pre-commit audit (Security, Concurrency, Precision, Tests). **MANDATE: Any 🔴 CRITICAL finding is a hard blocker.** |
| **Enterprise Architecture Reviewer** | `EnterpriseArchitectureReviewer` | Enterprise systems evaluation for 1,000 DAU scale (Scalability, Cost, Zero-Bloat). |

---

## 4. Master Promotion & Living Documentation Protocol
- **Zero-Critical-Defects & Reviewer Dealbreaker Policy (HARD GATE):** Under NO CIRCUMSTANCES may any code promotion, PR, or deployment proceed to staging or `master` if an unresolved 🔴 **CRITICAL** flag or architectural dealbreaker exists from **ANY** reviewer (Enterprise Systems Architect, Adversarial Quality Auditor, No-Mistakes Reviewer, Security Scanner, or In-Situ Staging Probes). All reviewer dealbreakers and critical flags are non-negotiable **BLOCKERS**.
- **Reviewer Dealbreaker Bug Remediation Trigger (HARD INVARIANT):** Whenever ANY reviewer flags a dealbreaker or red flag during the development cycle, workflow state transitions immediately to `BLOCKED` and automatically spawns the **Bug Remediation Protocol** (via `python scripts/protocol_graph.py reviewer-dealbreaker --name <issue> --reviewer <role>`). The defect must be isolated, verified RED with a reproduction test, resolved with a minimal diff, verified GREEN, and passed through adversarial audit before the parent feature workflow can resume.
- **Deterministic Secret Scanning Gate:** All repositories enforce automated pre-commit secret scanning via `scripts/check_secrets.py`. Prior to any release cycle completion or master promotion, agents MUST execute `python scripts/check_secrets.py --all` to guarantee zero secrets exist in tracked files.
- **Staging Validation Gate:** All changes must be deployed and validated in the develop staging containers (`develop2` on ports `8091`/`8096`) before merging to `master`.
- **Production-Only Architecture Sync (HARD RULE):** The `quant-architecture` repository is **ONLY updated and pushed when code is promoted to Production (`master` / `prod`)**. During everyday sandbox development and testing on `develop`/`develop2`, `quant-architecture` MUST remain frozen.
- **Mandatory Living Documentation Updates:** EVERY TIME code is promoted or merged into `master`, agents MUST automatically update the architecture documentation in `docs/` (`ARCHITECTURE_OVERVIEW.md`, `DATABASE_AND_DATA_MODELS.md`, `ETL_PIPELINES_AND_INGESTION.md`, `APIS_AND_GATEWAYS.md`, `FRONTEND_AND_BOT_APPLICATIONS.md`, `INFRASTRUCTURE_AND_CICD.md`), update `implementation_plans/00_ACTIVE_BACKLOG.md`, and push `quant-architecture` to `origin master`.
- **No Outdated Docs:** Architecture documentation must always reflect the exact code state running in production. Outdated documentation is treated as a critical defect.
- **Metadata Single Source of Truth (SSOT) Invariant:** Root \.agents/\ is the sole authority for agent skills and rules; root \implementation_plans/00_ACTIVE_BACKLOG.md\ is the sole authority for backlog tasks; root \HANDOFF.md\ tracks active session state; \quant-architecture/docs/\ is the sole authority for production architecture contracts. Creating or syncing shadow copies of \.agents/\ or \implementation_plans/\ inside sub-repos is strictly forbidden.

---

## 5. Mandatory Multi-Agent Execution Mandate
- **Zero Solo Execution:** For all non-trivial features and bug fixes, `QuantFleetCommander` MUST NOT implement code in a monolithic single-agent pass. `QuantFleetCommander` MUST orchestrate specialized subagents in parallel.
- **Workflow Enforcement Loop:**
  1. **Plan & Gate:** Create/update `implementation_plan.md` and get explicit user approval.
  2. **Crew Dispatch:** Spawn specialized crew subagents (`BackendSystemsEngineer`, `MobileFrontendEngineer`, `DataPipelineEngineer`) in parallel via `invoke_subagent`.
  3. **Local Testing:** Each crewmate must execute and pass local `pytest` suites before reporting back to `QuantFleetCommander`.
  4. **Adversarial Audit & Architecture Review:** `QuantFleetCommander` MUST dispatch the `AdversarialQualityAuditor` and `EnterpriseArchitectureReviewer` to audit the diff against security, concurrency, math precision, test coverage, and enterprise scalability.
  4.1. **Zero-Critical-Defects & Reviewer Dealbreaker Gate:** If ANY reviewer flags ANY 🔴 **CRITICAL** defect or architectural dealbreaker, workflow state transitions immediately to `BLOCKED`. It immediately triggers the Bug Remediation Protocol (`reviewer-dealbreaker` on the protocol graph), requiring verified RED reproduction, surgical fix, GREEN validation, and formal reviewer re-audit before proceeding.
  5. **Staging Deploy & Local Teardown:** Push to `develop2` staging containers (`8091`/`8096`) for human validation. `QuantFleetCommander` MUST immediately terminate all local background servers (`uvicorn`, `http.server`, Chrome instances) upon pushing to release local ports (3000, 8000) and free PC memory.
  6. **Master Promotion & Living Docs:** Upon user approval, promote to `master` and update `docs/` in the same commit cycle.

---

## 6. Mandatory Graph-Bound Domain Skill Adherence
Every phase in `scripts/protocol_graph.py` is bound to a specialized engineering skill runbook under `.agents/skills/`. Agents MUST consult and adhere strictly to the bound skill before writing code:
- **Intake & Orchestration (`PHASE_0_INTAKE`)**: Follow `.agents/skills/captain-orchestrator/SKILL.md` for Grill-Me interviews, user alignment, and task scoping.
- **Reproduction Scaffolding (`PHASE_1_RED_GATE`)**: Follow `.agents/skills/repro-scaffolder/SKILL.md` for fast TDD reproduction, isolated mock fixtures, and failing assert verification.
- **UI & PWA Frontend (`PHASE_2_SURGICAL_FIX` / `PHASE_3_EXECUTION`)**: Follow `.agents/skills/bloomberg-terminal-components/SKILL.md` for Bloomberg dark-theme tokens (`#0b0f19`, `#10b981`, `#fbbf24`), 44px minimum touch targets, and zero horizontal/vertical layout overflows.
- **Database & Hypertables (`PHASE_2_SURGICAL_FIX` / `PHASE_3_EXECUTION`)**: Follow `.agents/skills/timescale-options-analytics/SKILL.md` for single-scan CTE queries, composite partial indexes, and explicit date casting.
- **Streaming & FastAPI (`PHASE_2_SURGICAL_FIX` / `PHASE_3_EXECUTION`)**: Follow `.agents/skills/fastapi-sse-streaming/SKILL.md` for native `StreamingResponse`, `ReadableStream` client consumer loops, and keep-alive ping headers.
- **Pre-Commit Code Audit (`PHASE_4_AUDIT`)**: Follow `.agents/skills/no-mistakes-reviewer/SKILL.md` for adversarial diff inspection across security, precision, concurrency, and test coverage.
- **Staging & Docker Deployment (`PHASE_5_STAGING`)**: Follow `.agents/skills/docker/SKILL.md` (and `.agents/skills/synology-nas-guardian/SKILL.md`) for zero-bloat multi-stage Docker builds, strict `< 350MB` RAM budgets, and healthcheck watchdogs.
- **Production Gate (`PHASE_6_PRODUCTION_GATE`)**: Follow `.agents/skills/architecture-review-agent/SKILL.md` for 1,000 DAU enterprise scalability validation.


