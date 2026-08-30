# 🧭 Quant System: Master Agent Directives & Project Memory

> **System Standard:** L8 Principal Agentic Engineering Hierarchy  
> **Active Development Branch:** `develop2` (Protected Sandbox)

---

## 1. Core Operating Principles
- **Strict YAGNI (Anti-Bloat):** Implement ONLY what is explicitly requested. Do not introduce speculative wrappers, unneeded utility classes, or unsolicited styling churn.
- **Smallest Viable Diff:** Strive for clean, minimal diffs that solve the root cause with zero collateral complexity.
- **Mandatory Prior Plan Approval Gate (HARD INVARIANT):** NO code changes (modifications, creations, or deletions to source code) are allowed before an explicit `implementation_plan.md` has been presented and explicitly approved by the user. Zero code edits before prior plan approval.
- **Mandatory Multi-Agent Workflow Selection Gate (HARD INVARIANT):** Whenever the user approves an implementation plan, the agent **MUST ALWAYS ask the user which multi-agent workflow to use** (e.g. Standard Full Fleet, Targeted Crew, Pair Programming, etc.) and wait for the user's response before dispatching subagents or executing any implementation.
- **Bug Remediation Protocol:** When resolving bugs, strictly follow the 4-step loop (Isolate 1-2 bugs -> Write failing test -> Smallest viable diff -> Local verify & rule lock-in). Prioritize Tier 1 (Data loss & Concurrency) before Tier 2/3.
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
| **Orchestrator** | `captain` | Cross-repo planning, dependency sequencing, and task dispatching. |
| **Data Engineer** | `pipeline-crew` | Scrapers, ETL transformations, rate limits, and Oracle upsert loaders. |
| **API Engineer** | `backend-crew` | FastAPI endpoints, SQLAlchemy pooling, DTE math, and SSE streaming. |
| **Reviewer (Quality)** | `no-mistakes` | Adversarial pre-commit audit (Security, Concurrency, Precision, Tests). |
| **Reviewer (Architecture)** | `architecture-review-agent` | Enterprise systems evaluation for 1,000 DAU scale (Scalability, Cost, Zero-Bloat). |

---

## 4. Master Promotion & Living Documentation Protocol
- **Staging Validation Gate:** All changes must be deployed and validated in the develop staging containers (`develop2` on ports `8091`/`8096`) before merging to `master`.
- **Production-Only Architecture Sync (HARD RULE):** The `quant-architecture` repository is **ONLY updated and pushed when code is promoted to Production (`master` / `prod`)**. During everyday sandbox development and testing on `develop`/`develop2`, `quant-architecture` MUST remain frozen.
- **Mandatory Living Documentation Updates:** EVERY TIME code is promoted or merged into `master`, agents MUST automatically update the architecture documentation in `docs/` (`ARCHITECTURE_OVERVIEW.md`, `DATABASE_AND_DATA_MODELS.md`, `ETL_PIPELINES_AND_INGESTION.md`, `APIS_AND_GATEWAYS.md`, `FRONTEND_AND_BOT_APPLICATIONS.md`, `INFRASTRUCTURE_AND_CICD.md`), update `implementation_plans/00_ACTIVE_BACKLOG.md`, and push `quant-architecture` to `origin master`.
- **No Outdated Docs:** Architecture documentation must always reflect the exact code state running in production. Outdated documentation is treated as a critical defect.

---

## 5. Mandatory Multi-Agent Execution Mandate
- **Zero Solo Execution:** For all non-trivial features and bug fixes, the Captain MUST NOT implement code in a monolithic single-agent pass. The Captain MUST orchestrate specialized subagents in parallel.
- **Workflow Enforcement Loop:**
  1. **Plan & Gate:** Create/update `implementation_plan.md` and get explicit user approval.
  2. **Crew Dispatch:** Spawn specialized crew subagents (`backend_agent`, `frontend_agent`, `pipeline_agent`) in parallel via `invoke_subagent`.
  3. **Local Testing:** Each crewmate must execute and pass local `pytest` suites before reporting back to the Captain.
  4. **Adversarial Audit:** The Captain MUST dispatch the `no-mistakes-reviewer` subagent to audit the combined diff against security, concurrency, math precision, and test coverage.
  5. **Staging Deploy & Local Teardown:** Push to `develop2` staging containers (`8091`/`8096`) for human validation. The Captain MUST immediately terminate all local background servers (`uvicorn`, `http.server`, Chrome instances) upon pushing to release local ports (3000, 8000) and free PC memory.
  6. **Master Promotion & Living Docs:** Upon user approval, promote to `master` and update `docs/` in the same commit cycle.

