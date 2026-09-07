# 02 — Kernel Architecture (`gauntlet-operator`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Package**: `gauntlet-operator` (Core Policy Kernel)

---

## 1. Purpose

The Kernel is the authoritative policy engine of Forge OS. It defines and strictly enforces the conceptual state machine, semantic states, Goal Contract immutability, builder-critic ratchet invariants, and human approval gates. 

To maintain total mathematical integrity, the Kernel contains **zero execution or runtime logic**—it acts purely as a deterministic governor over the operating system.

---

## 2. Responsibilities

1. **State Machine Enforcement**: Govern the lifecycle (`IDEA` → `GRILLING` → `GOAL_READY` → `PLANNING` → `AWAITING_APPROVAL` → `EXECUTING` → `VERIFYING` → `DONE`).
2. **Semantic Gate Control**: Enforce transitions for `PASS`, `FAIL`, `BLOCKED`, `SURPRISE`, and `REPLAN_REQUIRED`.
3. **Pre-Approval Lockout**: Programmatically block all write and mutation operations on project code during stages 1 through 4.
4. **Goal Contract Governance**: Validate the 14 canonical fields and seal the contract with a cryptographic `scope_hash`.
5. **Ratchet Invariant Enforcement**: Mandate that working code only advances to the reigning champion upon an independent clean-slate critic `PASS` and zero test regressions.

---

## 3. Kernel Isolation: Policy vs. Mechanism

```
┌────────────────────────────────────────────────────────────────────────┐
│                        FORGE OS KERNEL (POLICY)                        │
│                         (gauntlet-operator)                            │
│  • What state are we in?                                               │
│  • Is this transition legal?                                           │
│  • Has the user approved the plan?                                     │
│  • Does the Goal Contract hash match?                                  │
│  • Did the slice pass The Bar?                                         │
│  • Has physical proof been attached?                                   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Directs & Audits
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                       FORGE OS RUNTIME (MECHANISM)                     │
│  • Spawning worker child processes                                     │
│  • Invoking model REST APIs (OpenAI, Google, xAI, Meta)                 │
│  • Git push / pull / branch manipulation                               │
│  • Google Drive multipart uploads / downloads                          │
│  • Heartbeat leases, timers, and retry loops                           │
└────────────────────────────────────────────────────────────────────────┘
```

The Kernel never opens sockets, never makes HTTP calls, and never manages threads. It reads and verifies state declarations in `.forge/state.json` and `.forge/goal_contract.json`.

---

## 4. The 5 Global Operating Laws (Embedded Invariants)

The Kernel mandates compliance with the 5 Global Operating Laws:
1. **Investigate Unknowns**: Unverified external library or API assumptions are rejected during planning.
2. **Never Stack Unverified Steps**: A slice cannot be scheduled until its prerequisite slice has attained `PASS` and champion status.
3. **Verify Before Assertion**: Verbal declarations of completion without machine test outputs are rejected by the verification gate.
4. **Surprising Result → Stop & Reassess**: Any contradiction of an approved assumption immediately shifts state to `SURPRISE`.
5. **Repeated Failure → Challenge Diagnosis**: 3 consecutive failed cycles on a slice halt speculative edits and trigger the 4-step Debug Mantra.

---

## 5. Inputs & Outputs
- **Inputs**: User prompts, drafted Goal Contracts, slice test results, blind critic verdicts, human approval tokens.
- **Outputs**: Verified state transitions, sealed Scope Hashes, ratchet champion promotions, rollback commands, completion approvals.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Corrupted or manually edited state file*.  
  **Defense**: The Kernel verifies the state file against the append-only `.forge/ledger.jsonl` audit log on startup. Discrepancies force state to `BLOCKED`.
- *Failure Mode 2: Premature builder execution*.  
  **Defense**: Read-only tool lockout strictly enforced prior to user confirmation in `AWAITING_APPROVAL`.
- *Failure Mode 3: Goal drift during implementation*.  
  **Defense**: Cryptographic Scope Hash verification on every slice integration.

---

## 7. Interactions with Other Modules
- Invoked by `03_RUNTIME_ARCHITECTURE.md` to authorize every task transition.
- Validates task inputs dispatched by `06_SCHEDULER.md`.
- Gates synchronization triggers for `08_STORAGE_LAYER.md`.

---

## 8. Future Extensions
- Multi-signature cryptographic approval gates for multi-stakeholder enterprise teams.
- Formal TLA+ verification of Kernel state transitions.
