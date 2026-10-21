---
id: FEAT-002
type: Feature
title: Macroeconomic Events RAG Retrieval & Synthesis
priority: P3
status: Active
components: [economic-events-pipeline, quant-pwa]
impact: Medium
effort: Med
target_release: v1.2.0
assignee: QuantFleetCommander
created: 2026-10-21
---

# [FEAT-002] Macroeconomic Events RAG Retrieval & Synthesis

## 🎯 Objective
Incorporate a Retrieval-Augmented Generation (RAG) system for macroeconomic awareness (e.g. FOMC rate decisions, CPI releases, Non-Farm Payrolls, Treasury auctions, flash PMIs) that injects timely macro event context directly into Cockpit synthesis and market reports.

## 📋 User Story
As an options trader, I want the Cockpit synthesis to be immediately aware of upcoming high-impact macroeconomic events and recent releases so that volatility spikes and binary risk windows are accounted for.

## 🏗️ Technical Architecture & Design
1. **Pipeline Extraction (`economic-events-pipeline`)**:
   - Ingests weekly/daily economic events from structured calendar feeds with failover and disk caching.
   - Categorizes impact tiers (`HIGH`, `MEDIUM`, `LOW`, `HOLIDAY`).
   - Generates pre-formatted RAG-ready chunk summaries (`synthetic_summary`) ready for embedding.
2. **Storage (`TimescaleDB / Postgres`)**:
   - Idempotently upserts records into `economic_events` table with composite primary key `(event_id, event_timestamp)`.
3. **Retrieval & RAG (`quant-pwa/gateway/app/core/macro_rag.py`)**:
   - Query macro events within $\pm 48\text{h}$ of current session.
   - Filter by high-impact catalysts and country (USD, EUR, GBP, JPY).
   - Inject concise 2-sentence macro context snippet into Cockpit prompt.

## ✅ Acceptance Criteria
- [ ] Automated tests in `economic-events-pipeline/tests/test_pipeline.py` passing with 100% mocked fixtures.
- [ ] Deduplication key guarantees zero duplicate rows on repeated cron runs.
- [ ] Gateway endpoint `GET /api/economic-events` operational with impact tier filters.
- [ ] Pre-computed synthetic summaries formatted without narrative fluff.
