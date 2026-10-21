---
id: ENH-001
type: Enhancement
title: Watchlist Live 5s Quote & Delta Polling Feed
priority: P2
status: Active
components: [quant-pwa]
impact: High
effort: Low
target_release: v1.1.43
assignee: QuantFleetCommander
created: 2026-10-21
---

# [ENH-001] Watchlist Live 5s Quote & Delta Polling Feed

## 🎯 Objective
Provide live, zero-cost 5-second spot price and percentage up/down delta updates for all tickers displayed in the Quant PWA Watchlists tab, backed by an in-memory batch cache and browser tab-visibility pausing.

## 📋 User Story
As a user monitoring my custom watchlists, I want spot prices and daily percentage changes to update dynamically every 5 seconds so that I can see market movements without manual page refreshes.

## 🏗️ Technical Architecture & Design
1. **Backend Batch Fetcher (`quant-pwa/gateway/app/core/quote_feed.py`)**:
   - Single batch query endpoint supporting up to 50 tickers per call.
   - Public multi-quote API with fallback pricing.
   - 4-second in-memory RAM cache (`_quote_cache: dict[str, tuple[float, QuoteData]]`) to prevent rate-limit exhaustion.
2. **Gateway Router (`quant-pwa/gateway/app/routers/watchlists.py`)**:
   - `GET /api/watchlists/{id}/quotes` returning `{ticker: {price: float, change: float, change_percent: float, timestamp: str}}`.
3. **Frontend Client (`quant-pwa/frontend/src/tabs/watchlist_view.js`)**:
   - 5-second polling interval using `setInterval`.
   - Automatic pause/resume bound to `document.addEventListener('visibilitychange', ...)` and tab switching.
   - Color flashes on delta changes: Bloomberg green (`#10b981`) for up, red (`#ef4444`) for down.

## ✅ Acceptance Criteria
- [ ] Backend unit tests in `gateway/tests/test_quote_feed.py` passing 100%.
- [ ] Polling stops when tab is backgrounded (`document.hidden == true`).
- [ ] Memory footprint inside Gateway stays $< 10\text{MB}$.
- [ ] Touch targets and layouts pass `audit_layout.js` without regressions.
