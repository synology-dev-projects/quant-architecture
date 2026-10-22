---
id: ENH-003
type: Enhancement
title: Pipeline Telemetry & Prefect Flow Health Modal
priority: P4
status: Active
components: [quant-pwa]
impact: Low
effort: Low
target_release: v1.2.0
assignee: QuantFleetCommander
created: 2026-10-21
---

# [ENH-003] Pipeline Telemetry & Prefect Flow Health Modal

## 🎯 Objective
Expose real-time pipeline execution health, last-run timestamps, total rows processed, and manual run triggers in the Quant PWA Settings Modal, powered by `GET /api/pipelines/status` and `POST /api/pipelines/{name}/trigger`.

## 📋 User Story
As an administrator, I want to check pipeline freshness (flow, quant levels, gexdex, market confluence) and manually trigger on-demand pipeline runs directly from the UI without needing SSH terminal access.

## 🏗️ Technical Architecture & Design
1. **Frontend Modal UI (`frontend/src/components/pipeline_health_modal.js`)**:
   - Table displaying: Pipeline Name, Status Badge (Success / In Progress / Failed / Not Started), Last Run Date, Rows Ingested, Trigger Button.
2. **Gateway Hook**:
   - Direct integration with `pipelines_router.py`.
   - Polling interval (10s) while modal is open; destroyed on close.

## ✅ Acceptance Criteria
- [ ] Passes layout audit (`audit_layout.js`) with $\ge 44\text{px}$ touch targets.
- [ ] Manual trigger button incorporates confirmation prompt and displays loading spinner.
- [ ] Authentication guard: Requires admin session passcode.
