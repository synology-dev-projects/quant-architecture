# ⚙️ ETL Data Pipelines & Ingestion Engines

## 1. Overview & Standardized Architecture

All ETL (Extract, Transform, Load) pipelines in the Quant System adhere to a consistent 3-stage module separation:

```mermaid
graph LR
    subgraph ExtractPhase [1. Extract (extract.py)]
        Source[External API / Portal / IBKR] --> Scraper[Session Authenticator & Scraper]
        Scraper --> Raw[Raw JSON / Dict / DF]
    end

    subgraph TransformPhase [2. Transform (transform.py)]
        Raw --> Parser[Regex Matcher & Schema Cleaner]
        Parser --> Validated[Typed Pandas DataFrame]
    end

    subgraph LoadPhase [3. Load (load.py)]
        Validated --> Staging[UUID Temp Staging Table]
        Staging --> Merge[Atomic Oracle MERGE INTO]
        Merge --> Destination[(Production Oracle Table)]
    end
```

---

## 2. Pipeline Deep Dives

### 2.1 `quant-level-pipeline`
* **Source:** TradingEdge Mighty Networks feed (`/api/v2/spaces/.../posts`).
* **Frequency:** Daily incremental cron (after market close at 16:30 ET) & manual historical backfill.
* **Extraction:** Authenticates with session cookies (`TE_COOKIE`), paginates feed, downloads attached `.txt`/`.csv` files, and falls back to HTML body parsing.
* **Parsing Engine:**
  - Extracts market session date from post title (`parser.isoparse`).
  - Matches strike price boundaries using `re.compile(r'^\s*(\d{2,6}(?:\.\d+)?)(?:\s*-\s*(\d{2,6}(?:\.\d+)?))?\s*(.*)')`.
  - Normalizes single price points (e.g. `5800 Bounce`) to `PRICE_LOW=5800.0`, `PRICE_HIGH=5800.0`.
* **Target Table:** `QUANT_LVL_DATA_TE`

---

### 2.2 `ibkr-historical-data-pipeline`
* **Source:** Interactive Brokers Gateway (`ib_insync` socket connection on port 4002).
* **Supported Contracts:** Stocks (STK), Indices (IND), Futures (FUT), Options (OPT).
* **Parameters:** `symbol`, `barSizeSetting` (`1 min`, `5 mins`, `1 hour`, `1 day`), `durationStr` (`1 D`, `1 M`, `1 Y`), `whatToShow` (`TRADES`, `MIDPOINT`, `BID_ASK`).
* **Connection Lifecycle:** Dynamic client ID allocation (`random.randint(100, 9999)`), and strict `try...finally: ib.disconnect()` socket safety.
* **Target Table:** `TICKER_DATA_IBKR`

---

### 2.3 `mm-dex-gex-pipeline`
* **Source:** TradingEdge DEX/GEX raw options chain API.
* **Mathematical Greek Formulations:**
  $$\text{Call GEX}_K = \Gamma_K \times S \times \text{OI}_K \times 100$$
  $$\text{Put GEX}_K = -\Gamma_K \times S \times \text{OI}_K \times 100$$
  $$\text{Net GEX}_K = \text{Call GEX}_K + \text{Put GEX}_K$$
  $$\text{Net DEX}_K = (\Delta_{\text{Call}, K} \times \text{OI}_{\text{Call}, K} - |\Delta_{\text{Put}, K}| \times \text{OI}_{\text{Put}, K}) \times S \times 100$$
  where $S$ is spot price, $K$ is strike price, and $\text{OI}$ is open interest.
* **Target Table:** `MM_DEX_GEX_TE`

---

### 2.4 `unusual-option-flow-pipeline`
* **Source:** Institutional option tape feed.
* **Filtering & Quality Rules:**
  1. $\text{Total Dollar Premium} \ge \$100,000$.
  2. $\text{Volume / Open Interest Ratio} \ge 1.2$ (detecting aggressive new position opening vs closing existing contracts).
  3. Order type classification: Multi-exchange sweeps, block crossings, or split orders.
* **Target Table:** `UNUSUAL_OPTION_FLOW_TE`

---

## 3. Resilience, Schedulers & Alerts

1. **Failure Notification (`ntfy.py`):** Fatal exceptions trigger HTTP POST push notifications with priority 5 to `https://richntfynotifier.synology.me/alerts`.
2. **Weekend & Holiday Invariance:** When scrapers execute on non-trading days and detect 0 new records, they log an informational event and terminate with `exit code 0` to prevent false positive CI/CD alerts.
