---
id: BUG-010
title: Build Version Increment & Parity Automation Life-Cycle
category: bugs
status: completed
priority: P2
completed_release: v1.2.1
completed_date: 2026-10-30
assignee: QuantFleetCommander
components:
  - quant-pwa
---

# BUG-010: Build Version Increment & Parity Automation Life-Cycle

## 1. Problem Description
The daily build number (`YYYY-MM-DD-NN`) in Quant PWA remained stuck at an old date (`2026-10-28-01`) across releases because:
1. `bump_version.py` only checked version parity during pre-commit hooks, with no automated trigger running during `promote-prod`.
2. `test_reproduce_build_version.py` hardcoded `TARGET_VERSION = "v1.1.45"`, blocking any automated incrementation from passing tests.
3. `config.py` and `settings_modal.js` did not expose or render the dynamic daily build number string alongside semantic versions.

## 2. Root Cause
- Missing pipeline hook to auto-execute `bump_version.py` upon promotion to `master`.
- Hardcoded frozen test assertion in `test_reproduce_build_version.py`.
- Missing `APP_BUILD` and `CLIENT_BUILD` synchronization.

## 3. Resolution
- Updated `scripts/bump_version.py` to synchronize `APP_BUILD` in `gateway/app/config.py` and `CLIENT_BUILD` in `frontend/src/components/settings_modal.js`.
- De-hardcoded `gateway/tests/test_reproduce_build_version.py` to read dynamically from `version.json`.
- Integrated automated version bump into `cmd_promote_prod()` in `scripts/protocol_graph.py`.
- Formatted Settings Modal UI to display `${CLIENT_VERSION} (${CLIENT_BUILD}) [Environment]`.
- Verified container-safe resolution in `test_reproduce_build_increment_lifecycle.py`.

## 4. Verification Evidence
- Staging (`:8096`): Verified `v1.2.0 (2026-10-30-01) [Staging]`.
- Production (`:8095`): Verified `v1.2.1 (2026-10-30-02) [Production]`.
- Full pytest suite (290 tests) passing with exit code 0.
