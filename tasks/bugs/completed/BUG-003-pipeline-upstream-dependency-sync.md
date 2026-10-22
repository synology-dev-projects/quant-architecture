---
id: BUG-003
type: Bug
title: Pipeline Upstream Dependency Synchronization & Table Freshness Fallback
priority: P2
status: Completed
components: [common-lib, market-confluence-pipeline, gexdex-snapshot-pipeline, quant-pwa]
impact: High
effort: Low
completed_release: v1.1.42
completed_date: 2026-10-21
---

# [BUG-003] Pipeline Upstream Dependency Synchronization & Table Freshness Fallback

## Summary
Resolved DAG orchestration dependency blocking where downstream pipelines (such as `market_confluence`) were blocked when upstream pipelines (`unusual_option_flow`, `gexdex_snapshot`) ran standalone and populated their target database tables without writing an explicit entry to `pipeline_runs`.

## Root Cause
`common_lib.orchestration.state.check_upstream_dependencies()` strictly checked only `pipeline_runs` table for `status == 'SUCCESS'`. Standalone cron executions wrote directly to application tables (`unusual_option_flow_te`, `gexdex_snapshot`) but bypassed central run tracking, causing `check_upstream_dependencies()` to return `is_ready=False`.

## Resolution
1. Added target table data freshness inspection in `common_lib/orchestration/state.py` (`_check_table_has_data_for_date()`). If `pipeline_runs` has no record for `session_date`, it verifies whether the upstream's target tables contain records for that date and auto-backfills `pipeline_runs`.
2. Aligned dry-run simulation in `common_lib/orchestration/executor.py` with `check_upstream_dependencies()`.
3. Added lifecycle recording (`record_run_start`, `record_run_success`) to `daily_snapshot.py` and `daily_incremental.py`.
4. Verified via in-situ reproduction test `gateway/tests/test_reproduce_upstream_fallback.py` (RED -> GREEN) and verified 100% CI pass across all 8 fleet repositories on Staging (`develop2`) and Production (`master`).
