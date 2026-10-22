# 📋 Quant System Master Task Board (Jira Architecture)

> **Standard:** L8 Principal Systems Architecture  
> **Repository:** `quant-architecture/tasks/`  
> **Status:** Living Master Task Board  
> **Convention:** All tasks are categorized by **Category** (`features/`, `enhancements/`, `bugs/`) and partitioned by **Lifecycle State** (`active/`, `completed/`).

---

## 📊 Summary Metrics

| Category | 🚀 Features | ⚡ Enhancements | 🛡️ Bugs | Total |
| :--- | :---: | :---: | :---: | :---: |
| **Active / Backlog** | 3 | 3 | 2 | **8** |
| **Completed / Done** | 5 | 5 | 3 | **13** |
| **Total Tracked** | 8 | 8 | 5 | **21** |

---

## 🚀 Active Sprint & In-Flight Priorities

| Priority | ID | Category | Summary & Scope | Components | Target | Ticket Spec |
| :---: | :---: | :---: | :--- | :--- | :---: | :--- |
| 🟠 **P2** | `ENH-001` | ⚡ Enhancement | **Watchlist Live 5s Quote & Delta Polling Feed**<br>Batch quote fetcher with 4s RAM cache, live price markers, % change delta, and tab-blur auto-pause. | `quant-pwa` | `v1.1.43` | [ENH-001](enhancements/active/ENH-001-watchlist-live-quotes.md) |
| 🟡 **P3** | `FEAT-001` | 🚀 Feature | **Technical Analysis (TA) Indicator Engine & Bar Stream**<br>Vectorized calculations (RSI, MACD, VWAP, EMA ribbons) over IBKR OHLCV bars. | `common-lib`, `quant-pwa` | `v1.2.0` | [FEAT-001](features/active/FEAT-001-ta-indicator-engine.md) |
| 🟠 **P2** | `BUG-001` | 🛡️ Bug Fix | **NAS Staging Container Auto-Lifecycle & Permission Guard**<br>Non-root docker socket orchestration / automated launch script on Synology NAS for `:8096`. | `nas-infra`, `quant-pwa` | `v1.1.43` | [BUG-001](bugs/active/BUG-001-nas-staging-lifecycle.md) |

---

## 📋 Active Backlog

### 1. 🚀 New Features (`features/active/`)
| Priority | ID | Summary & Scope | Components | Impact | Effort | Ticket Spec |
| :---: | :---: | :--- | :--- | :---: | :---: | :--- |
| 🟡 **P3** | `FEAT-001` | Technical Analysis (TA) Indicator Engine & Bar Stream | `common-lib`, `quant-pwa` | Medium | Med | [FEAT-001](features/active/FEAT-001-ta-indicator-engine.md) |
| 🟡 **P3** | `FEAT-002` | Macroeconomic Events RAG Retrieval & Synthesis | `economic-events-pipeline`, `quant-pwa` | Medium | Med | [FEAT-002](features/active/FEAT-002-economic-events-rag.md) |
| 🟢 **P4** | `FEAT-003` | SEC Filings 10-K/10-Q Material Risk Factor Ingestion | `sec-filings-pipeline`, `quant-pwa` | Low-Med | Med | [FEAT-003](features/active/FEAT-003-sec-filings-risks.md) |

### 2. ⚡ Enhancements (`enhancements/active/`)
| Priority | ID | Summary & Scope | Components | Impact | Effort | Ticket Spec |
| :---: | :---: | :--- | :--- | :---: | :---: | :--- |
| 🟠 **P2** | `ENH-001` | Watchlist Live 5s Quote & Delta Polling Feed | `quant-pwa` | High | Low | [ENH-001](enhancements/active/ENH-001-watchlist-live-quotes.md) |
| 🟡 **P3** | `ENH-002` | Quant Level Email Attachment PDF Fallback Parser | `quant-level-pipeline` | Low-Med | Low | [ENH-002](enhancements/active/ENH-002-quant-level-pdf-parser.md) |
| 🟢 **P4** | `ENH-003` | Pipeline Telemetry & Prefect Flow Health Modal | `quant-pwa` | Low | Low | [ENH-003](enhancements/active/ENH-003-pipeline-telemetry-modal.md) |

### 3. 🛡️ Bug Fixes & Resilience Hardening (`bugs/active/`)
| Priority | ID | Summary & Scope | Components | Impact | Effort | Ticket Spec |
| :---: | :---: | :--- | :--- | :---: | :---: | :--- |
| 🟠 **P2** | `BUG-001` | NAS Staging Container Auto-Lifecycle & Permission Guard | `nas-infra`, `quant-pwa` | High | Low | [BUG-001](bugs/active/BUG-001-nas-staging-lifecycle.md) |
| 🟡 **P3** | `BUG-002` | Zero-Quote & Missing Ticker Safe Division Guard | `quant-pwa` | Medium | Low | [BUG-002](bugs/active/BUG-002-zero-quote-safe-division.md) |

