# Workspace Development Rules

This document defines behavioral and workflow directives that must always be loaded and followed during development across the **Quant System** ecosystem.

---

## 1. Commit & Deployment Rules

### Rule 1.1: Mandatory Pre-Commit Review & Approval (Strict Hard Gate)
- **STRICT PROHIBITION**: Never execute `git commit` or `git push` without first outputting the full `git diff` in chat and waiting for explicit user sign-off.
- **ZERO CARRYOVER APPROVAL**: An approval from an earlier turn (e.g., "push", "go ahead") NEVER carries over to subsequent turns or new batches of changes. Every single new edit, bugfix, or refactor requires its own independent diff review and approval turn.
- **MANDATORY STOP & WAIT**: Whenever modifying code or fixing an issue, you must ALWAYS stop after presenting the diff and wait for explicit confirmation. Never edit and commit in the same turn without explicit permission for that specific change set.

### Rule 1.2: Strict Step-Gated Multi-Repo Push Order
When pushing updates across multiple repositories, you MUST strictly adhere to the following blocking sequence:

* **Gate 1 (`common-lib` First)**:
  1. Push `common-lib` to `develop`.
  2. **BLOCK & WAIT**: You are STRICTLY FORBIDDEN from committing or pushing any other repository until the GitHub Actions CI/CD runner on Synology NAS completes and `common-lib` is confirmed merged into `master`.
* **Gate 2 (Dependent Backend Microservices)**:
  1. Only after `common-lib` is merged into `master`, push dependent backend services (e.g., `gexdex-api`, `quant-level-pipeline`, `mm-dex-gex-pipeline`, `unusual-option-flow-pipeline`, `discord-quant-bot`, `ibkr-historical-data-pipeline`, `etl-pipeline-template`).
  2. Wait for their deployments to succeed.
* **Gate 3 (`quant-pwa` Last)**:
  1. `quant-pwa` MUST ALWAYS be pushed last after all backend services are fully deployed and healthy.

### Rule 1.3: Mandatory Local End-to-End Testing Before Presenting Diff & Committing
- Before presenting any `git diff` or requesting approval to commit/push, you MUST execute a complete local end-to-end (E2E) test verifying:
  1. All automated unit tests (`pytest`) pass 100% locally.
  2. The affected service(s) spin up cleanly without runtime errors or missing dependencies.
  3. The core API endpoints or CLI flows respond with expected payloads and status codes.
- Only after physical local verification passes with zero errors can you present the diff and request review.

Before executing any deployment, you MUST output the explicit **Step-Gating Checklist** in chat and check off each gate only after it is physically verified.

---

## 2. Documentation Standards

### Rule 2.1: Comprehensive Project Documentation & Continuous Maintenance
- Every project repository/folder must have up-to-date documentation (`README.md` or architecture guide) clearly explaining:
  - Purpose, architecture, and internal workflow.
  - User-specified design goals, key parameters, and constraints.
  - Endpoints, pipeline schedules, schemas, or service interfaces.
- **Continuous Revision**: Whenever changes are made to a project folder, the project's documentation must be updated accordingly in the same task.

---

## 3. Communication & Tool Usage Rules

### Rule 3.1: No Status Polling Loops
- Do not execute repetitive polling loops (`status` checks on running background tasks).
- Yield control and rely on reactive task completion notifications from the runtime environment.
