# 09 — Cloud Runtime Architecture (`forge-cloud`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Package**: `forge-cloud` (Continuous 24/7 Cloud Engine)

---

## 1. Purpose

Forge Cloud provides the continuous, headless hosting infrastructure for Forge OS. It enables the operating system to run 24/7 without manual intervention, surviving network disconnections, system reboots, and provider rate-limits across diverse environments—from a local developer machine or home Mini PC to cloud VPS instances and GitHub Actions cron runners.

---

## 2. Responsibilities

1. **24/7 Continuous Execution**: Maintain an uninterrupted daemon loop that pulls pending tasks, monitors leases, and executes verification cycles.
2. **Environment Portability**: Provide frictionless deployment across Docker containers, bare-metal Mini PCs, cloud VMs (AWS/GCP/Hetzner), and GitHub Actions.
3. **Cloud Scheduler & Cron Workers**: Coordinate periodic tasks (nightly regression runs, dependency audits, memory garbage collection).
4. **Heartbeat & Liveness Broadcast**: Expose a minimal telemetry heartbeat endpoint for health monitoring.
5. **Zero-Downtime Handover**: Transfer execution state smoothly between a local CLI session and a remote cloud daemon.

---

## 3. Cloud Deployment Topologies

```
┌────────────────────────────────────────────────────────────────────────┐
│                   TOPOLOGY A: LOCAL MINI PC / DESKTOP                  │
│  • Always-on Mini PC (Intel N100 / Mac Mini) running Forge OS daemon   │
│  • Direct local access to host tools, fast disk, local Ollama models   │
├────────────────────────────────────────────────────────────────────────┤
│                   TOPOLOGY B: HEADLESS CLOUD CONTAINER                 │
│  • Lightweight Alpine/Debian Docker container on VPS ($5/mo)           │
│  • Runs daemon in background, syncs state via GitHub & Drive           │
├────────────────────────────────────────────────────────────────────────┤
│                   TOPOLOGY C: SERVERLESS CRON RUNNERS                  │
│  • GitHub Actions workflow triggered hourly or via repository dispatch │
│  • Spins up, claims pending slice, ratchets code, commits, shuts down  │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Heartbeat & Liveness Protocol

```typescript
interface ForgeHeartbeat {
  daemon_id: string;
  uptime_seconds: number;
  active_project: string;
  current_state: "EXECUTING" | "VERIFYING" | "AWAITING_APPROVAL" | "PAUSED";
  active_slice: string | null;
  healthy_providers: string[];
  last_checkpoint_timestamp: string;
}
```

The daemon emits a heartbeat ping every 30 seconds to `.forge/workers/daemon.heartbeat` (and mirrors it to the Google Drive status sheet). If no heartbeat is recorded for 180 seconds, external monitors or backup runners can safely assume node failure and initiate recovery.

---

## 5. Inputs & Outputs
- **Inputs**: Environment variable configurations, OS signals (`SIGINT`, `SIGTERM`), scheduled cron triggers.
- **Outputs**: Running system daemon, health status endpoints, automated checkpoint commits.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Cloud VM out of memory (OOM Kill)*.  
  **Defense**: State is committed to disk and synced to Git on every atomic slice; newly spawned VM resumes from `.forge/state.json` without data loss.
- *Failure Mode 2: Zombie daemon process holding stale locks*.  
  **Defense**: Lease TTL ensures locks expire automatically after 300 seconds of silence.

---

## 7. Interactions with Other Modules
- Hosts `03_RUNTIME_ARCHITECTURE.md` and `06_SCHEDULER.md`.
- Synchronizes with `08_STORAGE_LAYER.md`.
- Enforces credentials securely via `12_SECURITY_MODEL.md`.

---

## 8. Future Extensions
- Multi-region active-passive failover with automatic DNS/webhook switching.
- Telegram/Discord notification bot bridging user approvals directly to mobile.
