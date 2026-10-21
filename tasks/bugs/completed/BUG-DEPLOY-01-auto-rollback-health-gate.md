---
id: BUG-DEPLOY-01
type: Bug
title: Automated Health-Gate & Auto-Rollback in Synology Pipeline
priority: P0
status: Completed
components: [quant-pwa, nas-infra]
impact: Critical
effort: Med
completed_release: v1.1.19
completed_date: 2026-09-14
---

# [BUG-DEPLOY-01] Automated Health-Gate & Auto-Rollback in Synology Pipeline

## Summary
Resolved production deployment failure modes by introducing Stage 3 rollback image preservation and Stage 4 automated rollback with high-priority NTFY alerts on health or smoke test failures.
