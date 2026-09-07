# 11 — State Machine Architecture (`forge-state`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Module**: Formal Finite State Machine & Transition Invariants

---

## 1. Purpose

The State Machine defines the complete, deterministic lifecycle of any initiative managed by Forge OS. It enforces legal state progressions, prohibits unauthorized phase jumps (such as building before plan approval), provides robust exception handling for surprises, and ensures verifiable exit conditions.

---

## 2. Responsibilities

1. **Deterministic State Invariants**: Enforce that the system can exist in exactly one valid state at any instant.
2. **Transition Validation**: Reject any event or transition not explicitly permitted in the formal transition table.
3. **Hard Approval Barrier**: Block transition from planning to execution until explicit human authorization is granted.
4. **Exception Routing**: Route unforeseen contradictions to `SURPRISE` or `REPLAN_REQUIRED` without silent goal changes.
5. **Session Resumability**: Ensure state can be restored cleanly from disk via `RESUMING`.

---

## 3. The Comprehensive Transition Table

| Current State | Event / Trigger | Next State | Condition / Invariant |
|---|---|---|---|
| `IDEA` | Environmental Pre-Survey Complete | `GRILLING` | Repo scanned, initial context loaded |
| `GRILLING` | Decision Frontier Settled | `PLANNING` | All 14 Goal Contract fields confirmed by user |
| `PLANNING` | Dual-Path Evaluated & Best Plan Chosen | `AWAITING_APPROVAL` | "The Bar" acceptance harness defined |
| `AWAITING_APPROVAL` | User commands `APPROVE` | `EXECUTING` | Scope Hash cryptographically sealed |
| `AWAITING_APPROVAL` | User requests architectural changes | `PLANNING` | Re-plan with new constraints |
| `AWAITING_APPROVAL` | User changes fundamental mission | `GRILLING` | Re-open intent interview |
| `EXECUTING` | All slices pass Blind Critic & tests | `VERIFYING` | 100% slices merged into champion |
| `EXECUTING` | Critical assumption disproven | `SURPRISE` | Execution halted immediately |
| `EXECUTING` | Missing tool, credential, or dependency | `BLOCKED` | Awaiting dependency resolution |
| `EXECUTING` | 5 consecutive slice failures | `REPLAN_REQUIRED` | Rollback to champion; re-enter planning |
| `SURPRISE` | Disproved assumption requires new scope | `REPLAN_REQUIRED` | Requires revised Goal Contract |
| `SURPRISE` | Surprise resolved within existing scope | `EXECUTING` | Fix verified with repro test |
| `REPLAN_REQUIRED` | Revised plan synthesized | `AWAITING_APPROVAL` | Requires human approval of new plan |
| `BLOCKED` | Dependency installed / resolved | `EXECUTING` | Resume from active slice |
| `VERIFYING` | Post-integration audit passes with proof| `DONE` | Exit code 0, 100% test pass, clean trace |
| `VERIFYING` | Defect found during outsider audit | `EXECUTING` | Re-open slice with defect citation |
| `ANY STATE` | Process receives pause signal | `PAUSED` | State and leases saved to disk |
| `PAUSED` | User commands resume | `RESUMING` | Re-validate state against Git/Drive |
| `RESUMING` | State validated cleanly | *Prior State* | Resume execution seamlessly |
| `ANY STATE` | Unrecoverable corruption / fatal error | `FAILED` | Human intervention required |

---

## 4. Forbidden Transitions (Hard Invariants)

1. **`IDEA → EXECUTING`**: **STRICTLY FORBIDDEN**.
2. **`GRILLING → EXECUTING`**: **STRICTLY FORBIDDEN**.
3. **`PLANNING → EXECUTING`**: **STRICTLY FORBIDDEN**.
4. **`EXECUTING → DONE`**: **STRICTLY FORBIDDEN** (Must pass through `VERIFYING`).
5. **`SURPRISE → EXECUTING`**: Forbidden if the surprise contradicts approved `scope_hash` without re-approval.

---

## 5. Inputs & Outputs
- **Inputs**: Lifecycle transition requests, tool evaluation results, human approval commands, OS signals.
- **Outputs**: Validated state updates written to `.forge/state.json`, broadcasted audit events in `.forge/ledger.jsonl`.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Subagent attempts to skip approval gate*.  
  **Defense**: Kernel strictly forbids transition to `EXECUTING` unless `user_approved_plan_at` timestamp is signed and verified.

---

## 7. Interactions with Other Modules
- Implemented and enforced by `02_KERNEL_ARCHITECTURE.md`.
- Tracked by `10_PROJECT_LEDGER.md`.
- Monitored by `09_CLOUD_RUNTIME.md`.

---

## 8. Future Extensions
- Branching state machines for concurrent speculative branches merged via git rebase.
