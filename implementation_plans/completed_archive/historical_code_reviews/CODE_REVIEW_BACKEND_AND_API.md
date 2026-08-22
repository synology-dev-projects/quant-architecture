# 🔍 Deep-Dive Code Review: Backend Core & APIs

**Target Directory:** `C:\Coding\VSCode\Quant System`  
**Repositories Reviewed:**
1. **`common-lib`** (Shared connectors, config, utility, and deployment scripts)
2. **`common_config`** (Database metadata catalog, environment definitions)
3. **`etl-pipeline-template`** (Scaffolding for pipelines)
4. **`gexdex-api`** (FastAPI backend for GEX/DEX options analytics)

---

## 1. Security & Secrets Management

### SEC-01: Plaintext SSH & Sudo Password in Deployment Script
- **File:** `common-lib/scripts/deploy_all.py` (Line 24)
- **Severity:** 🔴 **Critical**
- **Problem:**
  ```python
  cmd = 'plink.exe -ssh -pw "4354GoGo!!" -batch rachardv@192.168.1.68 "echo \'4354GoGo!!\' | sudo -S /usr/local/bin/docker logs --tail 5 synology-github-runner"'
  ```
  The Synology NAS user password and sudo password are hardcoded in plain text in source control.
- **Remediation:**
  1. Rotate the Synology user password immediately.
  2. Set up SSH public key authentication (`authorized_keys`).
  3. Grant passwordless sudo to docker runner commands via `/etc/sudoers.d/docker-runner`.

---

### SEC-02: Plaintext Production Secrets in `common_config/.env`
- **File:** `common_config/.env` (Lines 1–12)
- **Severity:** 🔴 **Critical**
- **Problem:**
  The `.env` file containing live Oracle passwords (`ORACLE_PASS`), third-party portal passwords (`TE_PASS`), and active authenticated session cookies (`TE_COOKIE`) is tracked in Git without a `.gitignore`.
- **Remediation:**
  1. Add a `.gitignore` inside `common_config` containing `.env` and `*.env`.
  2. Provide a sanitized `common_config/.env.example`.
  3. Rotate all database passwords and third-party portal credentials.

---

### SEC-03: Fallback API Key in `gexdex-api`
- **File:** `gexdex-api/app/config.py` (Line 21), `docker-compose.yml` (Line 10)
- **Severity:** 🟠 **High**
- **Problem:**
  ```python
  expected_api_key = os.getenv("API_KEY", "YOUR_SECRET_API_KEY_HERE")
  ```
  If `API_KEY` is not set in the environment, the API falls back to a publicly known dummy key, allowing anyone to bypass authentication.
- **Remediation:**
  Fail fast and fail closed on startup:
  ```python
  expected_api_key = os.getenv("API_KEY")
  if not expected_api_key:
      raise RuntimeError("CRITICAL: API_KEY environment variable is required.")
  ```

---

### SEC-04: Insecure CORS Configuration in `gexdex-api`
- **File:** `gexdex-api/app/main.py` (Lines 12–18)
- **Severity:** 🟠 **High**
- **Problem:**
  ```python
  app.add_middleware(
      CORSMiddleware,
      allow_origins=["*"],
      allow_credentials=True,
      allow_methods=["*"],
      allow_headers=["*"],
  )
  ```
  Combining `allow_origins=["*"]` with `allow_credentials=True` violates CORS specifications and allows third-party browser contexts to make authenticated requests.
- **Remediation:**
  ```python
  trusted_origins = os.getenv("ALLOWED_ORIGINS", "http://localhost:3000,http://192.168.1.68").split(",")
  app.add_middleware(
      CORSMiddleware,
      allow_origins=trusted_origins,
      allow_credentials=True,
      allow_methods=["GET", "POST", "OPTIONS"],
      allow_headers=["*"],
  )
  ```

---

### SEC-05: Constant-Time API Key Comparison
- **File:** `gexdex-api/app/config.py` (Line 22)
- **Severity:** 🟠 **High**
- **Problem:**
  Using `api_key != expected_api_key` is vulnerable to timing side-channel attacks.
- **Remediation:**
  ```python
  import secrets
  if not api_key or not secrets.compare_digest(api_key, expected_api_key):
      raise HTTPException(status_code=401, detail="Invalid API Key")
  ```

---

## 2. Concurrency, Database & Engine Lifecycle

### COR-01: Static Temporary Table Race Condition in Oracle Upsert
- **File:** `common-lib/common_lib/connectors/oracle.py` (Lines 248–295)
- **Severity:** 🔴 **Critical**
- **Problem:**
  ```python
  temp_table_name = "TEMP_" + table_name[:20]
  _df_to_oracle_overwrite(engine, df, temp_table_name, primary_keys)
  merge_sql = _create_merge_statement(engine, temp_table_name, table_name, "upsert")
  conn.execute(sa.text(merge_sql))
  # in finally:
  _drop_table_internal(engine, temp_table_name)
  ```
  If two pipelines (e.g. historical load and intraday scrape) execute concurrently on the same table, both attempt to create and drop `TEMP_<table_name>` simultaneously, causing `ORA-00054` (resource busy), `ORA-00942` (table does not exist), or mixing data between sessions.
