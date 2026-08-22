# 🔴 Priority 1 Implementation Plan: IBKR Async Socket Lifecycle & Client ID Leak Prevention

## 1. Problem Statement
In ibkr-historical-data-pipeline, the ingestion script connects to Interactive Brokers Gateway (:4002) using ib_insync.IB().
Under unexpected network terminations, unhandled exceptions, or cron timeouts, the socket connection to TWS/IB Gateway is left dangling.
When the next scheduled cron job runs, IB Gateway rejects the connection with:
Error 326: Unable to connect as the client id is already in use.

---

## 2. Target Solution: Strict Context Managed Disconnect & ID Fallback

`python
class SafeIBConnection:
    def __init__(self, host: str, port: int, base_client_id: int = 1):
        self.host = host
        self.port = port
        self.base_client_id = base_client_id
        self.ib = IB()

    def __enter__(self):
        for offset in range(5):
            try:
                client_id = self.base_client_id + offset
                self.ib.connect(self.host, self.port, clientId=client_id, timeout=10)
                return self.ib
            except Exception:
                continue
        raise ConnectionError("Failed to connect to IB Gateway across client ID pool.")

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.ib.isConnected():
            self.ib.disconnect()
`

---

## 3. Implementation Checklist
1. Update ibkr-historical-data-pipeline/src/collector.py with SafeIBConnection.
2. Add automated unit test with mocked socket disconnection handling.
3. Validate clean exit on SIGTERM/SIGINT.
