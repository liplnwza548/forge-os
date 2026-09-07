# Changelog — Forge OS

All notable changes to the Forge OS architecture corpus will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [v1.0.0-architecture] - 2026-09-08

### Added — Architecture Freeze & Governance Layer
- **`RFC-0000_FORGE_CONSTITUTION.md`**: Derived supreme governing charter. Added Section 4.2 Non-Derogable Constitutional Laws (Absolute Human Primacy, Incorruptible Audit Trail, Strict Boundary Isolation, Empirical Verification Precedence, Unconditional Revocation). Added ratification metadata schema.
- **`RFC-0001_KERNEL_CONTRACT.md`**: Codified pure mathematical policy engine, 13-state deterministic machine (`UNINITIALIZED` $\to$ `DONE`), canonical 11-field Goal Contract schema, HMAC-SHA256 approval hashing, `EXECUTING` $\to$ `REPLAN_REQUIRED`, and `REPLAN_REQUIRED` $\to$ `GRILLING` state transition paths.
- **`RFC-0002_MEMORY_GRAPH_PROTOCOL.md`**: Created 6-tier typed Memory Graph Fabric ($E_0$ scratch to $L_4$ experience), node provenance chain schema, dynamic trust scoring, and atomic Checkpoint Protocol.
- **`RFC-0003_WORKER_LEASE_PROTOCOL.md`**: Created single-writer task lease protocol, Compare-And-Swap (CAS) lease renewal with nonce validation, task deadlines, and crash recovery.
- **`RFC-0004_PROVIDER_CAPABILITY_MATRIX.md`**: Created brand-neutral semantic capability descriptors, dynamic provider health EWMA tracking, and Builder/Critic orthogonality enforcement.
- **`RFC-0005_SKILL_PACKAGE_SPEC.md`**: Created hermetic skill packaging specification, 4D sandboxing, and universal loopback network proxy confinement.
- **`RFC-0006_PROJECT_LEDGER_PROTOCOL.md`**: Created append-only Causal Event DAG ledger (`parent_event_id`), 17 canonical event types (including `GOAL_AMENDED`), and immutable receipt logging.
- **`RFC-0007_FORGE_RATCHET_PROTOCOL.md`**: Created two-phase Ratchet Gate architecture (Phase 1 Entry Condition vs Phase 2 Advancement Transaction), 5 ratchet tiers, and automatic rollback to Champion Commit.
- **`RFC-0008_VERIFICATION_CONTRACT_PROTOCOL.md`**: Created canonical Verification Contract Protocol, $V_0$–$V_5$ empirical strictness levels, physical evidence receipts, critic orthogonality, timeouts, and Flaky Test Quarantine.
- **`RFC-0009_SCOPE_SHIELD_PROTOCOL.md`**: Created defensive boundary confinement engine, AST diff path traversal analysis, whitelist enforcement, and automatic tripwire to `REPLAN_REQUIRED`.
- **`RFC-0010_TRUST_AND_PROVENANCE_PROTOCOL.md`**: Created end-to-end cryptographic chain of custody, prompt hashing, critic digital attestations, and zero-trust verification algorithms.
- **`GLOSSARY.md`**: Created single authoritative system vocabulary containing 100% of defined concepts, state names, and schema types.
- **`README.md`**: Created master RFC registry, dependency graph, role-based tracks, and **The Forge Boot Sequence** for incoming AI provider workers.
- **`docs/golden-tests/`**: Generated 100% coverage test report (`COVERAGE_REPORT.md`), Founder Freeze Certificate (`FOUNDER_FREEZE_CERTIFICATE.md`), and final consistency audit (`FINAL_AUDIT.md`).
- **`docs/ratification/`**: Created formal Ratification Record (`RATIFICATION_RECORD_v1.0.md`) and machine-readable `ARCHITECTURE_MANIFEST.json`.
- **`HANDOFF_ARCHITECTURE_v1.0.md`**: Created comprehensive AI provider handoff & bootstrap specification.

### Fixed — Architectural Corrections (AC-01 through AC-16)
- **AC-01**: Eliminated Kernel boundary ambiguity; Kernel is strictly referentially transparent policy.
- **AC-02**: Replaced isolated directory trees with a unified, typed Memory Graph Fabric ($E_0$ to $L_4$).
- **AC-03**: Resolved rollback memory desync via atomic Checkpoint Protocol tied to Git Champion Commit.
- **AC-04**: Overrode zombie worker heartbeats with a hard 10-minute monotonic task deadline.
- **AC-05**: Replaced blind lease file renames with Compare-And-Swap (CAS) nonce validation.
- **AC-06**: Replaced vendor brand hardcoding with abstract Semantic Capability Descriptors & lineage tokens.
- **AC-07**: Closed local workstation network escapes via universal loopback proxy confinement.
- **AC-08**: Added canonical `GOAL_AMENDED` event to the ledger event catalog (17 events total).
- **AC-09**: Resolved ledger clock jitter by structuring ledger into a Causal Event DAG using `parent_event_id`.
- **AC-10**: Eliminated Ratchet gate circular deadlocks by splitting into Phase 1 Entry vs Phase 2 Advancement.
- **AC-11**: Resolved symbol collision by renaming Verification Levels to $V_0$–$V_5$, reserving $L_n$ for Memory Layers.
- **AC-12**: Added `approved_at` and `founder_id` to Goal Contract schema for deterministic HMAC verification.
- **AC-13**: Formalized Scope Shield as a standalone specification with direct replanning tripwires.
- **AC-14**: Mandated structured cryptographic provenance chains across all memory nodes and diffs.
- **AC-15**: Formalized dedicated Verification Contract protocol with machine-readable receipt schemas.
- **AC-16**: Established Flaky Test Quarantine protocol to eliminate non-deterministic CI deadlocks.
