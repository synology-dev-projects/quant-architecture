# 🏛️ System Architecture Overview

## 1. System Vision & Core Value Proposition

The **Quant System** is an end-to-end quantitative options intelligence and automation ecosystem. It bridges institutional market microstructure (Dealer Gamma Exposure, Delta Exposure, Option Flow, Algorithmic Price Levels) with modern generative AI agents, low-latency mobile interfaces, and real-time Discord community alerting.

---

## 2. Component Taxonomy & Responsibilities

| Subsystem | Repository / Directory | Technology Stack | Core Responsibility |
| :--- | :--- | :--- | :--- |
| **Shared Core Library** | `common-lib` | Python 3.13, SQLAlchemy, Pandas, Pydantic, Oracledb | Unified database connectors, dynamic UUID staging tables, connection pooling, push alerts (`ntfy`), and shared mathematical utilities. |
| **Configuration Hub** | `common_config` | YAML, Markdown, Git | Central metadata catalog (`db_catalog.yaml`), steering documents, and environment specs. |
| **Options Analytics API** | `gexdex-api` | FastAPI, ThreadPoolExecutor, Pydantic, Requests | High-concurrency parallel ingestion and calculation engine for Gamma Exposure (GEX), Delta Exposure (DEX), Call/Put Walls, Zero GEX flips, and session pooling. |
| **Quant AI Mobile PWA** | `quant-pwa` | HTML5 Canvas, Vanilla JS, CSS3, FastAPI (Gateway), Gemini SDK | Mobile-first AI chat with sub-300ms SSE streaming, pure client-side HTML5 Canvas options charts, SWR caching, and Model Context Protocol (MCP) server. |
| **Quant Levels ETL** | `quant-level-pipeline` | Python, BeautifulSoup4, Dateutil, Oracle | Scrapes daily quantitative resistance, support, and bounce levels from TradingEdge, parsing structured price ladders into Oracle. |
| **IBKR Tick Ingestion** | `ibkr-historical-data-pipeline` | Python, `ib_insync`, Pandas, Oracle | Connects to Interactive Brokers Gateway to extract historical 1-min, 5-min, and 1-hour OHLCV and volume-weighted average price (WAP) bars. |
| **MM DEX/GEX Pipeline** | `mm-dex-gex-pipeline` | Python, Requests, Pandas, Oracle | Batch pipeline extracting complete market-maker exposure tables across strike ladders and persistence into relational tables. |
| **Unusual Option Flow** | `unusual-option-flow-pipeline` | Python, Requests, Pandas, Oracle | Scans institutional options flow, filtering for unusual Volume/Open Interest ratios and aggressive high-premium blocks (> $100k). |
| **Discord Quant Bot** | `discord-quant-bot` *(Archived)* | Python, `discord.py` | *Decommissioned in favor of mobile Quant PWA frontend.* |

---

## 3. Network Topology & Microservices Map

```mermaid
graph LR
    subgraph LAN_192_168_1_68 [Synology NAS: 192.168.1.68]
        subgraph DockerBridge [Docker Network: quant-system-network]
            GW[quant-gateway :8000]
            FE_PROD[quant-frontend-prod :8095]
            FE_DEV[quant-frontend-dev :8096]
            API_PROD[gexdex-api-prod :8090]
            API_DEV[gexdex-api-dev :8091]
            CFT[quant-cloudflared-prod]
        end
        
        ORA[Oracle DB :1521 / XE]
        IB[IBKR Gateway :4002]
    end

    CFT <== Encrypted Zero-Port Tunnel ==> CFEdge[Cloudflare Edge Network]
    CFEdge <== HTTPS ==> MobileUser[📱 Mobile Phone / 5G]
    
    FE_PROD --> GW
    GW --> API_PROD
    API_PROD --> ORA
    API_PROD --> TE_Cloud[TradingEdge Cloud (Parallel Scrape)]
```

---

## 4. Key Architectural Patterns

1. **Backend-For-Frontend (BFF) Gateway Pattern:**
   The `quant-pwa` Gateway acts as an intelligent aggregator between mobile clients and internal NAS services. It converts 4 sequential high-latency cellular hops into sub-5ms internal Docker network hops.
2. **Idempotent Database Persistence (`MERGE INTO`):**
   All pipeline loaders use staging tables and atomic Oracle `MERGE INTO` statements to ensure that repeated cron executions or historical backfills never produce duplicate primary keys.
3. **Pluggable Model Context Protocol (MCP):**
   The Gateway exposes a standardized JSON-RPC MCP hub allowing AI agents to dynamically discover and execute quant tools (`get_gexdex`, `get_market_status`).
4. **Dynamic Split-Model Router (Router-Worker-Synthesizer Pattern):**
   The Gateway dynamically classifies incoming prompt complexity and history turns to route to the optimal model tier:
   - *Tier 1 Fast Worker (`gemini-3.5-flash-lite`):* Handles high-frequency, single-ticker lookups with sub-1.0s tool determination latency.
   - *Tier 2 Strategic Synthesizer (`gemini-3.7-flash`):* Activated for macro/multi-source queries with a dedicated 512-token Chain-of-Thought thinking budget.
5. **Asymmetric Payload Separation & Client-Side Canvas Charting:**
   Visual strike distributions (40KB JSON arrays) are emitted directly to the client's GPU via native `<canvas>` over SSE, while a compact ~150-token quantitative brief is sent to Gemini, slashing token costs by 90% and eliminating LLM hallucination.
6. **Top 20 RAM Pre-Cache Warmer & Persistent HTTP/2 Pooling:**
   A background market-hours asyncio cron pre-warms the 20 most liquid benchmark tickers in RAM every 3 minutes, yielding `0.0ms` responses for >95% of user traffic while persistent HTTP/2 connection pooling reuses TCP sockets for cold scrapes.
7. **In-Memory Ring Buffer Remote Logging & Distributed Tracing:**
   Every request is correlated by a unique 6-character trace ID (`tr-xxxxxx`). The Gateway stores the last 200 logs in an in-memory ring buffer accessible via `GET /api/diagnostics/logs`, enabling real-time remote debugging directly from DSM containers without SSH access.
8. **Pure Client-Side HTML5 Canvas Charting:**
   Visual strike distributions and dual-sided gamma/delta bars are rendered entirely on the client's GPU via native `<canvas>`, eliminating server-side rendering latency and image transmission bandwidth.
9. **Rule 5 Strict Completeness Invariant:**
   Guarantees that every ticker requested in a batch query is explicitly accounted for in the output breakdown table with zero dropped rows.
