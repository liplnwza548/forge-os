# 10 — Project Ledger Architecture (`.forge/`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Module**: On-Disk Canonical Filesystem & Ledger Engine

---

## 1. Purpose

The Project Ledger defines the standardized directory structure and serialization schemas used by Forge OS to persist state, contracts, memory, audit trails, and worker telemetry on disk. It is the single interface between the operating system and the local filesystem.

---

## 2. Responsibilities

1. **Standardized Hierarchy**: Enforce a universal `.forge/` layout across all projects managed by Forge OS.
2. **Append-Only Event Ledger**: Record every state transition, tool call, and critic review in an immutable `.forge/ledger.jsonl` stream.
3. **Atomic State Serialization**: Ensure corruption-proof reading and writing of `state.json` and `goal_contract.json`.
4. **Lock & Lease Directory**: Provide atomic file-lock primitives for worker process isolation.

---

## 3. The Canonical `.forge/` Filesystem Layout

```
.forge/
├── goal_contract.json       # Canonical machine-readable Goal Contract (14 fields + hash)
├── GOAL_CONTRACT.md         # Human-readable contract representation
├── state.json               # Active lifecycle state, current slice, and champion commit
├── ledger.jsonl             # Append-only structured event and telemetry log
├── workers/
│   ├── leases/              # Active task slice locks (<slice_id>.lock)
│   └── daemon.heartbeat     # Daemon liveness timestamp and status ping
├── tasks/
│   ├── ready/               # Decomposed task slices awaiting builder claim
│   ├── in_flight/           # Slices currently being executed by a worker
│   └── completed/           # Ratcheted slices verified by clean-slate critic
├── memory/
│   ├── post_mortems/        # Root-cause analysis records from Debug Circuit
│   └── session_cache/       # Compaction buffers and temporary scratchpads
├── checkpoints/             # Serialized execution snapshots for crash recovery
├── artifacts/               # Generated test harnesses, reports, and build outputs
├── knowledge/               # Ingested repository docs, API specs, and ADRs
├── patterns/                # Golden reusable code templates proven by verification
└── decisions/               # Architectural fork records and trade-off justifications
```

---

## 4. Key Serialization Schemas

### 1. `state.json`
```json
{
  "project_name": "string",
  "current_state": "EXECUTING",
  "active_slice": "slice-04",
  "scope_hash": "sha256-digest",
  "champion_commit": "git-commit-sha",
  "active_worker_id": "worker-grok-01",
  "last_updated": "2026-09-08T02:25:00Z"
}
```

### 2. `ledger.jsonl` (Append-Only Event Stream)
```json
{"ts":"2026-09-08T02:25:01Z","phase":"EXECUTING","slice":"slice-04","actor":"grok","action":"BUILD_COMPLETE","exit_code":0}
{"ts":"2026-09-08T02:25:15Z","phase":"EXECUTING","slice":"slice-04","actor":"llama","action":"CRITIC_VERDICT","verdict":"PASS","evidence":"test/auth.test.ts passed 6/6"}
{"ts":"2026-09-08T02:25:18Z","phase":"EXECUTING","slice":"slice-04","actor":"kernel","action":"RATCHET_COMMIT","new_champion":"c4b9e1"}
```

---

## 5. Inputs & Outputs
- **Inputs**: File writes, state mutations, and worker heartbeats across all subsystems.
- **Outputs**: Immutable audit logs, crash recovery checkpoints, Git-trackable project state.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Corrupted JSON file due to mid-write crash*.  
  **Defense**: Atomic write pattern (`write tmp -> rename to dest`).
- *Failure Mode 2: Uncontrolled log growth*.  
  **Defense**: Log rotation archiving entries older than 30 days to Google Drive.

---

## 7. Interactions with Other Modules
- Read by `02_KERNEL_ARCHITECTURE.md` on startup.
- Written by `03_RUNTIME_ARCHITECTURE.md` during task execution.
- Synchronized by `08_STORAGE_LAYER.md`.

---

## 8. Future Extensions
- Fast SQLite / DuckDB query layer over `.forge/ledger.jsonl` for instant analytics.
- Real-time event streaming via local Unix domain sockets.
