# ENH-005: Blast-Radius Hard Escalation Triggers for Database, Auth & Infra Diffs

## Metadata
- **ID**: `ENH-005`
- **Category**: ⚡ Enhancement
- **Priority**: 🟠 **P2**
- **Status**: Backlog
- **Target Release**: `v1.1.43`
- **Owner**: Principal Systems Architect
- **Components**: `protocol_graph.py`, `captain-orchestrator`, `governance`

---

## 1. Problem Statement & Background
Under a pure file-count or line-count blast radius heuristic, a 5-line diff in database connection pooling (`oracle.py`, `postgres.py`), security middleware (`auth.py`), or container resource limits (`Dockerfile`, `docker-compose.yml`) could be classified as `Tier S` (Small / Solo). This would bypass the `EnterpriseArchitectureReviewer` and SRE checks, risking memory leaks or connection pool starvation on Synology NAS hardware ($<350\text{MB}$ RAM ceiling).

---

## 2. Proposed Solution
1. Introduce deterministic **Hard Escalation Triggers** in `protocol_graph.py` and `captain-orchestrator`:
   - Any diff touching `common_lib/database/`, `gateway/app/core/auth.py`, `docker-compose.yml`, or `Dockerfile` automatically escalates from `Tier S` to **`Tier M` or `Tier L`** regardless of file or line count.
2. Mandatory inclusion of the `<350MB` Synology RAM watchdog check and connection leak probe for any escalated change.

---

## 3. Acceptance Criteria
- [ ] Automated regex / path detection in `protocol_graph.py` auto-escalates sensitive subsystems to Tier M/L.
- [ ] Tier S is strictly forbidden for changes affecting connection pools, credentials, or container limits.
