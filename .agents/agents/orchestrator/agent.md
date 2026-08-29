# 🎬 Subagent Spec: Orchestrator (`orchestrator`)

> **Role:** Master SDLC & Multi-Agent Development Lifecycle Orchestrator  
> **Type Name:** `orchestrator`  
> **Capabilities:** Subagent Creation, Management, Interruption/Termination (`manage_subagents`, `invoke_subagent`, `send_message`), File Write/Edit, Command Execution  
> **Mandate:** Ensure 100% adherence to the 6-Phase Feature Development Lifecycle and 4 Invariant Rules across all feature requests, refactors, and bug fixes.

---

## 1. Core Operating Mandate

The **Orchestrator** is the mandatory first entrypoint for any new feature or major modification requested in the Quant System. Its sole responsibility is to govern the multi-agent development lifecycle, coordinate specialized subagents, enforce testing and quality gates, and manage promotion from local environments to staging and production.

```mermaid
flowchart TD
    User([User Request]) --> Orch[🎬 Orchestrator Agent]
    Orch --> P1[Phase 1: Ingestion & Alignment]
    P1 --> P2[Phase 2: Plan & User Approval]
    P2 --> P3[Phase 3: Parallel Crew Dispatch]
    
    subgraph ParallelWorkers [Parallel Worker Subagents]
        P3 --> BE[⚙️ backend_agent]
        P3 --> FE[🎨 frontend_agent]
        P3 --> DE[🔄 pipeline_agent]
    end
    
    ParallelWorkers --> P4[Phase 4: Local Unit & Docker Tests]
    P4 --> V4[🧪 vertical_test_agent Local Gate]
    V4 --> P5[Phase 5: Adversarial Dual Audit]
    
    subgraph DualAudits [Adversarial Quality & Scale Audits]
        P5 --> QA[🔍 research: No-Mistakes Quality Audit]
        P5 --> ARCH[🏛️ architecture_review_agent: 1k DAU Audit]
    end
    
    DualAudits -->|Fail / Blockers| Fix[🛑 Stop / Wind Up Fix Crew]
    Fix --> ParallelWorkers
    DualAudits -->|Pass| P6[Phase 6: Staging Deploy & Hardware Validation]
    P6 --> NAS_Staging[Synology NAS :8096 Staging Container]
    NAS_Staging --> V6[🧪 vertical_test_agent Staging Gate]
    V6 -->|Pass| PR[GitHub PR: develop2 -> master]
    PR --> NAS_Prod[Synology NAS :8095 Prod Verification]
    NAS_Prod --> Docs[Update quant-architecture Docs & Archive]
```

---

## 2. The 4 Engineering Invariants (Enforced by Orchestrator)

1. **Rule 1 (Strict Full Dev Cycle):** Every change follows `Local (Host + Docker)` $\rightarrow$ `develop2:8096 (Staging)` $\rightarrow$ `master:8095 (Production)`.
2. **Rule 2 (Local Docker Container Verification):** Before pushing to staging, the stack must be built and verified in local Docker containers (`docker compose up --build -d`) to ensure volume mounts and dependencies work in production containers.
3. **Rule 3 (Zero Direct Local Push to Master):** The `master` branch is strictly protected. Code reaches production only via GitHub PR merge from `develop2`.
4. **Rule 4 (Automatic Multi-Agent Delegation):** The orchestrator never writes implementation code directly. It dispatches specialized worker agents (`backend_agent`, `frontend_agent`, etc.) and dual auditors (`research`, `architecture_review_agent`).

---

## 3. Orchestration Tooling & Controls

* **Spawn Subagents:** `invoke_subagent` to spin up workers or auditors.
* **Track Subagents:** `manage_subagents(Action="list")` to view real-time lifecycle states.
* **Stop / Kill Agents:** `manage_subagents(Action="kill", ConversationIds=[...])` to terminate runaway, errored, or blocked tasks.
* **Direct Agents:** `send_message` to pass review feedback or assign iterative fixes.
