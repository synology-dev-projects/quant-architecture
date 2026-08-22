# 🗄️ Database Architecture & Data Models

## 1. Overview & Oracle Persistence Engine

All structured quantitative data is stored in a centralized **Oracle Database (Oracle 19c/21c XE)** hosted on the Synology NAS. 

The shared database connector in [`common-lib/common_lib/connectors/oracle.py`](file:///C:/Coding/VSCode/Quant%20System/common-lib/common_lib/connectors/oracle.py) manages all table creation, schema inspection, dynamic upsert statements, and connection pooling.

---

## 2. Table Schemas & Data Catalog

### 2.1 `QUANT_LVL_DATA_TE` (TradingEdge Daily Quant Levels)
Stores daily quantitative support, resistance, reversal, and breakout levels extracted from TradingEdge daily posts.

| Column Name | Oracle Data Type | Nullable | Primary Key | Description |
| :--- | :--- | :---: | :---: | :--- |
| `DATETIME` | `DATE` / `TIMESTAMP` | NO | 🔑 PK | Timestamp of the quant levels post (Market Session Date). |
| `TICKER` | `VARCHAR2(20)` | NO | 🔑 PK | Underlying asset symbol (e.g. `SPX`, `QQQ`, `NVDA`, `AAPL`). |
| `PRICE_LOW` | `NUMBER(18, 4)` | NO | 🔑 PK | Lower bound of the price level range. |
| `PRICE_HIGH` | `NUMBER(18, 4)` | NO | 🔑 PK | Upper bound of the price level range (equal to `PRICE_LOW` for point levels). |
| `BUY_SELL_IND` | `VARCHAR2(50)` | YES | - | Signal indicator (e.g. `Bounce / Support`, `Rejection / Resistance`, `Breakout`). |
| `COMMENTS` | `VARCHAR2(4000)`| YES | - | Raw commentary, notes, or target annotations associated with the level. |
| `WEB_LINK` | `VARCHAR2(1000)`| YES | - | Canonical URL to the TradingEdge source post. |

---

### 2.2 `TICKER_DATA_IBKR` (Interactive Brokers Historical Tick & Bar Data)
Stores OHLCV and volume-weighted average price (WAP) historical market bars.

| Column Name | Oracle Data Type | Nullable | Primary Key | Description |
| :--- | :--- | :---: | :---: | :--- |
| `DATETIME` | `TIMESTAMP` | NO | 🔑 PK | Bar open timestamp in UTC. |
| `SYMBOL` | `VARCHAR2(20)` | NO | 🔑 PK | Ticker symbol (e.g. `SPY`, `TSLA`). |
| `BARSIZE` | `VARCHAR2(20)` | NO | 🔑 PK | Granularity of the bar (e.g. `1 min`, `5 mins`, `1 hour`, `1 day`). |
| `OPEN` | `NUMBER(18, 4)` | NO | - | Opening price of the bar interval. |
| `HIGH` | `NUMBER(18, 4)` | NO | - | Highest traded price during the interval. |
| `LOW` | `NUMBER(18, 4)` | NO | - | Lowest traded price during the interval. |
| `CLOSE` | `NUMBER(18, 4)` | NO | - | Closing price of the bar interval. |
| `VOLUME` | `NUMBER(18, 0)` | NO | - | Total share/contract volume traded. |
| `WAP` | `NUMBER(18, 4)` | YES | - | Volume-Weighted Average Price (WAP) calculated by IBKR. |
| `BARCOUNT` | `NUMBER(10, 0)` | YES | - | Number of completed trades during the bar. |

---

### 2.3 `MM_DEX_GEX_TE` (Market Maker Delta & Gamma Exposure)
Stores strike-by-strike dealer Greek exposure matrices.

| Column Name | Oracle Data Type | Nullable | Primary Key | Description |
| :--- | :--- | :---: | :---: | :--- |
| `SNAPSHOT_TIME` | `TIMESTAMP` | NO | 🔑 PK | Timestamp when the options chain snapshot was captured. |
| `TICKER` | `VARCHAR2(20)` | NO | 🔑 PK | Underlying equity/index ticker. |
| `EXPIRATION` | `DATE` | NO | 🔑 PK | Option expiration date. |
| `STRIKE` | `NUMBER(18, 4)` | NO | 🔑 PK | Option strike price. |
| `CALL_GEX` | `NUMBER(18, 4)` | NO | - | Dealer Gamma Exposure contributed by Call open interest ($/pt). |
| `PUT_GEX` | `NUMBER(18, 4)` | NO | - | Dealer Gamma Exposure contributed by Put open interest ($/pt). |
| `NET_GEX` | `NUMBER(18, 4)` | NO | - | Net Dealer Gamma (`CALL_GEX - PUT_GEX`). |
| `CALL_DEX` | `NUMBER(18, 4)` | NO | - | Dealer Delta Exposure contributed by Calls ($). |
| `PUT_DEX` | `NUMBER(18, 4)` | NO | - | Dealer Delta Exposure contributed by Puts ($). |
| `NET_DEX` | `NUMBER(18, 4)` | NO | - | Net Dealer Delta (`CALL_DEX - PUT_DEX`). |
| `SPOT_PRICE` | `NUMBER(18, 4)` | NO | - | Underlying asset spot price at time of snapshot. |

---

### 2.4 `UNUSUAL_OPTION_FLOW_TE` (Institutional Flow Tracker)
Stores filtered high-premium institutional sweeps, splits, and block trades.

| Column Name | Oracle Data Type | Nullable | Primary Key | Description |
| :--- | :--- | :---: | :---: | :--- |
| `EXEC_TIME` | `TIMESTAMP` | NO | 🔑 PK | Execution timestamp of the trade. |
| `TICKER` | `VARCHAR2(20)` | NO | 🔑 PK | Underlying symbol. |
| `EXPIRATION` | `DATE` | NO | 🔑 PK | Option contract expiration. |
| `STRIKE` | `NUMBER(18, 4)` | NO | 🔑 PK | Strike price. |
| `CALL_PUT` | `VARCHAR2(4)` | NO | 🔑 PK | Option type (`CALL` or `PUT`). |
| `PREMIUM` | `NUMBER(18, 2)` | NO | - | Total dollar premium paid ($\ge \$100\text{k}$). |
| `VOLUME` | `NUMBER(12, 0)` | NO | - | Number of contracts traded in the order. |
| `OPEN_INT` | `NUMBER(12, 0)` | NO | - | Open interest prior to trade execution. |
| `VOL_OI_RATIO` | `NUMBER(8, 2)` | NO | - | Volume-to-Open-Interest ratio ($\text{Volume} / \text{OI}$). |
| `ORDER_TYPE` | `VARCHAR2(20)` | YES | - | Order routing (`SWEEP`, `BLOCK`, `SPLIT`). |
| `SENTIMENT` | `VARCHAR2(20)` | YES | - | Inferred market maker sentiment (`BULLISH`, `BEARISH`, `NEUTRAL`). |

---

## 3. Idempotent Atomic MERGE Protocol

To ensure concurrent pipelines never create duplicate records or drop tables in race conditions:

```sql
MERGE INTO QUANT_LVL_DATA_TE target
USING TMP_QUANT_LVL_DATA_TE_A7B39C12 source
ON (
    target.DATETIME = source.DATETIME AND
    target.TICKER = source.TICKER AND
    target.PRICE_LOW = source.PRICE_LOW AND
    target.PRICE_HIGH = source.PRICE_HIGH
)
WHEN MATCHED THEN
    UPDATE SET
        target.BUY_SELL_IND = source.BUY_SELL_IND,
        target.COMMENTS = source.COMMENTS,
        target.WEB_LINK = source.WEB_LINK
WHEN NOT MATCHED THEN
    INSERT (DATETIME, TICKER, PRICE_LOW, PRICE_HIGH, BUY_SELL_IND, COMMENTS, WEB_LINK)
    VALUES (source.DATETIME, source.TICKER, source.PRICE_LOW, source.PRICE_HIGH, source.BUY_SELL_IND, source.COMMENTS, source.WEB_LINK);
```

### Key Concurrency Rules:
1. **Dynamic Hex UUID Suffixes:** Every staging table is created with a unique UUID suffix: `f"TMP_{table_name[:12]}_{uuid.uuid4().hex[:8]}".upper()`.
2. **Auto-Cleanup in `finally` Block:** Staging tables are dropped immediately following the `MERGE` execution.

---

## 4. SQLAlchemy Connection Pooling Architecture (`common-lib`)

To prevent database connection starvation and `ORA-12516` (TNS listener could not find available handler with matching protocol stack), `common-lib/common_lib/connectors/oracle.py` uses a cached engine pool singleton:

```python
@lru_cache(maxsize=8)
def _get_engine_cached(oracle_user: str, oracle_pass_secret: str, host: str, service: str, port: int = 1521) -> sa.Engine:
    """
    Creates and caches a pooled SQLAlchemy engine.
    """
    dsn = f"oracle+oracledb://{oracle_user}:{oracle_pass_secret}@{host}:{port}/?service_name={service}"
    return sa.create_engine(
        dsn,
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
* `pool_pre_ping=True`: Emits a lightweight `SELECT 1 FROM DUAL` probe before checkout to automatically recover from dropped connections.
* **Zero Per-Query `engine.dispose()`:** Connections are returned to the pool via context managers (`with engine.begin() as conn:`) rather than terminating the underlying engine pool.

