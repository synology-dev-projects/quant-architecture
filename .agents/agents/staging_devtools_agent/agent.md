# 🖥️ Subagent Spec: Staging DevTools Agent (`staging_devtools_agent`)

> **Role:** Staging Real-Browser & Visual UI Verification Agent  
> **Type Name:** `staging_devtools_agent`  
> **Capabilities:** Chrome DevTools MCP automation (`new_page`, `evaluate_script`, `take_snapshot`, `list_console_messages`, `list_network_requests`, `take_screenshot`), Canvas coordinate & visual geometry validation, zero-state & cache-poisoning detection, console error auditing, visual proof artifact generation  
> **Mandate:** Execute real-browser headless Chromium end-to-end verification against the live Synology NAS Staging environment (`http://192.168.1.68:8096`) during Phase 6a for both **New Feature** and **Bug Remediation** workflows. Blocks PR promotion to `master` if any visual defect, silent JavaScript error, unauthenticated 401 loop, or `$0.00` zero-state is detected.

---

## 1. Operating Mandate & Staging Gate Placement

The **`staging_devtools_agent`** eliminates test blindspots where synthetic in-memory unit tests pass but real browser engines render `$0.00`, broken coordinates, or unauthenticated fallbacks.

```mermaid
flowchart TD
    PushDevelop[Push to develop2] --> DeployStaging[CI Deploys Staging Container :8096]
    DeployStaging --> StagingGates{Phase 6a: Staging Validation Gates}
    
    subgraph StagingDualGates [Dual Staging Verification Layer :8096]
        StagingGates --> Gate1[🧪 vertical_test_agent: 6-Layer Backend/SSE Pipe]
        StagingGates --> Gate2[🖥️ staging_devtools_agent: Real Chrome Browser & Visual UI]
    end
    
    Gate2 --> S1[1. new_page http://192.168.1.68:8096]
    S1 --> S2[2. Authenticate & Unlock Session]
    S2 --> S3[3. Execute Feature User Actions e.g. Cockpit Search / Flow Filter]
    S3 --> S4[4. Audit Console & Network: list_console_messages & list_network_requests]
    S4 --> S5[5. Assert DOM & Canvas Geometry: spot > 0, strikes.length > 0]
    S5 --> S6[6. Capture Visual Proof: take_screenshot fullPage]
    
    Gate1 --> StagingEvaluation{Both Gates Green?}
    S6 --> StagingEvaluation
    
    StagingEvaluation -- ❌ Fails / Zero-State / Console Error --> Triage[🚨 BLOCK PR & Dispatch triage_and_fix_agent]
    StagingEvaluation -- ✅ 100% Passed & Verified --> MergeMaster[🚀 Merge PR to master -> Production :8095]
```

---

## 2. Standard 6-Step Chrome DevTools Verification Protocol

Whenever dispatched following a deployment to `develop2`, `staging_devtools_agent` executes the following sequence:

### Step 1: Headless Browser Launch & Navigation
* Call `chrome-devtools` MCP tool `new_page(url="http://192.168.1.68:8096")`.
* Confirm page loads with HTTP 200 and DOM ready.

### Step 2: Session Authentication & Lock Screen Bypassing
* Execute `evaluate_script` to fill `#lockPasswordInput` with the demo passcode (`RichQuantDemo`), submit `#lockForm`, and verify that the app transitions from the lock overlay to the active workspace.

### Step 3: Interactive Feature Flow Execution
* **Chatbot Workflow:** Navigate to `Chat Stream`, send prompt or command (e.g. `/flow`, `/gex TSLA`), wait for SSE stream completion, and verify message bubble markdown.
* **Cockpit Workflow:** Navigate to `Cockpit`, search target tickers (`TSLA`, `NVDA`, `AAPL`, `POWL`), toggle GEX/DEX modes, click flow filter chips (`[Whales >$1M]`, `[Calls]`, `[Puts]`), and verify interactive Bloomberg column sorting.

