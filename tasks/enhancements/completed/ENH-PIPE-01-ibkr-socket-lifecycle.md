---
id: ENH-PIPE-01
type: Enhancement
title: IBKR Async Socket Lifecycle & Client ID Leak Prevention
priority: P1
status: Completed
components: [ibkr-historical-data-pipeline]
impact: High
effort: Low
completed_release: v1.1.12
completed_date: 2026-08-30
---

# [ENH-PIPE-01] IBKR Async Socket Lifecycle & Client ID Leak Prevention

## Summary
`SafeIBConnection` context manager with automated client ID pool retries (`base + 0..max_retries`) and unconditional socket disconnects; 100% offline unit tests verified.
