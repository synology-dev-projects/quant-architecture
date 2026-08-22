# 🧭 L8 Principal's Agentic Engineering Workflow: Complete Blueprint

**Source Video:** [L8 Principal's Agentic Engineering Workflow](https://www.youtube.com/watch?v=iQyg-KypKAA&t=958s)  
**Author:** Kun Chen (Former L8 Principal Engineer at Meta, Microsoft, Atlassian)  
**Core Philosophy:** Transition from a "line-by-line keyboard typist" to an **AI Director / Fleet Captain** managing autonomous agent crews across parallel Git worktrees.

---

## 1. The Core Paradigm: "Captain-and-Crew" Architecture

Traditional AI coding uses a single chat window where the human prompts, reads code, edits, and repeats (synchronous 1:1 bottleneck).

Kun Chen's workflow transforms this into an **asynchronous parallel assembly line**:

```mermaid
graph TD
    Human[👨‍💻 Human Engineering Director / Captain] -->|High-Level Goals & Architecture| CaptainAgent[🧭 Lead Architect / Captain Agent]
    
    subgraph ParallelWorktrees [🌲 Parallel Isolated Git Worktrees (Treehouse)]
        CaptainAgent -->|Dispatches Task A| Agent1[👷 Crewmate 1: Feature Dev]
        CaptainAgent -->|Dispatches Task B| Agent2[🐛 Crewmate 2: Bug Fixer]
        CaptainAgent -->|Dispatches Task C| Agent3[⚡ Crewmate 3: Performance Refactor]
        CaptainAgent -->|Dispatches Task D| Agent4[🔍 Crewmate 4: Multi-Agent Reviewer (No-Mistakes)]
    end

    Agent1 -->|PR / Diff| Verifier[🧪 Automated Test & Typecheck Gate]
    Agent2 -->|PR / Diff| Verifier
    Agent3 -->|PR / Diff| Verifier
    Agent4 -->|Review Report| Verifier

    Verifier -->|Approved Diff| Human
    Human -->|One-Click Merge| Master[🚀 Production / Master Branch]
```

---

## 2. The 7 Pillars of the L8 Workflow

### Pillar 1: High-Performance Terminal Ergonomics
* **Tools:** **WezTerm** (GPU-accelerated terminal) + **tmux** (session & window multiplexing) + **Neovim**.
* **Setup:** Split panes into distinct zones:
  - **Left Pane:** Lead Captain / Orchestrator prompt interface.
  - **Middle Panes (Grid):** 4–8 active subagent processes working in separate Git worktrees.
  - **Right Pane:** Live server logs, Docker container status, and test runners.
* **Voice-to-Text Acceleration:** Using rapid voice input (Whisper/Mac dictation) to dictate comprehensive architectural context and user intent faster than typing.

---

### Pillar 2: Context Memory Architecture (Global vs. Local)
AI agents degrade in reasoning quality when context windows are flooded with irrelevant files.
* **Global Memory (`~/.config/agents/memory.md`):** Personal engineering preferences, preferred libraries, naming styles, and communication style.
* **Project Memory (`AGENTS.md` / `GEMINI.md` in repo root):** Architecture boundaries, database schemas, API contracts, forbidden patterns, and testing commands.
* **Zero-Bloat Rule:** Keep memory files tight and declarative (<150 lines). Avoid dumping full source code into prompts; let agents use targeted grep/file lookup tools.

---

### Pillar 3: Parallel Worktree Isolation (`Treehouse`)
Instead of having multiple agents edit files in the same working directory (which causes race conditions, file locking, and broken builds):
* **Git Worktrees:** Each agent runs in its own isolated worktree (`git worktree add ../feature-branch feature-branch`).
* Each agent has its own local dependencies, test runner, and compiler.
* The developer or Captain merges approved worktrees via clean git PRs without branch-switching friction.

---

### Pillar 4: Interactive Planning & Specifications (`AXI` / `Lavish`)
Never let an agent start writing code from an ambiguous prompt.
1. **Interactive RFC / Implementation Plan:** The agent drafts an implementation plan with:
   - Technical design decisions & trade-offs
   - Open questions for human feedback
   - Exact file modifications (`[NEW]`, `[MODIFY]`, `[DELETE]`)
   - Verification test steps
2. **Alignment Gate:** The human reviews the plan, clarifies ambiguous requirements, and clicks "Approve".

---

### Pillar 5: Anti-Bloat & Minimalist Engineering (`Ponytail` / YAGNI)
LLMs tend to over-engineer, adding unnecessary abstractions, wrapper classes, and premature optimizations.
* **Enforce Strict YAGNI:** Agents must only implement what is explicitly requested.
* **Smallest Viable Diff:** Code reviews penalize unnecessary refactors or styling churn in unrelated files.

---

### Pillar 6: Multi-Perspective Automated Code Review (`No-Mistakes`)
Before any code is committed, specialized review agents inspect the git diff from independent angles:
1. **Security Reviewer:** Insecure secrets, SQL injection, authorization bypass, open CORS.
2. **Correctness & Edge Cases:** Null checks, off-by-one errors, timezone mismatches, unhandled exceptions.
3. **Performance & Resource Leaks:** Connection pool disposal, memory leaks, unindexed queries, blocking I/O.
4. **Test Verification:** Ensures regression unit tests cover the new logic.

---

### Pillar 7: Fast Feedback Loops & Continuous Learning
* When a bug is caught or the developer corrects the agent, the learning is immediately saved to the project's `AGENTS.md` or `.agents/rules/`.
* The entire agent fleet gets smarter over time and never repeats the same mistake.

---

## 3. How to Implement This Workflow in Your Quant System

Here is how you can apply the L8 Principal workflow directly to your **`C:\Coding\VSCode\Quant System`** project:

```
Quant System Dev Cycle
├── 1. Plan:     Create/update implementation_plans/<feature>.md (Architecture RFC)
├── 2. Isolate:  Run tasks in parallel across submodules (gexdex-api, quant-pwa, pipelines)
├── 3. Build:    Let agents implement with strict YAGNI & Pydantic typing
├── 4. Verify:   Run ./verify.sh or pytest in isolated test environments
├── 5. Review:   Run multi-agent code review (Security, Concurrency, Performance)
└── 6. Learn:    Update common_config/AGENTS.md with new rules and constraints
```

### Actionable Setup Steps
1. **Worktree Directory:** Create a `worktrees/` directory to spin up parallel pipeline worktrees for `mm-dex-gex-pipeline` and `unusual-option-flow-pipeline`.
2. **Enforce Project Rules:** Keep `implementation_plans/AGENTS.md` updated with guidelines (e.g. *always use UUID for Oracle temp tables*, *never hardcode client IDs in IBKR*, *always use native token streaming in Gateway*).
3. **Multi-Agent Review Gate:** Trigger automated multi-subagent code review on every major PR before pushing to production Synology runners.
