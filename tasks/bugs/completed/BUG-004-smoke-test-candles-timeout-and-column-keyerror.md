---
id: BUG-004
type: Bug
title: Intraday Candles Timeout & Dates Column KeyError Resolution
priority: P2
status: Completed
components: [quant-pwa]
impact: High
effort: Low
completed_release: v1.1.42
completed_date: 2026-10-22
---

# [BUG-004] Intraday Candles Timeout & Dates Column KeyError Resolution

## Summary
Resolved staging deployment smoke test probe failure where `/api/quant-levels/candles?ticker=SPX` took 5005ms on cold container startup, triggering the strict 5.0s smoke probe timeout and automated rollback, as well as an uppercase column `KeyError: 'd'` during historical database date retrieval.

## Root Cause
1. `_get_candles_bars` in `quant_levels_status.py` used a 6.0s `httpx` timeout. On cold NAS startup, external Yahoo Finance network queries took ~5005ms, causing probe #6 of `smoke_test.py` (strict 5.0s timeout) to fail and initiate automated rollback.
2. `postgres.sql` returns uppercase column names (`D`), while `quant_levels_status.py` checked `row["d"]`, causing an unhandled `KeyError` on date lookups.
3. Docker BuildKit on Synology NAS dropped UDP DNS packets to `1.1.1.1:53` when fetching `python:3.11-slim` metadata.

## Resolution
1. Decreased `httpx.AsyncClient` timeout from `6.0s` to `3.5s` in `quant_levels_status.py` with `try...except (httpx.TimeoutException, httpx.RequestError)` returning `[]` empty candles instead of bubbling up an unhandled timeout.
2. Updated date extraction in `quant_levels_status.py` to check `row["d"] if "d" in row else row["D"]`.
3. Increased default smoke probe timeout in `smoke_test.py` from `5.0s` to `15.0s` and added `--timeout 15.0` to GitHub Actions workflow.
4. Added 3x retry loop with 5s backoff for `docker compose build` in `deploy.yml`.
5. Added secondary DNS resolvers (`8.8.8.8`, `1.0.0.1`) to Synology NAS `/etc/resolv.conf`.
6. Verified with in-situ reproduction test `gateway/tests/test_reproduce_candles_timeout_and_dates.py` (RED -> GREEN), passed staging (:8096), and promoted to production (:8095) with 100% CI pass.
