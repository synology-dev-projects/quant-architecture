---
id: BUG-008
title: Cockpit IBKR GEX/DEX Tab Silent Fallback & SciPy Import Failure
category: bugs
status: completed
priority: P1
components:
  - common-lib
  - quant-pwa
created_date: 2026-10-29
completed_date: 2026-10-29
release: v1.1.45
---

# BUG-008: Cockpit IBKR GEX/DEX Tab Silent Fallback & SciPy Import Failure

## 1. Problem Statement
When switching to the new IBKR data source tab in the Cockpit GEX/DEX component, the UI displayed synthetic/fallback strikes rather than live CBOE options market data. The service silently caught exceptions and fell back to generated strikes without alerting the caller.

## 2. Root Cause Analysis
1. **Un-Guarded Pipeline Import:** `common_lib/calculators/gexdex.py` imported `OptionContractRecord` from `common_lib.extractors.ibkr_gexdex`. This triggered `common_lib/extractors/__init__.py`, which attempted top-level `from ib_async import ...`. The `quant-pwa` gateway container does not include `ib_async`, triggering `ModuleNotFoundError`.
2. **SciPy Dependency Violation:** `common_lib/calculators/gexdex.py` imported `from scipy.stats import norm`. SciPy is not installed in the NAS gateway container environment (<350MB RAM budget), causing a second `ModuleNotFoundError`.
3. **Data Provenance Blind Spot:** `StrikeDistributionResponse` lacked provenance flags (`is_fallback: bool`), masking calculation failures behind standard fallback structures during automated testing.

## 3. Surgical Fix Implemented
1. **Guarded Extractor Imports:**
   - Wrapped `ib_async` imports in `common-lib/common_lib/extractors/ibkr_gexdex.py` and `ibkr_options.py` inside `try...except ImportError`.
   - Guarded `OptionContractRecord` import in `common-lib/common_lib/calculators/gexdex.py` under `if TYPE_CHECKING:` to eliminate runtime module import side effects.
2. **Zero-SciPy Vectorized Normal Distribution:**
   - Replaced `scipy.stats.norm` in `common-lib/common_lib/calculators/gexdex.py` with pure NumPy vectorized Gaussian functions (`_vec_norm_cdf` and `_vec_norm_pdf`) using Python's standard `math.erf`.
3. **Data Provenance Contract:**
   - Added `is_fallback: bool = False` to `StrikeDistributionResponse` in `quant-pwa/gateway/app/engine/service.py`, explicitly setting `is_fallback=True` on fallback branches to prevent silent masking.
   - Updated gateway tests (`test_cockpit_ibkr_tab.py`) to assert `res.is_fallback is False`.

## 4. Verification & Proof
- **RED Gate:** `quant-pwa/gateway/tests/test_reproduce_ibkr_fallback.py` failed with exit code 1 replicating absent `ib_async` and `scipy`.
- **GREEN Gate:** Re-executed exact test with exit code 0 after applying the surgical patch.
- **Suite Verification:** `common-lib` 12 dynamic BSM tests pass; `quant-pwa` 99 frontend tests and 3 gateway tests pass with `is_fallback is False`.
- **Fleet CI:** GitHub Actions runs `40818886819` (`common-lib`) and `40818888997` (`quant-pwa`) passed 100% green on `develop2`.
- **Staging Verification:** NAS Staging port `:8096` verified live with non-fallback CBOE options market data (`NVDA`, `SPY`).
- **Production Gate:** Topologically merged and pushed to `origin/master`.
