# 🏛️ quant-architecture

> **Repository Standard:** Single Source of Truth for Quant System Architecture, Protocols, Specifications, and Living Documentation.  
> **Hard Rule:** This repository contains **STRICTLY DOCUMENTATION, SCHEMAS, RULES, AND RFC PLANS**. It contains **ZERO EXECUTABLE APPLICATION CODE**.

---

## 🧭 The 4-Pillar Agentic Framework

`	ext
quant-architecture/
├── 🧠 GEMINI.md                                  # PILLAR 1: The Kernel Directives & Invariants
├── 🎯 .agents/
│   ├── rules/                                   # PILLAR 2: Scoped Modular Domain Rules
│   │   ├── feature-development-protocol.md      # 6-Phase Lifecycle & Git gates
│   │   ├── telemetry-and-logging-standards.md   # tr-xxxxxx & ring buffer logs
│   │   ├── oracle-concurrency.md                # UUID staging tables & connection pooling
│   │   ├── api-security.md                      # HMAC session auth & fail-closed security
│   │   ├── bug-remediation-protocol.md          # 4-step bug remediation loop
│   │   └── pipeline-standards.md                # Scraper rate-limits & DTE Greek math
│   └── skills/                                  # PILLAR 4: Context-Isolated Subagents
│       ├── architecture-review-agent/SKILL.md   # Enterprise Systems Architect (1,000 DAU)
│       ├── no-mistakes-reviewer/SKILL.md        # Adversarial Code Reviewer (Phase 5a)
│       └── captain-orchestrator/SKILL.md        # Multi-repo Orchestrator
├── 📖 docs/                                     # PILLAR 3: Just-In-Time Living Contracts
│   ├── APIS_AND_GATEWAYS.md                     # Split-Model Router, SSE & Tool Schemas
│   ├── ARCHITECTURE_OVERVIEW.md                 # System Microservices Topology Map
│   ├── DATABASE_AND_DATA_MODELS.md              # Oracle Schemas & Table Catalog
│   ├── ETL_PIPELINES_AND_INGESTION.md           # Pipeline Runbooks & Crons
│   ├── FRONTEND_AND_BOT_APPLICATIONS.md         # PWA Bloomberg UI Standards & Diagnostics HUD
│   ├── INFRASTRUCTURE_AND_CICD.md               # Docker Compose, Ports & Synology NAS
│   └── MAINTENANCE_COST_AND_MAINTAINABILITY_REVIEW.md
└── 📋 implementation_plans/                     # Ranked Engineering Roadmap (Priority & Impact)
    ├── 00_ACTIVE_BACKLOG.md                     # Master Living Backlog & Effort Matrix
    ├── README.md                                # Implementation Directory Navigator
    ├── p0_strategic_architecture/               # Strategic Architecture Refactors
    │   └── P0_UNIFIED_MODULAR_MONOLITH.md       # Collapse Gateway + GexDex into Monolith
    ├── p1_high_impact_resilience/               # Critical Concurrency Fixes
    │   └── P1_IBKR_ASYNC_SOCKET_LIFECYCLE.md    # Prevent IBKR Client ID Cron Lockouts
    ├── p2_medium_impact_observability/          # Metrics & Telemetry
    │   └── P2_CENTRALIZED_LOGGING_METRICS.md    # Prometheus /metrics Exporter & JSON Logs
    ├── p3_low_medium_ingestion/                 # Extensibility Parsers
    │   └── P3_QUANT_LEVEL_PDF_PARSER.md         # Morning Report PDF Attachment Parser
    └── completed_archive/                       # Zero Completed Items in Active Backlog!
        └── historical_code_reviews/             # Archived Historical Audit Reports
`

---

## 🎯 Active Implementation Roadmap

| Priority | ID | Component | Summary | Impact |
| :---: | :---: | :--- | :--- | :---: |
| 🏛️ **P0** | ARCH-01 | quant-pwa & gexdex-api | **[Unified Modular Monolith](implementation_plans/p0_strategic_architecture/P0_UNIFIED_MODULAR_MONOLITH.md)** | **STRATEGIC** (10/10 Architecture) |
| 🔴 **P1** | PIPE-01 | ibkr-historical-data-pipeline | **[IBKR Socket Lifecycle Context Manager](implementation_plans/p1_high_impact_resilience/P1_IBKR_ASYNC_SOCKET_LIFECYCLE.md)** | **HIGH** (Production Reliability) |
| 🟠 **P2** | OBSV-01 | gexdex-api & gateway | **[Prometheus Exporter & JSON Logs](implementation_plans/p2_medium_impact_observability/P2_CENTRALIZED_LOGGING_METRICS.md)** | **MEDIUM** (Observability) |
| 🟡 **P3** | PIPE-02 | quant-level-pipeline | **[PDF Attachment Morning Report Parser](implementation_plans/p3_low_medium_ingestion/P3_QUANT_LEVEL_PDF_PARSER.md)** | **LOW-MED** (Extensibility) |
