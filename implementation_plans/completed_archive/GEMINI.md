# User Rules & Behavioral Directives

1. **Mandatory User Review Before Git Commits (Strict Hard Gate)**:
   - **STRICT PROHIBITION**: You are STRICTLY FORBIDDEN from executing any `git commit` or `git push` commands without first presenting the complete `git diff` in chat and waiting for explicit user approval.
   - **ZERO CARRYOVER APPROVAL**: An approval from a prior turn (e.g. "push", "go ahead") NEVER applies to subsequent changes. Every new code modification or bugfix requires its own independent diff presentation and approval turn.
   - **MANDATORY STOP & WAIT**: When fixing bugs or modifying files, you must ALWAYS stop and wait after presenting the proposed diff. Never edit and commit in the same turn without explicit permission for that specific change set.

2. **No Status Polling Loops**:
   - Do not poll running background tasks with repetitive status checks; wait for reactive notifications or user input.

3. **Strict Step-Gated Multi-Repo Push Order**:
   - **Gate 1**: `common-lib` MUST ALWAYS be pushed first.
   - **BLOCK & WAIT**: You are STRICTLY FORBIDDEN from committing or pushing any other repository until the GitHub Actions CI/CD runner on Synology NAS completes and `common-lib` is confirmed merged into `master`.
   - **Gate 2**: Only after `common-lib` has been successfully merged into `master`, proceed to push dependent backend microservices/pipelines (`gexdex-api`, `quant-level-pipeline`, `mm-dex-gex-pipeline`, `discord-quant-bot`, etc.).
   - **Gate 3**: `quant-pwa` MUST ALWAYS be pushed last after all backend services are deployed.
   - Before executing any deployment, you MUST output the explicit **Step-Gating Checklist** in chat and check off each gate only after it is physically verified.

4. **Project Documentation & Continuous Maintenance**:
   - Every project folder must maintain up-to-date documentation (`README.md` or architecture guide) documenting how it works and all design goals.
   - Whenever any code/config is updated in a project folder, revise its documentation accordingly.

5. **Strict Sequential Single-Branch Promotion Workflow**:
   - NEVER push to `develop` and `master` simultaneously or in the same turn.
   - ALWAYS follow this exact linear promotion sequence:
     1. **Push to `develop`**
     2. **Wait for `develop` CI/CD on Synology NAS to complete and verify with user**
     3. **Only after explicit user approval and successful testing, merge `develop` into `master`**
     4. **Push `master`, wait for `master` CI/CD to complete, and verify production**

6. **Mandatory Local End-to-End Testing Before Presenting Diff & Committing**:
   - Before presenting any `git diff` or requesting approval to commit/push, you MUST execute a complete local end-to-end (E2E) test verifying:
     1. All automated unit tests (`pytest`) pass 100% locally.
     2. The affected service(s) spin up cleanly without runtime errors or missing dependencies.
     3. The core API endpoints or CLI flows respond with expected payloads and status codes.
   - Only after physical local verification passes with zero errors can you present the diff and request review.
