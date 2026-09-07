# 01 — Forge OS Vision & Philosophy

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Module**: System Vision & Core Operating Thesis

---

## 1. Purpose

Forge OS is an open, portable, AI-native Operating System designed to coordinate heterogeneous foundation models (ChatGPT, Gemini, Grok, Meta Llama, Claude, Codex) as a unified, autonomous software engineering organization without vendor lock-in.

---

## 2. Responsibilities

1. **Operating System vs. Agent Framework Boundary**: Establish the fundamental distinction between an application-level agent framework and an infrastructure-level agent operating system.
2. **Human Experience Shield**: Enforce the non-negotiable 6-stage human experience: `IDEA → GRILL → PLAN → APPROVE → BUILD → VERIFY → DONE`.
3. **Multi-Model Neutrality**: Treat all AI providers as interchangeable, specialized compute engines rather than proprietary walled gardens.
4. **Architectural Tiering**: Define clear boundaries between Kernel, Runtime, Memory, Storage, Cloud, and Skills.

---

## 3. The Core Thesis: Operating System vs. Agent Framework

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          AGENT FRAMEWORKS                               │
│              (CrewAI, AutoGen, LangGraph, OpenHands)                    │
│  • Ephemeral Python process memory (Lost on crash)                      │
│  • Synchronous conversational loops (Token exhaustion / Hallucination)  │
│  • Chat room metaphor (Agents debating aimlessly in natural language)   │
│  • User micro-manages prompts, tools, and agent definitions             │
└─────────────────────────────────────────────────────────────────────────┘
                                   VS
┌─────────────────────────────────────────────────────────────────────────┐
│                              FORGE OS                                   │
│  • File-backed state persistence (.forge/state.json & .forge/ledger/)   │
│  • Asynchronous ticket & PR workflow (Like a real human dev team)       │
│  • Machine-verifiable Goal Contracts & Ratchet Rollback Circuits        │
│  • Hardware/Provider abstraction: Kernel schedules tasks to best model  │
│  • User interacts ONLY at Goal Definition, Grilling, & Plan Approval    │
└─────────────────────────────────────────────────────────────────────────┘
```

### Comprehensive Landscape Comparison

| Dimension | CrewAI / AutoGen | LangGraph | Claude Code / Codex | OpenHands / Manus | **Forge OS** |
|---|---|---|---|---|---|
| **Metaphor** | Role-playing chatroom | Directed Graph DAG | Single-agent CLI | Browser/Desktop agent | **Deterministic Operating System** |
| **Model Portability** | Adapter wrappers | Custom node binding | Vendor-locked (Anthropic / OpenAI) | Dockerized sandboxes | **Provider-agnostic Capability Router** |
| **State Persistence** | Memory variables / SQLite | Checkpointer DB | Session JSON history | Container filesystem | **Git (Code) + Drive (Memory) + Local Ledger** |
| **Fault Tolerance** | Retry prompt loops | Step checkpointing | Human interrupt | Timeout / Retry | **Hard crash recovery (<3s) + Blind Ratchet Rollback** |
| **Human UX** | Complex configuration | Node programming | Conversational terminal | Autonomous screen recording | **Clean 6-Stage Human Lifecycle (No agent noise)** |

---

## 4. Architectural Boundaries: What Belongs Where

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                 SKILLS                                  │
│            Domain capabilities, tool packs, CLI wrappers                │
├─────────────────────────────────────────────────────────────────────────┤
│                             CLOUD RUNTIME                               │
│        Daemon, 24/7 background scheduler, Docker, VPS, Cron             │
├─────────────────────────────────────────────────────────────────────────┤
│                               MEMORY OS                                 │
│   5-tier memory: Session, Project, Skills, Knowledge, Experience        │
├─────────────────────────────────────────────────────────────────────────┤
│                                RUNTIME                                  │
│       Worker leases, heartbeats, task queue, provider adapters          │
├─────────────────────────────────────────────────────────────────────────┤
│                                KERNEL                                   │
│    Goal Contract, State Machine, Ratchet Rules, Pre-Approval Lockout    │
└─────────────────────────────────────────────────────────────────────────┘
```

1. **Kernel (`gauntlet-operator`)**: Pure, immutable policy. Owns the state machine, semantic states (`PASS`, `FAIL`, `SURPRISE`, `REPLAN_REQUIRED`, `BLOCKED`), Goal Contract schema, and verification gates. Zero runtime dependencies.
2. **Runtime (`forge-runtime`)**: Process management. Owns worker process lifecycles, task leasing, heartbeat monitoring, and crash recovery.
3. **Agent Manager & Router (`forge-scheduler`)**: Capability-based dispatch. Decides dynamically which model (Grok, Gemini, ChatGPT, Llama) receives each task slice based on cost, context size, and reasoning strength.
4. **Memory OS (`forge-memory`)**: Five-layer hierarchical memory engine managing session cache, project knowledge graphs, decision trees, and post-mortems.
5. **Storage Layer (`forge-storage`)**: Dual-vault bridge: GitHub as the authoritative code source; Google Drive as the artifact and checkpoint memory vault.
6. **Cloud Runtime (`forge-cloud`)**: Headless daemon supporting continuous 24/7 execution across bare metal, Docker, and cloud VMs.
7. **Skills (`forge-skills`)**: Decoupled domain plugins and verification harnesses invoked on demand.

---

## 5. The Design Tree of Intent

```
FORGE OS
├── [Human UX Interface]
│   ├── No agent log leakage to user
│   ├── 6 human stages: IDEA → GRILL → PLAN → APPROVE → BUILD → VERIFY → DONE
│   └── Hard Lockout: Zero source code modification before explicit plan approval
├── [Multi-Model Compute Fabric]
│   ├── OpenAI (Architectural planning & high-level reasoning)
│   ├── Google Gemini (Deep research, 1M+ context window, media assets)
│   ├── xAI Grok (High-throughput coding, refactoring, and debugging)
│   └── Meta Llama (Independent, clean-slate blind critic & adversarial auditor)
├── [Dual-Storage Authority]
│   ├── GitHub: Authoritative Git commits, PRs, CI, and locked Goal Contracts
│   └── Google Drive: Persistent checkpoints, model context dumps, and large media
└── [Fault Tolerance & Safety]
    ├── 5 Global Operating Laws (Tackling Hard Tasks)
    ├── Scientific Debug Mantra circuit breaker on 3 consecutive failures
    └── 100% Physical evidence proof gate before declaring DONE
```

---

## 6. Inputs & Outputs
- **Inputs**: User's initial project idea, environment capability probe, human plan approval.
- **Outputs**: Verified, production-grade GitHub repository, synchronized Google Drive memory archive, complete execution ledger.

---

## 7. Failure Modes & Defenses
- *Failure Mode 1: Multi-agent groupthink / sycophancy*.  
  **Defense**: Isolated Clean-Slate Blind Critic reviewing diffs without access to builder reasoning.
- *Failure Mode 2: Premature execution during planning*.  
  **Defense**: Kernel-level tool lockout enforced at the state machine level.
- *Failure Mode 3: Provider API outage or rate limit*.  
  **Defense**: Capability Router transparently re-routes task slices to fallback providers.

---

## 8. Interactions with Other Modules
- Dictates foundational invariants to `02_KERNEL_ARCHITECTURE.md`.
- Governs scheduling requirements for `05_AGENT_MANAGER.md` and `06_SCHEDULER.md`.
- Establishes storage separation rules for `08_STORAGE_LAYER.md`.

---

## 9. Future Extensions
- Federation protocol for peer Forge OS nodes collaborating across organizations.
- Automated token budget arbitrage across spot-priced model API providers.
