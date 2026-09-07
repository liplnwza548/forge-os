# 08 — Storage Layer Architecture (`forge-storage`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Package**: `forge-storage` (Dual-Vault Storage Engine)

---

## 1. Purpose

The Storage Layer implements Forge OS's Dual-Vault architecture. It delegates responsibilities strictly between two storage paradigms: **GitHub** serves as the canonical version-controlled source of truth for code and contracts, while **Google Drive** serves as the infinite warm/cold vault for heavy artifacts, context snapshots, and media binaries.

---

## 2. Responsibilities

1. **Dual-Vault Boundary Enforcement**: Guarantee that code lives exclusively in Git and heavy binary state lives in Google Drive.
2. **Cryptographic State Hashing**: Compute and verify SHA-256 tree hashes across both vaults to ensure end-to-end auditability.
3. **Resilient Synchronization**: Manage multipart resumable uploads and background incremental sync to Google Drive without blocking execution.
4. **Conflict Resolution**: Enforce single-writer lock mechanics to eliminate split-brain write conflicts.
5. **Local Offline Cache**: Maintain a local disk cache (`.forge/cache/`) permitting full functionality during temporary network dropouts.

---

## 3. Dual-Vault Division of Authority

```
┌────────────────────────────────────────────────────────────────────────┐
│                   CANONICAL SOURCE OF TRUTH (GITHUB)                   │
│  • Production Source Code & Unit/Integration Test Suites               │
│  • Canonical Goal Contracts (.forge/goal_contract.json)                │
│  • Project State Transitions (.forge/state.json)                       │
│  • Task PRs, Review Threads, and Git Champion Tags                     │
│  • CI/CD Workflows (.github/workflows/)                                │
└────────────────────────────────────────────────────────────────────────┘
                                   AND
┌────────────────────────────────────────────────────────────────────────┐
│                      MEMORY & ASSET VAULT (GOOGLE DRIVE)               │
│  • Serialized Model Checkpoints & Context Dumps (.forge/checkpoints/)  │
│  • Multi-Gigabyte Media Files (Audio, Video, Raw Recordings)          │
│  • Cross-Session Knowledge Base & Historical Scraping Logs             │
│  • Complete Raw Tool Telemetry & JSONL Audit Trails                    │
│  • Emergency Cold Backups of Entire Project State                      │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Synchronization & Conflict Resolution Protocol

```
[Local Mutation on Task Slice]
              │
              ├──> [Text / Code Change] ────> Commit to Git Branch ──> Push to GitHub Remote
              │
              └──> [Binary / Checkpoint] ───> Write to Local Cache ──> Background Sync to Google Drive
                                                                          │
                                                                   Compute SHA-256
                                                                          │
                                                                          ▼
                                                               Record Asset ID + Hash in
                                                                .forge/ledger.jsonl (Git)
```

### Conflict Resolution Invariants:
1. **Git Authoritative Invariant**: In any conflict between a code file on disk and GitHub remote, Git remote branch state takes precedence.
2. **Drive Manifest Invariant**: Large assets in Drive are referenced by SHA-256 hash in Git. If a Drive file hash does not match the manifest in Git, the file is rejected and re-downloaded.
3. **Atomic Commit**: A milestone is marked complete only when both GitHub commit SHA and Drive manifest IDs are sealed in `.forge/state.json`.

---

## 5. Inputs & Outputs
- **Inputs**: Modified project code, generated binaries/media, checkpoint serialization streams.
- **Outputs**: GitHub commit SHAs, Google Drive File IDs, unified sync telemetry.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Google Drive API rate limit (User Rate Limit Exceeded)*.  
  **Defense**: Asynchronous queue with exponential backoff; local disk acts as write-ahead log until sync completes.
- *Failure Mode 2: Git push rejected (non-fast-forward)*.  
  **Defense**: Rebase against origin champion; if merge conflict occurs, halt and trigger `SURPRISE` state.

---

## 7. Interactions with Other Modules
- Stores checkpoints produced by `04_MEMORY_OS.md`.
- Verifies code integrity for `13_VERIFICATION_MODEL.md`.
- Governed by security boundaries in `12_SECURITY_MODEL.md`.

---

## 8. Future Extensions
- Pluggable storage providers: S3 / Cloudflare R2 adapter mirroring Google Drive.
- Peer-to-peer IPFS pinning for immutable public release artifacts.
