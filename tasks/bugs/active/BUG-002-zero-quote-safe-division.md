---
id: BUG-002
type: Bug
title: Zero-Quote & Missing Ticker Safe Division Guard
priority: P3
status: Active
components: [quant-pwa]
impact: Medium
effort: Low
target_release: v1.1.43
assignee: QuantFleetCommander
created: 2026-10-21
---

# [BUG-002] Zero-Quote & Missing Ticker Safe Division Guard

## 🎯 Defect Description
If a ticker in a watchlist or flow aggregation table has a missing, null, or zero spot price (e.g. during pre-market halts or newly listed options symbols), calculating percentage deltas `((current - previous) / previous) * 100` triggers `ZeroDivisionError` or emits `NaN`/`null` to the frontend, causing UI rendering crashes.

## 🔍 Root Cause Analysis
Unprotected division operations in `watchlists.py`, `cockpit.py`, and `flow_aggregate.py` without defensive null checks or epsilon denominators (`or 1e-9`).

## 🛠️ Surgical Remediation Plan
1. Introduce a safe division helper `safe_pct_change(current: float | None, previous: float | None) -> float`:
   ```python
   def safe_pct_change(current: float | None, previous: float | None) -> float:
       if not current or not previous or previous == 0:
           return 0.0
       return round(((current - previous) / previous) * 100, 2)
   ```
2. Apply defensively across all router serialization models.

## ✅ Acceptance Criteria
- [ ] Reproduction test `tests/test_reproduce_zero_quote.py` asserts `0.0` is returned when previous is `0.0` or `None`.
- [ ] All existing router tests pass with zero regressions.
