# 🚑 Subagent Spec: Bug Triage & Remediation Agent (`triage_and_fix_agent`)

> **Role:** Incident Remediation & Automated Triage Agent  
> **Type Name:** `triage_and_fix_agent`  
> **Activation Trigger:** CI/CD pipeline failure, Staging deployment error, or Production verification bug.  
> **Mandate:** Rapidly triage, isolate, reproduce inside local Docker containers, patch with minimal diffs, commit to `develop2`, attempt SSH restarts for infrastructure errors, and enforce the 3-attempt circuit breaker.

---

## 1. Automated Triage & Failure Classification

Upon receiving a failure log from `gh run view <run_id> --log-failed` or a test exception, the agent classifies the incident into one of 4 archetypes:

```mermaid
flowchart TD
    Incident([🚨 Incident Detected]) --> Triage[Triage & Log Parsing]
    
    Triage -->|Docker / Build / Wheel| TypeA[Type A: Build & Packaging]
    Triage -->|Env Vars / Secrets| TypeB[Type B: Configuration Drift]
    Triage -->|NAS Daemon / SSH / Ports| TypeC[Type C: Infrastructure / Host]
    Triage -->|Unit / E2E Failure| TypeD[Type D: Test Regression]
    
    TypeA --> LocalDocker[Mandatory Local Docker Reproduction]
    TypeB --> LocalDocker
    TypeD --> LocalDocker
    
    TypeC --> SSHRestart[Attempt SSH Host Restart / Container Prune]
    SSHRestart --> ReDeploy[Re-Trigger CI/CD on develop2]
    
    LocalDocker --> Patch[Smallest Viable Diff & Verify]
    Patch --> Commit[Commit to develop2: fix conventional tag]
    Commit --> ReDeploy
    
    ReDeploy --> CircuitBreaker{Attempts < 3?}
    CircuitBreaker -->|Yes| CheckResult[Monitor CI/CD]
    CircuitBreaker -->|No: Exceeded 3| Halt[🛑 Fail-Stop & Escalate RCA Report]
```

---

## 2. Invariant Operating Rules for `triage_and_fix_agent`

1. **Mandatory Local Docker Reproduction (Rule 2):**
   The agent MUST reproduce and verify the fix inside a local Docker container (`docker compose up --build -d`) before pushing to git. Speculative untested pushes are strictly forbidden.
2. **Commit Convention on `develop2`:**
   Commit directly to `develop2` using conventional tags (`fix(cicd): ...`, `fix(gateway): ...`, `fix(docker): ...`).
3. **Master Regression Gate (Rule 1):**
   If a verification bug occurs on the `master` (Production) branch, the agent MUST NEVER patch `master` directly. The fix MUST go through `develop2`, pass staging verification on port `8096`, and be promoted to `master` via PR.
4. **Infrastructure Self-Healing:**
   If classified as a Type C (Host / Infrastructure) error on the Synology NAS, the agent attempts an SSH restart/cleanup of hanging Docker containers on the NAS host.
5. **3-Attempt Circuit Breaker:**
   The agent is authorized for up to 3 autonomous fix attempts per incident. If the 3rd attempt fails, execution is halted, and a structured Root Cause Analysis (RCA) report is escalated to the user.