---

## ✅ Completed Milestones & Done Archive

### 1. 🚀 Completed Features (`features/completed/`)
| ID | Title & Release Summary | Completed Release | Date | Ticket Spec |
| :---: | :--- | :---: | :---: | :--- |
| `FEAT-PWA-01` | Quant Levels Terminal Tab & SPX Price Ladders | `v1.1.20` | 2026-09-15 | [FEAT-PWA-01](features/completed/FEAT-PWA-01-quant-levels-tab.md) |
| `FEAT-MCP-01` | Polymorphic Dual-Transport (POST & GET) MCP Server | `v1.1.18` | 2026-09-08 | [FEAT-MCP-01](features/completed/FEAT-MCP-01-polymorphic-transport.md) |
| `FEAT-AGENT-01`| Quant AI Agent Evaluation & Benchmarking Harness | `v1.1.15` | 2026-09-02 | [FEAT-AGENT-01](features/completed/FEAT-AGENT-01-eval-benchmark-harness.md) |
| `FEAT-FLOW-01` | Unusual Options Flow Ingestion Pipeline & AI Agent Tool | `v1.1.0` | 2026-08-25 | [FEAT-FLOW-01](features/completed/FEAT-FLOW-01-options-flow-pipeline.md) |
| `FEAT-DB-01`   | PostgreSQL 16 + TimescaleDB Migration & Oracle Deprecation | `v1.0.0` | 2026-08-20 | [FEAT-DB-01](features/completed/FEAT-DB-01-timescaledb-migration.md) |

### 2. ⚡ Completed Enhancements (`enhancements/completed/`)
| ID | Title & Release Summary | Completed Release | Date | Ticket Spec |
| :---: | :--- | :---: | :---: | :--- |
| `ENH-OBSV-01`  | Centralized Log Aggregation & Prometheus Metrics Exporter | `v1.1.25` | 2026-09-18 | [ENH-OBSV-01](enhancements/completed/ENH-OBSV-01-prometheus-metrics-exporter.md) |
| `ENH-DEPLOY-03`| CI-Driven Automated Living Documentation & Architecture Sync | `v1.1.24` | 2026-09-17 | [ENH-DEPLOY-03](enhancements/completed/ENH-DEPLOY-03-living-docs-ci-sync.md) |
| `ENH-DEPLOY-02`| Git Worktree Isolation for Parallel Subagent Fleet | `v1.1.22` | 2026-09-12 | [ENH-DEPLOY-02](enhancements/completed/ENH-DEPLOY-02-git-worktree-isolation.md) |
| `ENH-PIPE-01`  | IBKR Async Socket Lifecycle & Client ID Leak Prevention | `v1.1.12` | 2026-08-30 | [ENH-PIPE-01](enhancements/completed/ENH-PIPE-01-ibkr-socket-lifecycle.md) |
| `ENH-FLOW-03`  | Smart `/flow` Overload for Single-Day & Ticker Flow | `v1.1.8` | 2026-08-28 | [ENH-FLOW-03](enhancements/completed/ENH-FLOW-03-smart-flow-overload.md) |

### 3. 🛡️ Completed Bug Fixes (`bugs/completed/`)
| ID | Title & Release Summary | Completed Release | Date | Ticket Spec |
| :---: | :--- | :---: | :---: | :--- |
| `BUG-003`       | Pipeline Upstream Dependency Synchronization & Table Freshness Fallback | `v1.1.42` | 2026-10-21 | [BUG-003](bugs/completed/BUG-003-pipeline-upstream-dependency-sync.md) |
| `BUG-DEPLOY-01`| Automated Health-Gate & Auto-Rollback in Synology Pipeline | `v1.1.19` | 2026-09-14 | [BUG-DEPLOY-01](bugs/completed/BUG-DEPLOY-01-auto-rollback-health-gate.md) |
| `BUG-FLOW-EDGE`| Options Flow Passcode Auth & .EDGE Verification Gate | `v1.1.40` | 2026-10-18 | [BUG-FLOW-EDGE](bugs/completed/BUG-FLOW-EDGE-auth-passcode-verification.md) |

---

## 📝 Lifecycle Workflow Rules
1. **New Task Intake**: Create a markdown file under `tasks/{features, enhancements, bugs}/active/{ID}-{name}.md` with standard YAML frontmatter. Add row to `BOARD.md`.
2. **Execution**: Link the ticket ID in the commit message (e.g. `feat(watchlist): [ENH-001] add live quote polling`).
3. **Completion**: When promoted to `master`, move the ticket from `active/` to `completed/`, update `status: Completed`, and move row to the Done table in `BOARD.md`.
