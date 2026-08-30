# 🏆 Active Implementation Backlog & Impact Matrix

> **Target Standard:** L8 Principal Systems Architecture (1,000 DAU Baseline)  
> **Status:** Living Ranked Engineering Backlog  
> **Rule:** ZERO completed tasks are retained in this active register. All completed milestones are moved to completed_archive/.

---

## 📊 Priority & Impact Matrix

`
HIGH IMPACT   │  [P1] IBKR Socket Lifecycle    │  [P0] Unified Modular Monolith
              │  (Operational Failure Risk)    │  (Strategic Latency & Simplicity)
──────────────┼────────────────────────────────┼─────────────────────────────────
LOW/MED IMPACT│  [P3] Quant Level PDF Parser   │  [P2] Prometheus Metrics
              │  (Edge Case Document Ingestion)│  (Centralized Log Store)
              └────────────────────────────────┴─────────────────────────────────
                               LOW EFFORT                       HIGH EFFORT
`

---

## 🎯 Active Ranked Engineering Backlog

| Priority | ID | Component | Summary & Goal | Impact | Effort | Spec Path |
| :---: | :---: | :--- | :--- | :---: | :---: | :--- |
| 🔴 **P1 (Top)** | PIPE-01 | ibkr-historical-data-pipeline | **IBKR Async Socket Lifecycle & Client ID Leak Prevention**<br>Wrap ib_insync sockets in strict `try...finally: ib.disconnect()` and randomized client ID fallbacks to prevent cron lockout. | **HIGH** (Reliability) | Low-Med | [p1_high_impact_resilience/P1_IBKR_ASYNC_SOCKET_LIFECYCLE.md](p1_high_impact_resilience/P1_IBKR_ASYNC_SOCKET_LIFECYCLE.md) |
| 🔴 **P1** | AGENT-01 | quant-pwa (Gateway) | **Quant AI Agent Evaluation & Benchmarking Harness**<br>Automated test harness for evaluating agent tool accuracy, split-model routing, prompt regressions, and synthetic multi-turn stress tests. | **HIGH** (Quality) | Med | `implementation_plans/p1_agent_harness.md` |
| 🟠 **P2** | PWA-01 | quant-pwa (Frontend) | **Quant Levels Terminal Tab & PostgreSQL Price Ladders**<br>Dedicated PWA tab rendering structured key support/resistance levels, daily bounce targets, and spot distance delta from PostgreSQL DB. | **MEDIUM** (UX/Alpha) | Low-Med | `implementation_plans/p2_quant_level_tab.md` |
| 🟡 **P3** | TA-01 | common-lib & quant-pwa | **Technical Analysis (TA) Indicator Engine & Bar Stream**<br>Vectorized TA calculations (RSI, MACD, VWAP, EMA ribbons) over IBKR OHLCV bars with agent tool discovery and chart overlays. | **MEDIUM** (Technical) | Med | `implementation_plans/p3_ta_data_flow.md` |
| 🟡 **P3** | OBSV-01 | gexdex-api & quant-pwa | **Centralized Log Aggregation & Prometheus Metrics Exporter**<br>Add /metrics endpoint tracking scrape durations, cache hit ratios, and structured JSON logs. | **MEDIUM** (Observability) | Med | [p2_medium_impact_observability/P2_CENTRALIZED_LOGGING_METRICS.md](p2_medium_impact_observability/P2_CENTRALIZED_LOGGING_METRICS.md) |
| 🟡 **P3** | PIPE-02 | quant-level-pipeline | **Quant Level Email Attachment Remote Fallback Parser**<br>Add pdfplumber/pypdf fallback to automatically extract structured price tables from attached PDF morning reports. | **LOW-MED** (Extensibility) | Low | [p3_low_medium_ingestion/P3_QUANT_LEVEL_PDF_PARSER.md](p3_low_medium_ingestion/P3_QUANT_LEVEL_PDF_PARSER.md) |

---

## 📦 Completed & Merged Milestones (Moved to completed_archive/)
* ✅ **Quant Levels Database Table Freshness Status Indicator, Manual Sync Button & Semantic v1.0.2 Centralized Versioning (`SETTINGS-06` / `v1.0.2`)** (Released August 2026)
* ✅ **App Version Reset to v1.0.1, Environment-Tagged Badging & Settings Action Button Centering (`SETTINGS-05`)** (Released August 2026)
* ✅ **PostgreSQL 16 Migration & Complete Oracle XE Retirement (`DB-01` / `ORACLE-RETIRE`)** (Released August 2026)
* ✅ **Options Flow Data Freshness Status Card & Conditional Manual Sync (`SETTINGS-04`)** (Released August 2026)
* ✅ **Tri-Mode GEX / DEX / Both Exposure Chart Switcher (`COCKPIT-02`)** (Released August 2026)
* ✅ **Ticker Cockpit 3-Panel Dashboard, Live Search View & Streaming Synergized Tactical Synthesis (`COCKPIT-01`)** (Released August 2026)
* ✅ **Tri-State Column Sorting Engine with Secondary Tie-Breaker and Layer 6 Automated UI Probe (`FLOW-08`)** (Released August 2026)
* ✅ **Streamlined Pure-Data Options Flow Skill with Interactive Paginated Bloomberg Terminal Table UI (`FLOW-07`)** (Released August 2026)
* ✅ **Authentic Bloomberg Terminal Options Flow Table UI with Binary Action Colors and Whale Badges (`UI-02` / `FLOW-04`)** (Released August 2026)
* ✅ **Smart `/flow` Overload for Single-Day Market Flow & Ticker Flow (`FLOW-03`)** (Released August 2026)
* ✅ **Standalone Incremental & Deep Historical Options Flow Scripts, Dynamic Site Discovery, and In-Situ Vertical Tester (`FLOW-02`)** (Released August 2026)
* ✅ **Unusual Options Flow Ingestion Pipeline & AI Agent Tool (`FLOW-01`)** (Released August 2026)
* ✅ **Unified Modular Monolith with In-Process Defensive Circuit Breakers** (Released August 2026)
* ✅ **Dynamic Split-Model Router & Tier Classifier** (Released August 2026)
* ✅ **Top 20 Benchmark RAM Pre-Cache Warmer** (Released August 2026)
* ✅ **Single-Flight Parallel Multi-Ticker Batching** (Released August 2026)
* ✅ **Persistent HTTP/2 Connection Pooling** (Released August 2026)
* ✅ **Institutional Bloomberg-Grade UI Emoji Purge** (Released August 2026)
* ✅ **5-Bar Hardware Latency Waterfall Diagnostic HUD** (Released August 2026)
* ✅ **In-Memory Ring Buffer Logging System (GET /api/diagnostics/logs)** (Released August 2026)
* ✅ **Enterprise Architecture Review Agent (1k DAU)** (Released August 2026)

