---
id: FEAT-PWA-02
type: Feature
title: Cockpit GEX/DEX Data Source Switcher (TradingEdge & IBKR)
priority: P2
status: Completed
components: [quant-pwa]
impact: High
effort: Medium
completed_release: v1.1.45
completed_date: 2026-10-29
---

# [FEAT-PWA-02] Cockpit GEX/DEX Data Source Switcher (TradingEdge & IBKR)

## Summary
Integrated dual-source options exposure analysis within the Cockpit GEX/DEX graph component in Quant PWA. Added a responsive `[ TradingEdge | IBKR ]` data source toggle in the chart panel header alongside `[ Both | Net GEX | Net DEX ]`. The Gateway backend concurrently queries and caches live IBKR CBOE delayed market data with vectorized analytical BSM Greeks re-evaluation (`recompute_dynamic_bsm_gexdex`) with a 15-minute in-memory cache, ensuring instantaneous 0ms client-side switching, identical 25-strike distribution schema, and dynamic Key Levels updates (`Spot Price`, `Zero Flip`, `Call Wall`, `Put Wall`).

## Verification
- Gateway router & service tests: 3/3 tests passing (`test_cockpit_ibkr_tab.py`).
- Frontend regression test suite: 99/99 DOM assertions passing (`test_cockpit_view.js`).
- Chrome DevTools MCP UI audit: Verified zero horizontal/vertical layout overflow on desktop (1280px) and mobile (375px).
- Fleet CI Run `40813587669` passed cleanly on `develop2` and verified on Staging port `:8096`.
- Promoted and deployed cleanly to `master` (Production port `:8095`).
