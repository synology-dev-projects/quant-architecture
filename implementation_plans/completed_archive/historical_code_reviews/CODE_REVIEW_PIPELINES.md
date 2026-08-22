# 📊 Deep-Dive Code Review: Data & Ingestion Pipelines

**Target Directory:** `C:\Coding\VSCode\Quant System`  
**Pipelines Reviewed:**
1. **`quant-level-pipeline`** (TradingEdge daily quant levels scraper & loader)
2. **`ibkr-historical-data-pipeline`** (Interactive Brokers historical tick & bar extractor)
3. **`mm-dex-gex-pipeline`** (Market maker DEX & GEX options exposure pipeline)
4. **`unusual-option-flow-pipeline`** (Unusual options flow filter and loader)

---

## 1. Pipeline Status & Health Overview

| Pipeline | Completeness | Key Vulnerabilities | Production Status |
| :--- | :---: | :--- | :---: |
| **`quant-level-pipeline`** | **85%** | 1-day incremental cutoff drops posts; regex limit (`\d{4}`); CI tests mutate prod DB | 🟡 **Near-Prod (Needs Fixes)** |
| **`ibkr-historical-data-pipeline`** | **60%** | Hardcoded `clientId=1` collision; socket connection leak; empty bar crashes | 🟠 **Beta / Dev** |
| **`mm-dex-gex-pipeline`** | **20%** | `transform.py` & `load.py` are empty; `extract.py` contains scratch GenAI code | 🔴 **Pre-Alpha / Stub** |
| **`unusual-option-flow-pipeline`** | **10%** | `extract.py`, `transform.py`, `load.py` are 0-byte stubs | 🔴 **Pre-Alpha / Skeleton** |

---

## 2. Findings by Pipeline

---

### 2.1 `quant-level-pipeline`

#### [CRITICAL] PIP-01: Silent Data Loss in Incremental Cutoff Logic
- **File:** `quant-level-pipeline/src/extract.py` (Line 144)
- **Problem:**
  ```python
  if item_date >= cutoff_date + timedelta(days=1):
      keep_list.append(item)
  ```
  In `daily_incremental.py`, `cutoff_date` is the latest datetime in Oracle. Adding `+ timedelta(days=1)` causes `_prune_old_posts` to discard all posts published within 24 hours after `cutoff_date`. New levels published the next day are silently dropped.
- **Fix:**
  ```python
  def _prune_old_posts(posts_list: list, cutoff_date: datetime) -> list:
      if not cutoff_date or not posts_list:
          return posts_list
      keep_list = []
      for item in posts_list:
          raw_date_str = item.get('post', {}).get('created_at')
          if raw_date_str:
              item_date = parser.isoparse(raw_date_str)
              if cutoff_date.tzinfo is None and item_date.tzinfo is not None:
                  cutoff_date = cutoff_date.replace(tzinfo=item_date.tzinfo)
              if item_date > cutoff_date:
                  keep_list.append(item)
          else:
              keep_list.append(item)
      return keep_list
  ```

---

#### [CRITICAL] PIP-02: Integration Tests Mutate Production Database
- **File:** `quant-level-pipeline/tests/test_running_scripts.py` (Lines 19, 39)
- **Problem:**
  `test_historical_load` runs `load.run(env_config, "overwrite", clean_df)` and `test_incremental_load` executes `DELETE FROM {env_config.oracle_quant_table_name} WHERE DATETIME = (SELECT MAX(DATETIME) ...)`. Running `pytest` locally or in CI overwrites live production tables.
- **Fix:**
  Configure test fixtures to write to a dedicated sandbox table (`QUANT_LVL_DATA_TE_TEST`) or mock the database layer with `unittest.mock`.

---

#### [HIGH] PIP-07: Price Regex Rigidly Restricted to 4 Digits
- **File:** `quant-level-pipeline/src/transform.py` (Line 43)
- **Problem:**
  ```python
  line_pattern = re.compile(r'^\s*(\d{4}(?:\.\d+)?)(?:\s*-\s*(\d{4}(?:\.\d+)?))?\s*(.*)')
  ```
  `\d{4}` requires exactly 4 digits. It fails on:
  - 3-digit tickers & ETFs (e.g. SPY at 580, QQQ at 490, IWM at 220)
  - 5-digit indices (e.g. NDX at 21,000, DJIA at 44,000)
- **Fix:**
  Change `\d{4}` to `\d{2,6}`:
  ```python
  line_pattern = re.compile(r'^\s*(\d{2,6}(?:\.\d+)?)(?:\s*-\s*(\d{2,6}(?:\.\d+)?))?\s*(.*)')
  ```

---

