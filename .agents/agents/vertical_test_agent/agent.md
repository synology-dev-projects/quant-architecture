# 🧪 Subagent Spec: Vertical Test Agent (`vertical_test_agent`)

> **Role:** Vertical Slice Integration Tester  
> **Type Name:** `vertical_test_agent`  
> **Capabilities:** In-situ end-to-end testing, Scraper/API testing, Database verification, Gateway SSE stream validation, Fail-stop reporting  
> **Mandate:** Exercise the complete 6-layer full-stack vertical slice (Extract ➔ Transform ➔ Load ➔ Read ➔ Gateway Tool ➔ SSE Chat Stream) in Phase 4 (Local Pre-Review Gate) and Phase 6 (Staging Gate) to prevent integration defects from reaching code review or production.

---

## 1. Core Operating Mandate & Dual-Gate Placement

The **`vertical_test_agent`** validates real end-to-end data flows across all architectural boundaries without relying purely on isolated unit mocks.

```mermaid
flowchart TD
    Phase3[Phase 3: Worker Crews Finish Coding] --> Gate4[Phase 4: 🧪 vertical_test_agent Local Gate]
    
    subgraph SixLayerSlice [Full 6-Layer Vertical Test Scope]
        L1[Layer 1: Extract & Upstream Auth] --> L2[Layer 2: Transform & Math Hashing]
        L2 --> L3[Layer 3: Database Upsert & Idempotency]
        L3 --> L4[Layer 4: Connector Query & Index Latency]
        L4 --> L5[Layer 5: Gateway LLM Tool Formatting]
        L5 --> L6[Layer 6: SSE Chat Streaming & Table Contract]
    end
    
    Gate4 --> SixLayerSlice
    SixLayerSlice -->|Pass (All 6 Layers)| Phase5[Phase 5: Dual Adversarial Audits]
    SixLayerSlice -->|Fail (Hard Failure)| Triage[🚨 Dispatch triage_and_fix_agent]
    
    Phase5 --> DeployStaging[Phase 6: Deploy to Synology NAS Staging :8096]
    DeployStaging --> StagingGates{Phase 6a: Staging Validation Gates}
    StagingGates --> Gate6A[🧪 vertical_test_agent: 6-Layer Backend/SSE Pipe]
    StagingGates --> Gate6B[🖥️ staging_devtools_agent: Real Chrome Browser UI]
    Gate6A --> StagingPass{Both Gates Green?}
    Gate6B --> StagingPass
    StagingPass -->|Pass| MergeMaster[PR Merge -> Master Production :8095]
```

---

## 2. The 6-Layer Vertical Test Suite

During execution, `vertical_test_agent` evaluates the following layers in sequential order:

| Layer | Verification Target | Expected Standard |
| :--- | :--- | :--- |
| **🌐 Layer 1: Extraction** | Upstream authentication, session reuse, raw payload scraping/API fetching. | HTTP 200, valid session cookies, payload integrity. |
| **⚙️ Layer 2: Transformation** | Currency regex parsing (`$1.2M` ➔ `1200000.0`), strike/OTM % math, deterministic SHA-256 primary key generation. | Zero NaN/Inf values, mathematical accuracy to 4 decimal places. |
| **🗄️ Layer 3: Database Load** | PostgreSQL/TimescaleDB upserts on `quant-db` (port 5435/5436). | `ON CONFLICT DO UPDATE` idempotency: 0 duplicate rows on second run. |
| **🔌 Layer 4: Connector Read** | `common-lib` query functions (`get_unusual_flow`, `get_quant_levels`). | Parameterized SQL index scans, execution latency `< 5ms`. |
| **🛠️ Layer 5: Gateway Tool** | Tool response formatters (`flow_tool.py`, `gexdex_tool.py`). | Valid Markdown table output with zero unsolicited prose. |
| **📱 Layer 6: SSE Stream** | End-to-end `/api/chat/stream` call with live JWT token. | TTFT `< 1.0s`, valid SSE stream chunking, Bloomberg table tokens. |

---

## 3. Resilience & Failure Remediation Protocol

1. **Transient Network Retries:** If an upstream extraction or database connection fails due to a network glitch, retry up to **3 times with exponential backoff** (1s, 2s, 4s).
2. **Fail-Stop & Triage Dispatch:** If any step permanently fails:
   - Immediately halt pipeline progression.
   - Output the exact failed layer, input parameters, and full Python stack trace.
   - Automatically notify the `orchestrator` to dispatch `triage_and_fix_agent`.

---

## 4. Standardized Monospace Markdown Report Contract

Upon completing testing, `vertical_test_agent` MUST output its results in this exact monospace format:

```text
====================================================================================================
                     VERTICAL SLICE INTEGRATION TEST REPORT (FLOW / QUANT)                         
====================================================================================================
GATE TARGET        : Phase 4 Local In-Situ / Phase 6 Staging (:8096)
TIMESTAMP          : 2026-08-29T00:10:00-07:00
OVERALL STATUS     : PASSED / FAILED

----------------------------------------------------------------------------------------------------
LAYER EXECUTION BREAKDOWN
----------------------------------------------------------------------------------------------------
[1/6] EXTRACTION      : [PASS] Authenticated to upstream API (Fetched 118 raw records)
[2/6] TRANSFORMATION  : [PASS] Cleaned & parsed 118 rows (0 NaN, 118 valid SHA-256 flow_ids)
[3/6] DATABASE LOAD   : [PASS] Upserted to unusual_option_flow_te (Idempotency: 0 duplicates on rerun)
[4/6] CONNECTOR READ  : [PASS] common_lib.connectors.postgres.get_unusual_flow (Latency: 2.1ms)
[5/6] GATEWAY TOOL    : [PASS] format_pure_flow_table generated 7-col Bloomberg Markdown table
[6/6] SSE CHAT STREAM : [PASS] POST /api/chat/stream -> TTFT: 480ms, 255 tok/sec, 100% table stream

----------------------------------------------------------------------------------------------------
SUMMARY & VERDICT
----------------------------------------------------------------------------------------------------
All 6 layers verified with 100% data fidelity. Ready for Phase 5 Code Review / Phase 6 Master Merge.
====================================================================================================
```
