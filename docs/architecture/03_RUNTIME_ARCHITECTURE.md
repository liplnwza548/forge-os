# 03 — Runtime Architecture (`forge-runtime`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Package**: `forge-runtime` (Process & Worker Engine)

---

## 1. Purpose

The Forge OS Runtime provides the execution machinery that brings Kernel policies to life. It manages worker processes, enforces resource leases, monitors heartbeats, handles provider retries, orchestrates queue ownership, and ensures deterministic crash recovery in under 3 seconds.

---

## 2. Responsibilities

1. **Worker Process Lifecycle**: Spawn, isolate, monitor, and gracefully terminate worker subagents and model subprocesses.
2. **Task Queue & Lease Management**: Grant exclusive time-bounded leases to worker agents to prevent write collisions and split-brain race conditions.
3. **Heartbeat Protocol**: Continuously monitor worker health and reclaim abandoned tasks.
4. **Resilient Retry Engine**: Exponential backoff and jitter for transient network or provider API failures (HTTP 429, 502, 503).
5. **Deterministic Crash Recovery**: Restore full operating state from local disk and remote storage following host crashes (`kill -9`, power loss, container reboot).

---

## 3. Worker Lifecycle & Lease Architecture

```
                 ┌──────────────────────────────────────┐
                 │       Task Slice Enters Queue        │
                 │         (.forge/tasks/ready/)        │
                 └──────────────────┬───────────────────┘
                                    │
                                    ▼
                 ┌──────────────────────────────────────┐
                 │        Worker Lease Claimed          │
                 │   - Exclusive Lock: PID + WorkerID   │
                 │   - Lease TTL: 300 seconds           │
                 └──────────────────┬───────────────────┘
                                    │
                                    ▼
                 ┌──────────────────────────────────────┐
                 │        Heartbeat Daemon Active       │
                 │  - Emits heartbeat every 15 seconds  │
                 │  - Extends lease if healthy          │
                 └──────────┬────────────────┬──────────┘
                            │                │
            Worker Healthy  │                │ Worker Stalls / Crashes
                            ▼                ▼
┌─────────────────────────────────┐   ┌─────────────────────────────────┐
│     Slice Finishes Execution    │   │      Lease TTL Expires (>300s)  │
│  - Diff & Test Output Submitted │   │  - Task Reclaimed to Queue      │
│  - Lease Released Cleanly       │   │  - Worker Process Terminated    │
└─────────────────────────────────┘   └─────────────────────────────────┘
```

### Worker Lease Specification
- **Storage**: `.forge/workers/leases/<task_id>.lock`
- **Fields**: `task_id`, `worker_id`, `provider`, `claimed_at`, `expires_at`, `heartbeat_seq`.
- **Concurrency Invariant**: Exactly ONE builder worker may hold a lease on a slice's working directory at any given moment.

---

## 4. Crash Recovery Protocol (<3 Second Resume)

When the Forge OS daemon starts or recovers from an unexpected termination:
1. **Sweep**: Read `.forge/state.json` and active worker locks in `.forge/workers/leases/`.
2. **Reclaim**: Any lease whose `expires_at` is in the past is marked abandoned. Active git branches are reset to the last confirmed champion commit.
3. **Re-anchor**: Parse `.forge/ledger.jsonl` up to the last verified transaction.
4. **Resume**: Re-queue the unfinished slice and resume the execution daemon without re-asking any resolved questions.

---

## 5. Inputs & Outputs
- **Inputs**: Scheduled tasks from `06_SCHEDULER.md`, Kernel authorization from `02_KERNEL_ARCHITECTURE.md`, model API responses.
- **Outputs**: Process execution outputs, test logs, heartbeat telemetry, lease release events.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Worker process hangs on a blocking network call*.  
  **Defense**: Hard process-level timeout kills the subprocess when heartbeat TTL expires.
- *Failure Mode 2: Host reboot during file write*.  
  **Defense**: Atomic write-and-rename pattern (`write to .tmp` then `mv to target`).
- *Failure Mode 3: Provider API rate limit burst*.  
  **Defense**: Exponential backoff with decorrelated jitter; automatic escalation to Capability Router if rate-limit persists beyond 3 retries.

---

## 7. Interactions with Other Modules
- Receives tasks from `06_SCHEDULER.md`.
- Enforces state boundaries defined by `02_KERNEL_ARCHITECTURE.md`.
- Logs every state change to `10_PROJECT_LEDGER.md`.

---

## 8. Future Extensions
- Containerized sandbox isolation (gVisor / Firecracker microVMs) per worker task slice.
- WebAssembly (Wasm) runtime for client-side sandboxed validation.
