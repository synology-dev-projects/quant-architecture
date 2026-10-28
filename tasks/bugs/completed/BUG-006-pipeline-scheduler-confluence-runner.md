---
id: BUG-006
title: Prefect 3 Ephemeral Server Timeout & Multi-Repo Runner Namespace Collision
category: bugs
status: completed
priority: P1
components:
  - common-lib
  - quant-pwa
  - nas-infra
created_date: 2026-10-28
completed_date: 2026-10-28
release: v1.1.45
---

# BUG-006: Prefect 3 Ephemeral Server Timeout & Multi-Repo Runner Namespace Collision

## 1. Problem Statement
1. **Scheduled Pipeline Failure:** Daily cron job at 06:30 PDT failed to run pipelines because Prefect 3 ephemeral server startup timed out after 20 seconds (`Timed out while attempting to connect to ephemeral Prefect API server http://127.0.0.1:8268`).
2. **Market Confluence Runner Failure:** On manual invocation, `market_confluence` failed with `ModuleNotFoundError: No module named 'src.scripts.daily_incremental'` due to Python's `sys.modules['src']` caching collision from the prior `gexdex-snapshot-pipeline` run. In addition, the gateway container was missing the volume mount for `market-confluence-pipeline`.

## 2. Root Cause Analysis
- **Prefect Startup:** Under Synology NAS I/O load, ephemeral SQLite/uvicorn API initialization exceeded the default 20s timeout without a fallback in `quant_pipeline_ctl.py`.
- **Namespace Collision:** Multiple pipeline repositories share `src` as their top-level package name (`gexdex-snapshot-pipeline/src`, `market-confluence-pipeline/src`). When `gexdex_snapshot` ran first, `sys.modules['src']` pointed to `gexdex-snapshot-pipeline/src`, preventing subsequent `market_confluence` imports from locating `src.scripts.daily_incremental`.
- **Docker Compose:** `quant-pwa/docker-compose.yml` mounted `gexdex-snapshot-pipeline` and `quant-level-pipeline`, but lacked `${CONFLUENCE_PIPELINE_PATH:-../market-confluence-pipeline}:/app/market-confluence-pipeline:ro`.

## 3. Surgical Fix Implemented
1. **Dynamic Namespace Isolation:** In `common-lib/common_lib/orchestration/registry.py`, updated `resolve_runner()` to dynamically purge foreign `src` and `src.*` modules from `sys.modules` and prepend the target candidate repo to `sys.path[0]` before dynamic module loading.
2. **Container Volume Mount:** Added `${CONFLUENCE_PIPELINE_PATH:-../market-confluence-pipeline}:/app/market-confluence-pipeline:ro` to `quant-gateway` in `quant-pwa/docker-compose.yml`.
3. **NAS Resilient Execution:** In `/volume2/homes/rachardv/scripts/quant_pipeline_ctl.sh`, configured `PREFECT_SERVER_EPHEMERAL_STARTUP_TIMEOUT_SECONDS=60` and `PREFECT_CLIENT_MAX_RETRIES=5`. In `/volume2/homes/rachardv/scripts/quant_pipeline_ctl.py`, added automated in-process `run_dag_cycle()` fallback if Prefect flow initialization times out.

## 4. Verification & Proof
- **RED Gate:** `quant-pwa/gateway/tests/test_reproduce_runner_resolution.py` failed with exit code 1 replicating the namespace collision.
- **GREEN Gate:** Re-executed exact test with exit code 0.
- **Staging Verification:** Deployed to `develop2` (staging port `:8096`). Live in-situ execution in `quant-gateway-dev` verified clean table initialization and execution.
- **Production Gate:** Topologically promoted to `master` across `common-lib` and `quant-pwa`.
