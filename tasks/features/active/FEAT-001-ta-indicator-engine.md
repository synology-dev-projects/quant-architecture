---
id: FEAT-001
type: Feature
title: Technical Analysis Indicator Engine & Bar Stream
priority: P3
status: Active
components: [common-lib, quant-pwa]
impact: Medium
effort: Med
target_release: v1.2.0
assignee: QuantFleetCommander
created: 2026-10-21
---

# [FEAT-001] Technical Analysis (TA) Indicator Engine & Bar Stream

## 🎯 Objective
Provide high-performance, vectorized technical analysis calculations (RSI, MACD, VWAP, EMA ribbons 9/21/50/200) computed over institutional IBKR OHLCV bars, with zero-latency caching, agent tool discovery, and Cockpit chart overlay hooks.

## 📋 User Story
As a quantitative analyst using the Quant PWA, I want real-time and historical technical indicators calculated consistently alongside options microstructure so that the Cockpit AI synthesis can evaluate price momentum and structural confluence together.

## 🏗️ Technical Architecture & Design
1. **Engine Layer (`common-lib/common_lib/analytics/ta_engine.py`)**:
   - Vectorized computation utilizing NumPy / Pandas for:
     - **RSI (14-period)**: Standard Wilders smoothing.
     - **MACD (12, 26, 9)**: Fast EMA, slow EMA, MACD line, signal line, histogram.
     - **VWAP**: Session-anchored volume weighted average price.
     - **EMA Ribbons**: 9, 21, 50, 200 exponential moving averages.
   - Guard against NaN/Inf values on insufficient bar histories (minimum bar thresholds).
2. **API & Gateway (`quant-pwa/gateway/app/routers/technical_analysis.py`)**:
   - Endpoint: `GET /api/ta/{ticker}?timeframe=1d&indicators=rsi,macd,vwap,ema`
   - In-memory 60-second TTL cache for intraday indicators.
3. **Cockpit Synthesis Integration**:
   - Incorporate TA momentum signals into `cockpit.py` deterministic microstructure summary.

## ✅ Acceptance Criteria
- [ ] Unit tests in `common-lib/tests/test_ta_engine.py` covering RSI, MACD, VWAP, EMA calculations against reference values.
- [ ] Sub-20ms calculation latency across 250 daily bars.
- [ ] Clean JSON response from `/api/ta/{ticker}` conforming to Pydantic schemas.
- [ ] Zero trade advice policy: strictly numerical metrics, zero buy/sell triggers.
