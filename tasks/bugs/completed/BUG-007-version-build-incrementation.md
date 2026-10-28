---
id: BUG-007
category: bug
status: completed
priority: P2
target_release: v1.1.42
date_completed: 2026-10-28
components:
  - quant-pwa
---

# BUG-007: Version Rollover at 10 & Daily Build Number Auto-Incrementation

## 1. Problem Description
- `quant-pwa/scripts/bump_version.py` simply performed `int(patch) + 1` without rollover, allowing patch numbers to grow indefinitely (e.g. `v1.1.42`). When the patch number reaches 10 (or when starting from 10, e.g. `2.1.10`), it must roll over to the minor version (`2.2.0`), with cascading rollover to major if minor reaches 10.
- Build numbers in `quant-pwa/version.json` were static (`2026-09-24-01`) because `bump_version.py` only updated `"version"` and `"updated_at"`.

## 2. Root Cause
- Missing base-10 modulo/rollover logic in `increment_patch_version()`.
- Omission of `"build"` property mutation in `update_all_files()`.

## 3. Surgical Fix
- Updated `increment_patch_version()` with base-10 rollover and cascading minor rollover.
- Added `get_next_build_number(current_build)` generating sequential `YYYY-MM-DD-NN` daily build strings.
- Updated `update_all_files()` to compute and write `new_build` into `version.json`.
- Added reproduction test suite `quant-pwa/tests/test_reproduce_version_incrementation.py`.

## 4. Verification Evidence
- RED Gate: `test_reproduce_version_incrementation.py` failed with exit code 1.
- GREEN Gate: All 4 test assertions passed in 0.03s with exit code 0.
- Full regression suite: 282 passed in gateway tests, 116 DOM flow tests passed.
- Multi-Repo Fleet CI: 100% green on `develop2` and `master`.
