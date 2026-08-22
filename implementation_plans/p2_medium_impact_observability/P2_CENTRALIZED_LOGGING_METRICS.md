# 🟠 Priority 2 Implementation Plan: Centralized Log Aggregation & Prometheus Metrics Exporter

## 1. Objective
Add standardized Prometheus metrics endpoint (/metrics) to gateway and gexdex-api to monitor:
- Scrape durations (Histogram)
- RAM pre-cache hit/miss ratio (Counter)
- Active SSE stream connections (Gauge)
- Tool error rates by status code (Counter)

---

## 2. Implementation Specs
1. Add prometheus-client to gateway/requirements.txt.
2. Instrument pp/main.py with make_asgi_app() mounted at /metrics.
3. Format application logs to structured JSON for seamless log ingestion.
