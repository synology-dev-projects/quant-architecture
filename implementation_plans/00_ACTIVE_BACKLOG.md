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
| 🏛️ **P0** | ARCH-01 | quant-pwa & gexdex-api | **Collapse Gateway + GexDex-API into Unified Modular Monolith**<br>Eliminate internal HTTP/2 network hop, 15ms IPC latency, and dual-container deployment overhead. | **STRATEGIC** (10/10 Architecture) | Med-High | [p0_strategic_architecture/P0_UNIFIED_MODULAR_MONOLITH.md](p0_strategic_architecture/P0_UNIFIED_MODULAR_MONOLITH.md) |
| 🔴 **P1** | PIPE-01 | ibkr-historical-data-pipeline | **IBKR Async Socket Lifecycle & Client ID Leak Prevention**<br>Wrap ib_insync sockets in strict 	ry...finally: ib.disconnect() and randomized client ID fallbacks to prevent cron lockout. | **HIGH** (Reliability) | Low-Med | [p1_high_impact_resilience/P1_IBKR_ASYNC_SOCKET_LIFECYCLE.md](p1_high_impact_resilience/P1_IBKR_ASYNC_SOCKET_LIFECYCLE.md) |
| 🟠 **P2** | OBSV-01 | gexdex-api & quant-pwa | **Centralized Log Aggregation & Prometheus Metrics Exporter**<br>Add /metrics endpoint tracking scrape durations, cache hit ratios, and structured JSON logs. | **MEDIUM** (Observability) | Med | [p2_medium_impact_observability/P2_CENTRALIZED_LOGGING_METRICS.md](p2_medium_impact_observability/P2_CENTRALIZED_LOGGING_METRICS.md) |
| 🟡 **P3** | PIPE-02 | quant-level-pipeline | **Quant Level Email Attachment Remote Fallback Parser**<br>Add pdfplumber/pypdf fallback to automatically extract structured price tables from attached PDF morning reports. | **LOW-MED** (Extensibility) | Low | [p3_low_medium_ingestion/P3_QUANT_LEVEL_PDF_PARSER.md](p3_low_medium_ingestion/P3_QUANT_LEVEL_PDF_PARSER.md) |

---

## 📦 Completed & Merged Milestones (Moved to completed_archive/)
* ✅ **Dynamic Split-Model Router & Tier Classifier** (Released August 2026)
* ✅ **Top 20 Benchmark RAM Pre-Cache Warmer** (Released August 2026)
* ✅ **Single-Flight Parallel Multi-Ticker Batching** (Released August 2026)
* ✅ **Persistent HTTP/2 Connection Pooling** (Released August 2026)
* ✅ **Institutional Bloomberg-Grade UI Emoji Purge** (Released August 2026)
* ✅ **5-Bar Hardware Latency Waterfall Diagnostic HUD** (Released August 2026)
* ✅ **In-Memory Ring Buffer Logging System (GET /api/diagnostics/logs)** (Released August 2026)
* ✅ **Enterprise Architecture Review Agent (1k DAU)** (Released August 2026)
