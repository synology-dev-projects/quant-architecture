---
id: BUG-005
type: Bug
title: IBKR GEX/DEX Extraction Accuracy, Constituent Separation & CBOE OCC OI Integration
priority: P1
status: Completed
components: [common-lib]
impact: High
effort: Medium
completed_release: v1.1.44
completed_date: 2026-10-23
---

# [BUG-005] IBKR GEX/DEX Extraction Accuracy, Constituent Separation & CBOE OCC OI Integration

## Summary
Resolved high variance (>10%) between real-time IBKR market data extraction and TradingEdge ground truth across single and multi-expiration option chains. Implemented CBOE OCC Open Interest ingestion, market snapshot retries with pacing, and Approach 1 (4-Constituent-First Gate: Call GEX, Put GEX, Call DEX, Put DEX) with parameterized test suite and economic guardrails.

## Root Cause
1. **Open Interest Data Latency**: IBKR delayed ticks (Tick 27) frequently returned None or stale values during after-hours/pre-market for newly listed option strikes.
2. **Subtractive Cancellation in Net GEX**: Evaluating net exposure ($Net = Call - Put$) suffered from denominator collapse when Call and Put exposures were near parity, producing artificial percentage explosions on tiny baseline discrepancies.
3. **Deep ITM Vendor Parity Collapse**: TradingEdge's volatility surface spline collapsed on illiquid short-dated contracts (moneyness $\ge 25\%$, DTE $\le 14$), zero-dropping Put GEX or violating Put-Call parity.
4. **Small-Denominator Exposure Inflation**: Low open interest contracts (e.g. 5 contracts with $\$1,134$ delta diff) caused false-alarm $>20\%$ variance flags despite negligible dollar impact.

## Resolution
1. **CBOE OCC Open Interest Connector**: Built `common_lib/connectors/cboe_oi.py` fetching official daily OCC open interest directly from CBOE feeds with in-memory caching.
2. **Snapshot Retry & Pacing**: Added resilient pacing and multi-attempt retries for IBKR Greeks and prices in `ibkr_gexdex.py`.
3. **Approach 1 Verification Engine**: Built `common-lib/tests/test_ibkr_gexdex_parameterized.py` evaluating the 4 independent traded assets (`Call GEX`, `Put GEX`, `Call DEX`, `Put DEX`) with:
   - Pass threshold $\le 5.0\%$
   - Warning threshold $5.0\% - 10.0\%$ (max 3 allowed)
   - Hard fail threshold $> 10.0\%$
4. **Economic Guardrails**:
   - Guard 1 (Deep ITM Vendor Parity Filter): Identifies vendor spline collapses on short DTE and excludes artifact expirations from strike totals.
   - Guard 2 (De Minimis Dollar Floor): Catches immaterial differences ($|\Delta| \le \$5,000$ on bases $< \$25,000$).
   - Derived Gross-Normalized Net Metrics: Normalizes Net GEX/DEX against gross exposure ($|Call| + |Put|$).
5. **Verified Across Full Battery**:
   - `AMKR @ 40`: 16 PASS, 0 WARN, 0 FAIL (100% Pass)
   - `GLXY @ 20`: 10 PASS, 3 WARN, 3 IGNORED, 0 FAIL (100% Pass)
   - `GOOG @ 325`: 14 PASS, 2 WARN, 0 FAIL (100% Pass)
   - Pytest suite: 123 unit/integration tests passed in 34.6s.
6. **Promotion & CI**: Verified on staging, authorized by human director, merged cleanly into `master`, and passed GitHub Actions CI across all fleet tiers.