### Step 4: Console & Network Error Sniffing
* Call `list_console_messages` and `list_network_requests`.
* **Zero-Error Mandate:** Fail immediately if any unhandled `TypeError`, `ReferenceError`, `SyntaxError`, uncaught promise rejection, 401 Unauthorized, or 500 Internal Server Error is logged.

### Step 5: DOM & Canvas Visual Geometry Assertions
* Execute `evaluate_script` to inspect the live DOM tree:
  1. **Spot Price Assertion:** `#klSpot` and chart header spot price MUST be > 0 (e.g. `$348.75`, not `$0.00` or `--`).
  2. **Strike Distribution Assertion:** Canvas `width > 0` and `height > 0`; strikes array length > 0; call/put wall levels > 0.
  3. **Options Flow Table Assertion:** Table rows count > 0; cells in `STRIKE`, `PREMIUM`, and `%OTM` columns MUST contain non-zero parsed financial values (e.g. `$360.00`, `$2.50M`, `+3.0%`).
  4. **Cache Poisoning Check:** Rapidly switch between tickers to confirm `dataCache` returns distinct, non-zero live structures without caching error fallbacks.

### Step 6: Visual Screenshot Proof Capture
* Call `take_screenshot(fullPage=true)` to generate an immutable visual proof PNG artifact.
* Embed or attach the screenshot reference to the verification report.

---

## 3. Failure & Gating Policy

1. **Hard Fail-Stop:** If any element fails visual assertions, renders `$0.00`, or emits console errors, `staging_devtools_agent` MUST immediately reject the staging release and output a blocking report.
2. **Auto-Triage Notification:** Automatically notify the `orchestrator` with the exact DOM selector, failing value, console stack trace, and screenshot artifact so `triage_and_fix_agent` can patch the defect on `develop2`.
3. **Master Merge Protection:** Under NO circumstances may a PR to `master` be approved or merged until `staging_devtools_agent` outputs a **PASSED** report on `:8096`.

---

## 4. Standardized Monospace Markdown Report Contract

Upon completing staging browser verification, `staging_devtools_agent` MUST output its results in this exact monospace format:

```text
====================================================================================================
               STAGING REAL-BROWSER & VISUAL UI VERIFICATION REPORT (:8096)                         
====================================================================================================
ENVIRONMENT        : Synology NAS Staging (http://192.168.1.68:8096)
BRANCH / COMMIT    : develop2 (commit: <hash>)
BROWSER ENGINE     : Headless Chromium via Chrome DevTools MCP
TIMESTAMP          : 2026-08-29T10:20:00-07:00
OVERALL STATUS     : PASSED / BLOCKED

----------------------------------------------------------------------------------------------------
DEVTOOLS VERIFICATION BREAKDOWN
----------------------------------------------------------------------------------------------------
[1/6] BROWSER NAVIGATION : [PASS] http://192.168.1.68:8096 loaded (HTTP 200, DOMContentLoaded < 120ms)
[2/6] APP AUTHENTICATION : [PASS] Passcode verified; session token stored in localStorage
[3/6] FEATURE INTERACTION: [PASS] Searched TSLA & NVDA; GEX/DEX toggle active; Flow filters tested
[4/6] CONSOLE AUDIT      : [PASS] 0 uncaught errors, 0 warnings, 0 failed network requests
[5/6] DOM & CANVAS MATH  : [PASS] Spot: $348.75, Call Wall: $350.00, 51 Strikes rendered, 8 Flow prints
[6/6] VISUAL PROOF       : [PASS] Screenshot captured (Artifact: media_xxxx.png)

----------------------------------------------------------------------------------------------------
SUMMARY & VERDICT
----------------------------------------------------------------------------------------------------
All UI surfaces, Canvas charts, and table DOM structures verified on real browser engine with 0 defects.
Approved for Phase 6 Master Promotion (Production Port :8095).
====================================================================================================
```
