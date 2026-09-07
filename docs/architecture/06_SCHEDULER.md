# 06 — Scheduler Architecture (`forge-scheduler`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Package**: `forge-scheduler` (Task Slicing & Dispatch Engine)

---

## 1. Purpose

The Scheduler breaks approved architectural milestones into atomic, verifiable task slices and dispatches them across heterogeneous worker pools. It pairs builders with independent blind critics, manages dependency trees, and advances the ratchet mechanism based strictly on objective verification.

---

## 2. Responsibilities

1. **Task Decomposition**: Slice approved plans into atomic execution units (target: ≤100 lines of change).
2. **Builder-Critic Pairing**: Guarantee that the builder and critic for any slice are distinct model instances.
3. **DAG Dependency Management**: Enforce execution order based on prerequisites (Operating Law: *Never stack unverified steps*).
4. **Ratchet Coordination**: Advance the project state upon Critic `PASS`; trigger working tree rollback upon Critic `FAIL`.
5. **Circuit Breaker Invocation**: Escalate to Debug Mantra after 3 consecutive failures; halt and replan after 5 failures.

---

## 3. The Slicing & Scheduling Pipeline

```
               ┌──────────────────────────────────────────────┐
               │         Approved Plan (from Phase 4)         │
               └──────────────────────┬───────────────────────┘
                                      │
                                      ▼
               ┌──────────────────────────────────────────────┐
               │             Atomic Decomposition             │
               │   Slice 1 ──> Slice 2 ──> Slice 3 (DAG)      │
               └──────────────────────┬───────────────────────┘
                                      │
                         For Each Slice in DAG:
                                      │
                                      ▼
               ┌──────────────────────────────────────────────┐
               │           Dispatch Builder Worker            │
               │         (e.g. Grok Bot / Code Model)         │
               └──────────────────────┬───────────────────────┘
                                      │ Emits Slice Diff + Tests
                                      ▼
               ┌──────────────────────────────────────────────┐
               │        Dispatch Fresh-Context Critic         │
               │         (e.g. Meta Llama / Claude)           │
               └──────────────────────┬───────────────────────┘
                                      │
                         Evaluates against The Bar
                                      │
                        ┌─────────────┴─────────────┐
                        │                           │
                     Critic FAIL                 Critic PASS
                        ▼                           ▼
          ┌───────────────────────────┐ ┌───────────────────────────┐
          │  Rollback to Last Champion│ │ Commit to Reigning Champ  │
          │  Log Failure Citations    │ │ Advance to Next DAG Slice │
          │  Cycle Count + 1          │ └───────────────────────────┘
          └─────────────┬─────────────┘
                        │
             3 Fails?   ▼
            Trigger Debug Circuit
```

---

## 4. Scheduling Invariants
- **Independence Invariant**: `Slice.builder.model !== Slice.critic.model` (or strictly isolated, wiped-context sessions).
- **Prerequisite Invariant**: Slice $K$ cannot enter the ready queue until Slices $1 \dots K-1$ have attained `PASS` and merged into the champion branch.
- **Budget Ceiling**: Maximum 5 ratchet cycles per slice; maximum 20 total cycles per phase.

---

## 5. Inputs & Outputs
- **Inputs**: Approved Goal Contract and milestone breakdown from `02_KERNEL_ARCHITECTURE.md`, worker pool availability from `05_AGENT_MANAGER.md`.
- **Outputs**: Active task leases in `.forge/workers/leases/`, committed champion tags, rollback signals.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Deadlock in task DAG*.  
  **Defense**: Graph cycle detection runs prior to queuing tasks; cyclic dependencies reject plan back to `PLANNING`.
- *Failure Mode 2: Builder-critic thrashing loop*.  
  **Defense**: Diminishing returns detector terminates loop after 2 consecutive identical failures.

---

## 7. Interactions with Other Modules
- Consults `07_CAPABILITY_ROUTER.md` to select model pairings.
- Dispatches processes through `03_RUNTIME_ARCHITECTURE.md`.
- Governed by state transitions in `02_KERNEL_ARCHITECTURE.md`.

---

## 8. Future Extensions
- Speculative execution of independent DAG branches in isolated parallel git worktrees.
- Reinforcement learning scheduler optimizing for minimum API cost per successful slice.
