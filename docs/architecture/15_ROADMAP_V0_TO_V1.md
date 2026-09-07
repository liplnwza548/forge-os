# 15 — Roadmap: v0.2 to v1.0 & The Founder Critic

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS Evolution  
> **Module**: Versioned Milestones & Deep Adversarial Critique

---

## 1. Purpose

This document outlines the strategic engineering roadmap of Forge OS from initial architectural genesis (v0.2) to general availability (v1.0). It concludes with the **Founder Critic Pass**, rigorously exposing missing abstractions, dangerous assumptions, and future RFC requirements.

---

## 2. Versioned Roadmap & Milestones

```
  v0.2 RUNTIME ──> v0.3 MEMORY OS ──> v0.4 FORGE CLOUD ──> v0.5 SDK ──> v0.6 TEAMS ──> v1.0 STABLE
  (Kernel Lock &    (5-Layer Memory    (24/7 Daemon &    (Extensible     (Heterogeneous   (Production GA,
   Worker Leases)    & Knowledge Graph) Dual Storage)     Developer API)  Specialists)     Zero Lock-in)
```

### v0.2 — The Runtime Engine (Foundation)
- **Milestone 2.1**: Monorepo layout & Kernel integration (`gauntlet-operator`).
- **Milestone 2.2**: Worker lifecycle manager and PID-backed lease locking.
- **Milestone 2.3**: Basic provider adapters (OpenAI, Gemini, xAI, Anthropic).
- **Milestone 2.4**: Local `.forge/` filesystem and `state.json` crash recovery (<3s).

### v0.3 — Memory OS (Intelligence & Knowledge)
- **Milestone 3.1**: Implementation of the 5-layer hierarchical memory engine (L0–L4).
- **Milestone 3.2**: Context compaction algorithm preserving the 0–40% "Smart Zone".
- **Milestone 3.3**: Decision Graph and Post-Mortem Store with automated garbage collection.
- **Milestone 3.4**: Local vector index for semantic retrieval over codebase knowledge.

### v0.4 — Forge Cloud (24/7 Autonomy)
- **Milestone 4.1**: Headless 24/7 daemon loop with heartbeat broadcasting.
- **Milestone 4.2**: Dual-Vault synchronization (GitHub REST + Google Drive v3).
- **Milestone 4.3**: Dockerized container deployment for bare-metal Mini PCs and VPS nodes.
- **Milestone 4.4**: GitHub Actions serverless cron runner integration.

### v0.5 — Developer SDK & CLI
- **Milestone 5.1**: Production-grade `forge` CLI with interactive TUI progress indicators.
- **Milestone 5.2**: Extensible Plugin SDK for custom tool packs (`@forge/skill-*`).
- **Milestone 5.3**: Programmatic TypeScript / Python client libraries for embedding Forge OS.

### v0.6 — Multi-Agent Specialized Teams
- **Milestone 6.1**: Full capability router scoring latency, quotas, and benchmark strength.
- **Milestone 6.2**: Dynamic fallback cascades during provider outages (HTTP 429/503).
- **Milestone 6.3**: Clean-slate critic context wiping with verifiable blind scoring.

### v1.0 — Forge OS Stable (Enterprise & Sovereign Release)
- **Milestone 1.0**: Formally verified Kernel state transitions (zero illegal state transitions).
- **Milestone 1.1**: Multi-tenant workspace isolation and hardware security module (HSM) signing.
- **Milestone 1.2**: Comprehensive 100-case adversarial regression test suite passing in CI.

---

## 3. THE FOUNDER CRITIC (MANDATORY ADVERSARIAL PASS)

> *"If Forge OS aims to become Windows/macOS for AI projects, what is fundamentally missing, weak, or dangerous about this architecture?"*

### 1. Missing Modules (Blind Spots)
1. **Virtual Filesystem Overlay (VFS)**: Right now, workers write directly to the working directory, relying on Git to rollback. A true OS needs a Copy-On-Write (CoW) virtual filesystem layer so that failed slices leave zero disk residue without needing `git reset`.
2. **Inter-Agent IPC & Event Bus**: Asynchronous file polling via `.forge/ledger.jsonl` works for CLI tasks, but will experience latency bottlenecks under high-frequency agent interactions. A lightweight IPC socket layer (like domain sockets or Redis Pub/Sub) will be required.
3. **Hardware Device Driver Layer**: An OS needs drivers. In AI terms, "drivers" are standardized adapters for local GPU inference (CUDA, Metal/Apple Silicon, ROCm, Vulkan) to support sovereign offline execution.

### 2. Weak Abstractions
1. **Google Drive as Object Storage**: Google Drive is fundamentally designed for human document collaboration, not machine-speed object storage. Under high-frequency sync, Google Drive API will enforce aggressive per-user rate limits.
   - *RFC Solution*: Abstract `StorageVault` interface so Google Drive is just one plugin; support S3, MinIO, Cloudflare R2, and local NAS as drop-in alternatives.
2. **Binary Critic Voting (PASS / FAIL)**: Real code reviews have nuance (e.g. "approved with minor nit"). Forcing strict binary PASS/FAIL can cause thrashing loops over trivial style preferences.
   - *RFC Solution*: Introduce semantic severity thresholds (Blocker vs. Advisory Nit).

### 3. Dangerous Assumptions
1. **Assumption that Model APIs Are Deterministic**: Models change weights without notice, and temperature 0 does not guarantee bit-exact reproducibility.
   - *Mitigation*: The test harness must test observable behavior and contract invariants, never exact token strings.
2. **Assumption that Human Approval is Instantaneous**: In a 24/7 cloud runtime, if Forge OS halts at `AWAITING_APPROVAL` at 3:00 AM, the entire system stalls until the user wakes up.
   - *Mitigation*: Pre-authorized "Action Bands" (from Team Constitution): safe autonomous exploration within defined budget boundaries, pausing only on breaking scope or cost ceilings.

### 4. Technical Debt to Prevent Early
- Avoid hardcoding vendor-specific prompt wrappers inside core scheduler logic.
- Avoid tight coupling between Git CLI invocations and the Kernel state engine.

### 5. Future RFC Pipeline
- **RFC-001**: `Forge VFS` — In-Memory Copy-on-Write Filesystem for Worker Isolation.
- **RFC-002**: `Universal Storage Driver API` — Unified abstraction across Google Drive, S3, R2, and Local POSIX.
- **RFC-003**: `Mobile Approval Bridge` — Encrypted webhook gateway connecting `AWAITING_APPROVAL` to iOS/Android push notifications and Telegram.
- **RFC-004**: `Autonomous Budget Governor` — Real-time token arbitrage across decentralized inference endpoints.

---

## 4. Inputs & Outputs
- **Inputs**: Architectural synthesis from Documents 01 through 14, empirical landscape analysis.
- **Outputs**: Prioritized development roadmap, open research questions, formal RFC backlog.

---

## 5. Interactions with Other Modules
- Coordinates milestone release criteria across all Forge OS subsystems.
- Establishes quality gates for transitioning from v0.2 to v1.0.
