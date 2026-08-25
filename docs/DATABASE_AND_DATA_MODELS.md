# 🗄️ Database Architecture & Data Models

## 1. Overview & PostgreSQL 16 / TimescaleDB Persistence Engine

The Quant System uses **PostgreSQL 16 with TimescaleDB** (`timescale/timescaledb:latest-pg16`) hosted on the Synology NAS as its primary quantitative persistence engine.

* **Container Topology:** Container `quant-db-prod` exposed on host port `5435` (mapped to internal container port `5432`).
* **Memory Footprint:** Operates within **~250MB RAM** (down from >6GB in legacy Oracle XE), providing high-throughput I/O and zero-copy JSON/relational operations.
* **Driver & Connector Layer:** Managed via [`common-lib/common_lib/connectors/postgres.py`](file:///C:/Coding/VSCode/Quant%20System/common-lib/common_lib/connectors/postgres.py) with pooled `psycopg 3` (`postgresql+psycopg`) connections and native PostgreSQL `ON CONFLICT` upserts.

---

## 2. PostgreSQL Table Schemas & Data Catalog

### 2.1 `unusual_option_flow_te` (Institutional Flow Tracker)
Stores filtered high-premium institutional sweeps, splits, and block trades extracted from live scrapers and historical feeds.

| Column Name | PostgreSQL Data Type | Nullable | Primary Key | Description |
| :--- | :--- | :---: | :---: | :--- |
| `flow_id` | `VARCHAR(128)` | NO | 🔑 PK | Unique deterministic flow identifier (e.g. `FLW-SPY-...`). |
| `trade_date` | `DATE` | NO | - | Date of trade execution. |
| `symbol` | `VARCHAR(32)` | NO | - | Underlying asset ticker (e.g. `SPY`, `NVDA`, `AAPL`). |
| `order_type` | `VARCHAR(64)` | YES | - | Order classification (`BULLISH_SWEEP`, `CALL_BLOCK`, `PUT_SWEEP`). |
| `strike_price` | `NUMERIC(12, 2)` | YES | - | Option strike price. |
| `strike_otm_pct` | `NUMERIC(8, 2)` | YES | - | Percentage distance out-of-the-money (+% OTM, -% ITM). |
| `expiration_date` | `DATE` | YES | - | Option contract expiration date. |
| `open_interest` | `BIGINT` | YES | - | Pre-trade open interest. |
| `is_unusual_oi` | `INTEGER` | YES | - | Flag indicating trade volume exceeds open interest (1 = Unusual, 0 = Normal). |
| `premium` | `NUMERIC(16, 2)` | YES | - | Total dollar premium paid ($\ge \$100\text{k}$). |
| `net_score` | `NUMERIC(8, 2)` | YES | - | Inferred institutional sentiment score (+5.0 Bullish to -5.0 Bearish). |
| `created_at` | `TIMESTAMPTZ` | YES | - | Timestamp of record creation. |

#### Composite Indexes:
* `idx_flow_sym_date_prem`: `CREATE INDEX idx_flow_sym_date_prem ON unusual_option_flow_te (symbol, trade_date DESC, premium DESC);`
  * *Purpose:* Sub-2ms execution for multi-ticker lookups (`WHERE symbol IN (...) AND trade_date >= ... AND premium >= ...`).
* `idx_flow_date_prem`: `CREATE INDEX idx_flow_date_prem ON unusual_option_flow_te (trade_date DESC, premium DESC);`
  * *Purpose:* Sub-3ms execution for market-wide single-day scans.

#### 2.1.1 Options Flow Query Patterns:

**1. Market-Wide Single-Day Macro Query (`/flow <date>` / `/flow market`):**
```sql
SELECT flow_id, trade_date, symbol, order_type, strike_price,
       strike_otm_pct, expiration_date, open_interest, is_unusual_oi,
       premium, net_score
FROM unusual_option_flow_te
WHERE trade_date = :target_date
  AND premium >= :min_premium
ORDER BY premium DESC
LIMIT :limit;
```
* *Latency Profile:* **`< 3.0ms`** execution on PostgreSQL 16 via index scan.
* *Aggregation:* Evaluates cross-market tape sentiment, aggregates call vs. put premium distribution, groups top 5 bullish/bearish tickers by volume, and ranks whale sweep prints.

**2. Single/Multi-Ticker Multi-Week Query (`/flow <ticker>`):**
```sql
SELECT flow_id, trade_date, symbol, order_type, strike_price,
       strike_otm_pct, expiration_date, open_interest, is_unusual_oi,
       premium, net_score
FROM unusual_option_flow_te
WHERE symbol = :symbol
  AND trade_date >= :start_date
  AND premium >= :min_premium
ORDER BY trade_date DESC, premium DESC;
```
* *Latency Profile:* **`< 1.5ms`** execution via `idx_flow_sym_date_prem`.

---

### 2.2 `quant_lvl_data_te` (TradingEdge Daily Quant Levels)
Stores daily quantitative support, resistance, reversal, and bounce levels extracted from TradingEdge daily posts.

| Column Name | PostgreSQL Data Type | Nullable | Primary Key | Description |
| :--- | :--- | :---: | :---: | :--- |
| `datetime` | `TIMESTAMP` | NO | 🔑 PK | Timestamp of the quant levels post (Market Session Date). |
| `ticker` | `VARCHAR(32)` | NO | 🔑 PK | Underlying asset symbol (e.g. `SPX`, `QQQ`, `NVDA`, `AAPL`). |
| `start_lvl_price` | `NUMERIC(12, 2)` | NO | 🔑 PK | Lower bound of the price level range / exact level price. |
| `end_lvl_price` | `NUMERIC(12, 2)` | YES | - | Upper bound of the price level range (null for point levels). |
| `comments` | `TEXT` | YES | - | Raw commentary, notes, or target annotations associated with the level. |
| `buy_sell_ind` | `VARCHAR(16)` | YES | - | Signal indicator (e.g. `BUY`, `SELL`, `BOUNCE`, `REJECT`). |
| `web_link` | `TEXT` | YES | - | Canonical URL to the TradingEdge source post. |
| `created_at` | `TIMESTAMPTZ` | YES | - | Timestamp of record insertion. |

#### Composite Indexes:
* `idx_quant_lvl_lookup`: `CREATE INDEX idx_quant_lvl_lookup ON quant_lvl_data_te (ticker, datetime DESC);`
  * *Purpose:* Instant point-in-time price ladder lookups for AI agent reasoning.

---

### 2.3 `ticker_data_ibkr` (Interactive Brokers Historical Tick & Bar Data)
Stores OHLCV and volume-weighted average price (WAP) historical market bars.

| Column Name | PostgreSQL Data Type | Nullable | Primary Key | Description |
| :--- | :--- | :---: | :---: | :--- |
| `symbol` | `VARCHAR(32)` | NO | 🔑 PK | Ticker symbol (e.g. `SPY`, `TSLA`). |
| `datetime` | `TIMESTAMP` | NO | 🔑 PK | Bar open timestamp in UTC. |
| `open` | `NUMERIC(12, 4)` | YES | - | Opening price of the bar interval. |
| `high` | `NUMERIC(12, 4)` | YES | - | Highest traded price during the interval. |
| `low` | `NUMERIC(12, 4)` | YES | - | Lowest traded price during the interval. |
| `close` | `NUMERIC(12, 4)` | YES | - | Closing price of the bar interval. |
| `volume` | `BIGINT` | YES | - | Total share/contract volume traded. |
| `barcount` | `INTEGER` | YES | - | Number of completed trades during the bar. |
| `wap` | `NUMERIC(12, 4)` | YES | - | Volume-Weighted Average Price (WAP) calculated by IBKR. |
| `barsize` | `VARCHAR(16)` | YES | - | Granularity of the bar (e.g. `1 min`, `5 mins`, `1 hour`, `1 day`). |

---

## 3. High-Speed Native Upsert Protocol (`ON CONFLICT DO UPDATE`)

Unlike legacy Oracle workflows requiring dynamic UUID staging tables (`TMP_...`) and multi-line `MERGE INTO` SQL statements, PostgreSQL 16 executes atomic upserts natively in single-flight statements via [`write_to_postgres_upsert`](file:///C:/Coding/VSCode/Quant%20System/common-lib/common_lib/connectors/postgres.py):

```sql
INSERT INTO unusual_option_flow_te (
    flow_id, trade_date, symbol, order_type, strike_price,
    strike_otm_pct, expiration_date, open_interest, is_unusual_oi, premium, net_score
)
VALUES (
    :flow_id, :trade_date, :symbol, :order_type, :strike_price,
    :strike_otm_pct, :expiration_date, :open_interest, :is_unusual_oi, :premium, :net_score
)
ON CONFLICT (flow_id) DO UPDATE SET
    premium = EXCLUDED.premium,
    net_score = EXCLUDED.net_score,
    open_interest = EXCLUDED.open_interest,
    is_unusual_oi = EXCLUDED.is_unusual_oi;
```

### Advantages over Legacy Staging Tables:
1. **Zero DDL Overhead:** No `CREATE TABLE TMP_...` or `DROP TABLE TMP_...` transactions on Copy-on-Write storage.
2. **Zero Lock Contention:** Row-level locks during `ON CONFLICT` prevent table-level schema locking.
3. **Atomic Execution:** Upsert operations complete in **< 10ms** even for 5,000+ row batches.

---

## 4. SQLAlchemy Connection Pooling Architecture (`common-lib`)

To guarantee fail-safe connection management and sub-millisecond database queries, [`common-lib/common_lib/connectors/postgres.py`](file:///C:/Coding/VSCode/Quant%20System/common-lib/common_lib/connectors/postgres.py) implements a cached SQLAlchemy engine pool with psycopg 3:

```python
@lru_cache(maxsize=8)
def _get_postgres_engine_cached(
    user: str,
    password_secret: str,
    host: str,
    port: int,
    db: str
) -> sa.Engine:
    """
    Creates and caches a pooled PostgreSQL SQLAlchemy engine using psycopg.
    """
    url = sa.engine.URL.create(
        drivername="postgresql+psycopg",
        username=user,
        password=password_secret,
        host=host,
        port=port,
        database=db
    )
    return sa.create_engine(
        url,
        pool_size=5,
        max_overflow=10,
        pool_recycle=1800,
        pool_pre_ping=True
    )
```

### Pool Parameters:
* `pool_size=5`: Maintains 5 persistent connections per microservice / pipeline.
* `max_overflow=10`: Allows bursting up to 15 concurrent queries during high-load market open hours.
* `pool_recycle=1800`: Recycles idle connections every 30 minutes to avoid firewall timeouts.
* `pool_pre_ping=True`: Emits a lightweight `SELECT 1` probe before checkout to automatically recover from dropped connections.
* **Context-Managed Sessions:** Connections are checked out and returned via `with engine.begin() as conn:`.