#### [HIGH] PIP-08: Failed Attachment Download Erases Post Body
- **File:** `quant-level-pipeline/src/extract.py` (Lines 285–292)
- **Problem:**
  ```python
  if file_tag:
      post["file_link"] = file_tag.get("href")
      post["quant_lvl_text"] = _get_file_content(post["file_link"])
  ```
  If `_get_file_content` fails (HTTP 403, expired link, network timeout), it returns `None`, which overwrites and destroys the `quant_lvl_text` already extracted from the HTML post body.
- **Fix:**
  ```python
  if file_tag:
      file_url = file_tag.get("href")
      post["file_link"] = file_url
      content = _get_file_content(file_url)
      if content:
          post["quant_lvl_text"] = content
  ```

---

### 2.2 `ibkr-historical-data-pipeline`

#### [CRITICAL] PIP-06: Hardcoded `clientId=1` & Unhandled Socket Leaks
- **File:** `common-lib/common_lib/connectors/ibkr.py` (Lines 26, 62, 95)
- **Problem:**
  1. `_connect_to_gateway` hardcodes `clientId=1`. Two concurrent jobs will disconnect each other (`Error 326: Client id already in use`).
  2. `ib.disconnect()` is only called on the happy path. If `reqHistoricalData` fails, the socket connection remains open, locking the port.
- **Fix:**
  ```python
  import random

  def extract_ibkr_ticker_data(config: MainConfig, h_config: HistoryReqConfig) -> pd.DataFrame:
      ib = IB()
      client_id = getattr(config, 'ibkr_client_id', random.randint(100, 9999))
      ib.connect(config.synology_main_ip, config.ibkr_gateway_port, clientId=client_id, timeout=15)
      try:
          contract = _define_contract(h_config)
          ib.qualifyContracts(contract)
          bars = ib.reqHistoricalData(...)
          return util.df(bars) if bars else pd.DataFrame()
      finally:
          if ib.isConnected():
              ib.disconnect()
  ```

---

#### [HIGH] PIP-09: Boolean Parsing Bug in CLI Arguments (`--useRTH`)
- **File:** `ibkr-historical-data-pipeline/src/parse_args.py` (Line 15)
- **Problem:**
  `parser.add_argument("--useRTH", type=bool, ...)` evaluates `bool("False")` as `True`. Passing `--useRTH False` enables RTH filtering.
- **Fix:**
  Use a helper function:
  ```python
  def str_to_bool(v):
      return str(v).lower() in ('true', '1', 't', 'yes', 'y')

  parser.add_argument("--useRTH", type=str_to_bool, default=False)
  ```

---

#### [HIGH] PIP-10: Intraday Date Parsing Drops Last Trading Day
- **File:** `common-lib/common_lib/connectors/ibkr.py` (Line 30)
- **Problem:**
  `endDateStr` parsed to `YYYY-MM-DD 00:00:00` stops data retrieval at midnight before the day opens.
- **Fix:**
  Set intraday end timestamp to market close or end of day (`23:59:59` or `16:00:00 US/Eastern`).

---

### 2.3 `mm-dex-gex-pipeline` & `unusual-option-flow-pipeline`

#### [CRITICAL] PIP-03 & PIP-04: Incomplete Pipeline Stubs
- **Files:**
  - `mm-dex-gex-pipeline/src/transform.py` & `src/load.py` (0 bytes)
  - `mm-dex-gex-pipeline/src/extract.py` (contains interactive scratchpad GenAI code)
  - `unusual-option-flow-pipeline/src/` (all 3 files are 0-byte or 7-line stubs)
- **Problem:**
  Both repositories are incomplete skeletons and cannot run in automated environments.
- **Fix Plan:**
  1. **`mm-dex-gex-pipeline`**:
     - `extract.py`: Call `common_lib.connectors.tradingedge.dexgex.extract_raw_data`.
     - `transform.py`: Clean strikes, expirations, call/put GEX, DEX, and net gamma distributions.
     - `load.py`: Persist into Oracle table `MM_DEX_GEX_TE` via `oracle.insert_into_table(..., mode="upsert")`.
  2. **`unusual-option-flow-pipeline`**:
     - `extract.py`: Call `common_lib.connectors.tradingedge.optionflow.extract_raw_data`.
     - `transform.py`: Calculate Volume/OI ratio, filter for unusual premium (> $100k), tag call/put sentiment.
     - `load.py`: Persist into Oracle table `UNUSUAL_OPTION_FLOW_TE`.

---

#### [CRITICAL] CI-01: Broken CI/CD Deployment Scripts
- **Files:**
  - `mm-dex-gex-pipeline/.github/workflows/deploy.yml`
  - `unusual-option-flow-pipeline/.github/workflows/deploy.yml`
- **Problem:**
  Both workflows contain invalid copy operations (`cp -r common_lib`), reference missing `pyproject.toml`, and write wrong environment variable keys (`ORACLE_HOST_IP` instead of `SYNOLOGY_MAIN_IP`).
- **Fix:**
  Update deployment workflows to match `common_config/CI_CD_DESIGN_STEERING_DOC.md`.
