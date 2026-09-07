# 13 — Verification Model Architecture (`forge-verify`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Module**: Multi-Tier Verification & Scrutiny Engine

---

## 1. Purpose

The Verification Model guarantees that Forge OS never declares any task `DONE` based on verbal model claims, hallucinations, or superficial passes. It establishes multi-dimensional testing barriers ("The Bar") that demand machine-verifiable physical evidence before advancing the project ratchet.

---

## 2. Responsibilities

1. **Physical Evidence Gatekeeping**: Require that all completions cite exit code 0, test stdout/stderr, and concrete file diffs.
2. **Multi-Tiered Verification**: Orchestrate testing across 7 distinct dimensions (Unit, Integration, Human, Adversarial, Resume, Cloud, and Provider Fallback).
3. **Outsider Scrutinize Audit**: Conduct end-to-end call-path tracing across integrated systems prior to final project conclusion.
4. **Clean-Slate Critic Protocol**: Provide context-wiped evaluations free from builder excuses or rationales.

---

## 3. The 7-Tier Verification Matrix

| Verification Tier | Focus & Target | Execution Mechanism | Pass Requirement |
|---|---|---|---|
| **1. Unit Verification** | Isolated functions, classes, and types | `npm test` / `pytest` / `cargo test` | 100% pass; 0 failures |
| **2. Integration Verification** | Inter-module contracts & storage sync | End-to-end API test suites | Real network/file operations succeed |
| **3. Human Verification** | 6-Stage UX simplicity & Goal Contract | User approval package review | Explicit affirmative user response |
| **4. Adversarial Verification** | Edge cases, empty inputs, bad schemas | Clean-slate blind critic evaluation | Critic issues `PASS` with citations |
| **5. Resume Verification** | Crash recovery and state re-anchoring | Hard kill simulation (`kill -9`) | System restores state in <3 seconds |
| **6. Cloud Verification** | 24/7 daemon liveness & cron execution | Heartbeat monitoring | Continuous heartbeat emitted >24h |
| **7. Provider Fallback** | Resilience under model rate limits (429) | Simulated network fault injection | Router automatically falls back to secondary provider |

---

## 4. The Outsider Scrutinize Protocol (Stage 6: VERIFY)

Before transitioning to `DONE`, the system executes the 4-step Scrutinize audit:
1. **Intent Audit**: Verify all 14 Goal Contract criteria are satisfied and zero Non-Goals were built.
2. **Call-Path Trace**: Trace execution from external entry point down to final database, disk, or network mutation.
3. **Adversarial Edge Test**: Probe edge cases (null payloads, unauthorized calls, malformed tokens).
4. **Physical Hygiene Check**: Ensure zero debug probes (`[DBG-...]`) or temporary files remain in the repository.

---

## 5. Inputs & Outputs
- **Inputs**: Modified code, executed test logs, clean-slate critic assessments, system telemetry.
- **Outputs**: Verified Proof-of-Completion report, terminal reproducibility commands for the human operator.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Vacuous test pass (e.g. `assert True`)*.  
  **Defense**: Scrutinize audits test source code to ensure assertions genuinely exercise production code paths.
- *Failure Mode 2: Test passes locally but fails in CI*.  
  **Defense**: Hermetic test runner isolates environment variables and pins dependencies before running tests.

---

## 7. Interactions with Other Modules
- Gates state transitions in `02_KERNEL_ARCHITECTURE.md`.
- Executed by `06_SCHEDULER.md` during task ratcheting.
- Audits integrity of `08_STORAGE_LAYER.md`.

---

## 8. Future Extensions
- Automated mutation testing to verify test suite quality by injecting intentional bugs.
- Formal property-based testing using QuickCheck / Hypothesis harnesses.
