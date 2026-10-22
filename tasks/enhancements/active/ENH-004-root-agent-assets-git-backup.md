# ENH-004: Version-Control Canonical Root Agent Assets in `quant-architecture`

## Metadata
- **ID**: `ENH-004`
- **Category**: ⚡ Enhancement
- **Priority**: 🟠 **P2**
- **Status**: Backlog
- **Target Release**: `v1.1.43`
- **Owner**: DevSecOps / Architecture Lead
- **Components**: `quant-architecture`, `.agents`, `workspace-governance`

---

## 1. Problem Statement & Background
The workspace root (`C:\Coding\VSCode\Quant System`) is a multi-repo parent container and not a Git repository. Currently, `quant-pwa/.agents/` acts as an unmaintained shadow backup, but Rule 10 mandates that sub-repositories must never host shadow copies of `.agents/`.
If `quant-pwa/.agents/` is removed to enforce Rule 10 SSOT, root `.agents/`, `AGENTS.md`, and `HANDOFF.md` would exist only as unversioned local files on the hard drive with zero git history or GitHub backup.

---

## 2. Proposed Solution
1. Establish `quant-architecture` (the SSOT repository for system architecture and task tracking) as the canonical Git host for root governance assets.
2. Mirror or track `.agents/` and root `AGENTS.md` inside `quant-architecture/` (e.g., `quant-architecture/governance/agents/` or via automated pre-commit sync script).
3. Ensure automated CI checks verify that any modifications to root `.agents/` are committed and pushed to `quant-architecture` on `develop2` and `master`.

---

## 3. Acceptance Criteria
- [ ] Canonical copy of `.agents/` is version-controlled and backed up in GitHub via `quant-architecture`.
- [ ] Zero loss of agent skills or directives if local workspace is cloned on a new machine.
- [ ] Pre-commit or sync script verifies parity between root `.agents/` and `quant-architecture`.
