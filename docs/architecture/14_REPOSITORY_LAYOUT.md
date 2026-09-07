# 14 — Repository Layout & Monorepo Architecture

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Module**: Repository Topography & Package Boundaries

---

## 1. Purpose

This document defines the complete directory topography and modular package boundaries for the `forge-os` parent operating system repository. It strictly separates policy (Kernel) from mechanism (Runtime/Storage/Agents) while maintaining clean monorepo ergonomics.

---

## 2. The Canonical `forge-os` Tree

```
forge-os/
├── .forge/                         # Local project state, ledger, and Goal Contract
├── .github/
│   └── workflows/                  # CI/CD, nightly regression, and cron runners
├── kernel/                         # Submodule / Vendored core: gauntlet-operator
│   ├── SKILL.md                    # Canonical policy kernel
│   ├── references/                 # Kernel policy specifications
│   └── templates/                  # Canonical contract schemas
├── runtime/                        # Process lifecycle, worker leases, heartbeats
│   ├── src/
│   └── tests/
├── scheduler/                      # Task slicing, DAG engine, builder-critic pairing
│   ├── src/
│   └── tests/
├── memory/                         # 5-layer hierarchical memory engine
│   ├── src/
│   └── tests/
├── providers/                      # Heterogeneous model adapters (OpenAI, Google, xAI, Meta)
│   ├── src/
│   └── tests/
├── storage/                        # Dual-vault synchronization (GitHub REST + Google Drive v3)
│   ├── src/
│   └── tests/
├── agents/                         # Agent pool management, role normalization
│   ├── src/
│   └── tests/
├── skills/                         # Executable verification and domain tool packs
│   └── std/
├── templates/                      # Standardized project blueprints and Goal Contracts
├── docs/                           # Architecture, RFCs, and user manuals
│   ├── architecture/               # 15 Canonical Architecture Documents
│   └── rfcs/                       # Design proposals and RFC process
├── tests/                          # Cross-package end-to-end integration test suites
│   ├── e2e/
│   └── adversarial/
├── examples/                       # Reference workflows and sample autonomous projects
│   ├── quickstart-cli/
│   └── multi-provider-demo/
├── cli/                            # User-facing 'forge' binary entry point
│   ├── src/
│   └── tests/
├── README.md                       # Public documentation and quickstart guide
├── LICENSE                         # MIT License
└── CHANGELOG.md                    # Release history
```

---

## 3. Package Responsibilities & Boundaries

| Directory | Package Name | Responsibility | Dependencies |
|---|---|---|---|
| `kernel/` | `gauntlet-operator` | Pure policy kernel: state machine, contract validation | **Zero Dependencies** |
| `runtime/` | `@forge/runtime` | Worker execution, process leases, heartbeats | OS, `kernel` |
| `scheduler/`| `@forge/scheduler` | Atomic task slicing, DAG routing, ratchet loops | `runtime`, `kernel` |
| `memory/` | `@forge/memory` | 5-tier context compaction, knowledge graphs | `storage` |
| `providers/`| `@forge/providers` | Unified API adapters (OpenAI, Gemini, Grok, Llama)| HTTP client |
| `storage/` | `@forge/storage` | GitHub REST & Google Drive v3 dual-sync | `git`, Drive API |
| `agents/` | `@forge/agents` | Normalized agent worker pool | `providers` |
| `cli/` | `forge` | User-facing terminal binary | All packages |

---

## 4. Inputs & Outputs
- **Inputs**: Modular code packages, test suites, architecture specifications.
- **Outputs**: Distributable CLI binary (`forge`), containerized daemon images, reusable library packages.

---

## 5. Failure Modes & Defenses
- *Failure Mode: Circular dependencies between packages*.  
  **Defense**: Strict unidirectional dependency flow: `CLI -> Scheduler -> Runtime -> Kernel`. Kernel never imports downstream packages.

---

## 6. Interactions with Other Modules
- Houses all 15 architecture specifications in `docs/architecture/`.
- Governs build and release pipelines across all Forge OS subsystems.

---

## 7. Future Extensions
- Plugin registry allowing community developers to publish custom `@forge/skill-*` and `@forge/provider-*` modules.
