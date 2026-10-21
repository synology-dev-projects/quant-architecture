# 🏛️ quant-architecture

> **Repository Standard:** Single Source of Truth for As-Built Production Quant System Architecture and Living Contracts.  
> **Hard Invariant:** This repository contains **STRICTLY LIVING PRODUCTION SYSTEM DOCUMENTATION (`docs/`)**. It contains **ZERO EXECUTABLE APPLICATION CODE** and **ZERO SHADOW AGENT / BACKLOG COPIES**.

---

## 🧭 System Metadata & Architecture Hierarchy (SSOT)

To prevent metadata drift across multi-repo development, metadata is partitioned strictly across three tiers:

```
Quant System/ (Workspace Root)
├── 🧠 .agents/                                  # TIER 1: Operational Agent Brain & Skills (SSOT)
│   ├── rules/                                   # Domain rules, protocols & concurrency guards
│   └── skills/                                  # Antigravity skill runbooks (timescale, repro, docker, etc.)
├── 📋 implementation_plans/                     # TIER 2: Active Roadmap & Task Registry (SSOT)
│   ├── 00_ACTIVE_BACKLOG.md                     # Categorized backlog (FEAT-xxx, ENH-xxx, BUG-xxx)
│   ├── p0_... to p3_...                         # Detailed design RFCs
│   └── completed_archive/                       # Historical archived specs
├── 📋 HANDOFF.md                                # Immediate session state & resume prompt (SSOT)
└── 🏛️ quant-architecture/                       # TIER 3: As-Built Production Architecture (This Repo)
    └── 📖 docs/                                 # Living Production Architecture Contracts
        ├── ARCHITECTURE_OVERVIEW.md             # System Microservices Topology Map
        ├── DATABASE_AND_DATA_MODELS.md          # TimescaleDB & Postgres Schemas & Catalog
        ├── ETL_PIPELINES_AND_INGESTION.md       # Pipeline Runbooks, Schedulers & Crons
        ├── APIS_AND_GATEWAYS.md                 # Split-Model Router, SSE & Tool Schemas
        ├── FRONTEND_AND_BOT_APPLICATIONS.md     # PWA Bloomberg UI Standards & Diagnostics HUD
        ├── INFRASTRUCTURE_AND_CICD.md           # Docker Compose, Ports & Synology NAS
        └── MAINTENANCE_COST_AND_MAINTAINABILITY_REVIEW.md
```

---

## 📖 Production Architecture Contracts (`docs/`)

The [`docs/`](docs/) directory is the authoritative specification of the live production system running on the Synology NAS (`:8095`):

1. **[Architecture Overview](docs/ARCHITECTURE_OVERVIEW.md)**: End-to-end service interaction, network routing, and hardware boundaries.
2. **[Database & Data Models](docs/DATABASE_AND_DATA_MODELS.md)**: Relational tables, TimescaleDB hypertables, indexing, and compression policies.
3. **[ETL Pipelines & Ingestion](docs/ETL_PIPELINES_AND_INGESTION.md)**: Daily batch scrapers, rate limits, IBKR connectors, and data freshness gates.
4. **[APIs & Gateways](docs/APIS_AND_GATEWAYS.md)**: FastAPI endpoints, Server-Sent Events (SSE), and MCP tool definitions.
5. **[Frontend & Bot Applications](docs/FRONTEND_AND_BOT_APPLICATIONS.md)**: PWA Bloomberg dark UI specifications, Canvas strike distributions, and caching policies.
6. **[Infrastructure & CI/CD](docs/INFRASTRUCTURE_AND_CICD.md)**: Synology NAS container configuration, memory caps (< 350MB), and GitHub Actions workflows.

---

## 🔄 Production Update Invariant
* This repository is **frozen during everyday development** on `develop`/`develop2`.
* It is updated and synchronized **strictly when code is promoted to Production (`master`)** with explicit human authorization.
