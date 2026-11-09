---
id: BUG-011
title: 4-Pipeline Scheduled Execution Parity and Staleness Remediation
category: bugs
status: Completed
priority: P1
release: v1.2.3
completed_date: 2026-11-09
components:
  - common-lib
  - market-confluence-pipeline
  - unusual-option-flow-pipeline
  - quant-pwa
---

# BUG-011: 4-Pipeline Scheduled Execution Parity and Staleness Remediation

## 1. Defect Description
Pipelines on the Quant Level / Settings page were consistently reporting "Not Up to Date" / "Not Run". Market Confluence was frozen without new records since 2026-10-29.

## 2. Root Causes Identified
1. **Status Router Date Mismatch:** `pipelines_router.py` evaluated `last_market_day` (yesterday) for `quant_levels` during active morning trading hours, while `quant_levels_status.py` expected today's date after 06:30 ET.
2. **Confluence Flow Target Date Staleness:** `market-confluence-pipeline/src/pipeline/collector.py` strictly queried `trade_date = :target_date`. If TradingEdge flow was delayed overnight, it returned 0 rows and silently bailed without running.
3. **DAG Executor Session Date Mismatch:** `common-lib/common_lib/orchestration/executor.py` evaluated yesterday's date for all pipelines in the morning cycle, mismatching morning quant levels.

## 3. Resolution & Verification
- Added dynamic date resolution in `executor.py` and `pipelines_router.py` for morning trading sessions with table data fallbacks.
- Added graceful fallback to `get_latest_flow_date(engine)` in `collector.py`.
- Verified with reproduction test `test_reproduce_pipeline_scheduling_and_staleness.py` (`exit code 0`, 2/2 passed).
- Confirmed Synology NAS Task Scheduler ID 5 runs `Prefect Daily Market Cycle` daily at 06:30 AM PST / 09:30 AM EST.
- Deployed to `master` across all affected repositories.
