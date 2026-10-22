---
id: BUG-001
type: Bug
title: NAS Staging Container Auto-Lifecycle & Permission Guard
priority: P2
status: Active
components: [quant-pwa, nas-infra]
impact: High
effort: Low
target_release: v1.1.43
assignee: QuantFleetCommander
created: 2026-10-21
---

# [BUG-001] NAS Staging Container Auto-Lifecycle & Permission Guard

## 🎯 Defect Description
When the deployment protocol transitions to Phase 5 (Staging Deploy), attempting to query or spin up staging containers via non-root SSH on Synology DSM fails due to `docker.sock` permission restrictions (`permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock`). Furthermore, when features are promoted to production and staging is auto-torn down, port `:8096` remains dormant and requires manual terminal intervention to restart.

## 🔍 Root Cause Analysis
The deployment runner on the Synology NAS runs under standard user `rachardv` which lacks group write permissions to `/var/run/docker.sock`, or uses an unaliased Docker wrapper script that fails non-interactive batch commands.

## 🛠️ Surgical Remediation Plan
1. Provide a dedicated deployment helper script on NAS (`~/deploy_staging.sh`) callable via sudo or setuid wrapper with strict arguments.
2. In `scripts/protocol_graph.py` Phase 5, add a deterministic health prober that waits up to 60 seconds with exponential backoff for `http://192.168.1.68:8096/health` before declaring staging ready.

## ✅ Acceptance Criteria
- [ ] Reproduction script verifies successful staging spin-up and teardown via SSH.
- [ ] Phase 5 in `protocol_graph.py` completes automatically without manual terminal commands.
- [ ] Staging container enforces memory limit $\le 350\text{MB}$ RAM.