- **Remediation:**
  Use unique table names with random UUID suffixes:
  ```python
  import uuid
  temp_table_name = f"TMP_{table_name[:12]}_{uuid.uuid4().hex[:8]}".upper()
  ```

---

### PERF-01: Engine Destruction & Missing Connection Pooling
- **File:** `common-lib/common_lib/connectors/oracle.py` (Lines 14–19, 39, 59, 77, 109)
- **Severity:** 🟠 **High**
- **Problem:**
  Every SQL helper calls `_get_engine(config)` and immediately runs `engine.dispose()` in a `finally` block. This destroys the SQLAlchemy connection pool on every query, forcing a new TCP handshake and Oracle session handshake (~200–500ms overhead per call).
- **Remediation:**
  Implement a module-level cached engine pool:
  ```python
  from functools import lru_cache
  import sqlalchemy as sa

  @lru_cache(maxsize=8)
  def get_oracle_engine(user: str, host: str, service: str, port: int = 1521) -> sa.Engine:
      dsn = f"oracle+oracledb://{user}:{pass}@{host}:{port}/?service_name={service}"
      return sa.create_engine(
          dsn,
          pool_size=5,
          max_overflow=10,
          pool_recycle=1800,
          pool_pre_ping=True
      )
  ```

---

## 3. Quantitative Math & Correctness

### COR-02: DTE Calculation Operator Precedence Bug
- **File:** `gexdex-api/app/services/gexdex_service.py` (Lines 171–177)
- **Severity:** 🟠 **High**
- **Problem:**
  ```python
  try:
      exp_date = pd.to_datetime(row.get("expiration"))
      dte = (exp_date.tz_localize('UTC') if exp_date.tz is None else exp_date - now_dt).days
      if dte <= 7:
          front_week_gex += abs_gex
  except Exception:
      pass
  ```
  When `exp_date.tz is None`, `(exp_date.tz_localize('UTC')).days` evaluates and raises `AttributeError: 'Timestamp' object has no attribute 'days'`. The `except Exception: pass` block silently swallows this error for every row, resulting in `front_week_gex = 0`, corrupting `pin_risk_level` and `front_week_gex_pct`.
- **Remediation:**
  ```python
  exp_date = pd.to_datetime(row.get("expiration"))
  if exp_date.tzinfo is None:
      exp_date = exp_date.tz_localize(timezone.utc)
  dte = (exp_date - now_dt).days
  if 0 <= dte <= 7:
      front_week_gex += abs_gex
  ```

---

### COR-03: Cookie Jar Extraction Bug in Option Flow
- **File:** `common-lib/common_lib/connectors/tradingedge/optionflow.py` (Line 55)
- **Severity:** 🟠 **High**
- **Problem:**
  `cookie_val = post_response.request.headers.get('Cookie')` retrieves the outbound request header instead of the cookies received from the ASP.NET authentication response.
- **Remediation:**
  ```python
  cookie_val = "; ".join([f"{c.name}={c.value}" for c in session.cookies])
  ```

---

## 4. Performance & Memory Management

### PERF-02: Unbounded In-Memory Cache in `gexdex-api`
- **File:** `gexdex-api/app/services/gexdex_service.py` (Lines 302–305)
- **Severity:** 🟡 **Medium**
- **Problem:**
  `_RAW_CACHE` and `_CHART_CACHE` dictionaries grow indefinitely without maximum size bounds or TTL eviction.
- **Remediation:**
  Use `cachetools.TTLCache`:
  ```python
  from cachetools import TTLCache
  import threading

  _RAW_CACHE = TTLCache(maxsize=100, ttl=300)      # 5 min TTL
  _CHART_CACHE = TTLCache(maxsize=200, ttl=3600)   # 1 hour TTL
  _CACHE_LOCK = threading.Lock()
  ```

---

## 5. Testing & CI/CD Isolation

### TST-01: Unit Tests Hitting Live External Services & Real Alerts
- **Files:** `common-lib/tests/conftest.py` (Lines 21–43), `common-lib/tests/test_ntfy.py` (Line 28)
- **Severity:** 🟠 **High**
- **Problem:**
  - `conftest.py` authenticates against live TradingEdge portals on test discovery.
  - `test_ntfy.py` sends live priority 5 push notifications during local test runs.
- **Remediation:**
  Mock external HTTP requests using `unittest.mock` or `responses`. Move live connectivity checks to separate `@pytest.mark.integration` test suites.

---

### ARCH-03: Broken CI/CD Workflow in `etl-pipeline-template`
- **File:** `etl-pipeline-template/.github/workflows/deploy.yml` (Lines 64–67, 116)
- **Severity:** 🟡 **Medium**
- **Problem:**
  `deploy.yml` contains `cp -r common_lib` (which does not exist in pipeline repos) and injects `ORACLE_HOST_IP` instead of `SYNOLOGY_MAIN_IP`.
- **Remediation:**
  Align template with `common_config/CI_CD_DESIGN_STEERING_DOC.md`.
