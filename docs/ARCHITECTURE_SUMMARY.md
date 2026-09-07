# Forge OS Architecture Summary (v1.0.0-architecture)

```yaml
Title: Forge OS Architecture Summary
Release: v1.0.0-architecture
Status: FOUNDER_FREEZE_RATIFIED
Founder: LiplnwZa
Chief Architect: ChatGPT GPT-5
Implementation Agent: AGY (Gemini CLI)
Repository: liplnwza548/forge-os
```

---

## 1. What is Forge OS?

**Forge OS** is an AI-native, multi-provider artificial intelligence operating system designed to orchestrate untrusted cognitive models (ChatGPT, Gemini, Claude, Grok, Meta AI) into a unified, deterministic software engineering organization without vendor lock-in.

The core user experience follows a clean 6-stage lifecycle:
$$\text{IDEA} \longrightarrow \text{GRILL} \longrightarrow \text{PLAN} \longrightarrow \text{APPROVE} \longrightarrow \text{BUILD} \longrightarrow \text{VERIFY} \longrightarrow \text{DONE}$$

Behind this user experience, Forge OS operates a mathematically pure policy Kernel, an append-only causal event DAG ledger, a 6-tier memory graph fabric, a 4D hermetic skill sandbox, and empirical verification gates.

---

## 2. Architecture Overview & Core Invariants

Forge OS is governed by five Non-Derogable Constitutional Laws ([RFC-0000](rfc/RFC-0000_FORGE_CONSTITUTION.md)):
1. **Absolute Human Primacy**: No AI swarm or autonomous process may bypass Founder HMAC authorization or alter sealed Goal Contracts.
2. **Incorruptible Audit Trail**: Every action is recorded as a parent-linked event in an append-only Causal Event DAG ([RFC-0006](rfc/RFC-0006_PROJECT_LEDGER_PROTOCOL.md)).
3. **Strict Boundary Isolation**: Filesystem modifications are jailed via Scope Shield AST diff analysis ([RFC-0009](rfc/RFC-0009_SCOPE_SHIELD_PROTOCOL.md)) and 4D skill sandboxes ([RFC-0005](rfc/RFC-0005_SKILL_PACKAGE_SPEC.md)).
4. **Empirical Verification Precedence**: Model self-attestations carry zero evidential weight. Monotonic forward progress requires physical exit code proofs ($V_0$–$V_5$, [RFC-0008](rfc/RFC-0008_VERIFICATION_CONTRACT_PROTOCOL.md)).
5. **Unconditional Revocation**: The Founder possesses instant break-glass revocation over any active lease ([RFC-0003](rfc/RFC-0003_WORKER_LEASE_PROTOCOL.md)).

---

## 3. Repository Structure

```
forge-os/
├── .github/                       # Issue templates, PR templates, Governance labels
│   ├── ISSUE_TEMPLATE/            # RFC amendment & defect templates
│   ├── PULL_REQUEST_TEMPLATE.md   # Architectural PR checklist
│   ├── DISCUSSION_TEMPLATE.md     # Governance discussion template
│   ├── labels.md                  # Standardized label registry
│   └── project_board.md           # Suggested GitHub Projects board
├── docs/                          # Architectural documentation corpus
│   ├── rfc/                       # Canonical 13 RFC Specifications (RFC-0000 - RFC-0010)
│   ├── golden-tests/              # Architecture Golden Test Suite & Audit Certificates
│   ├── ratification/              # Formal Ratification Records & Architecture Manifest
│   ├── ARCHITECTURE_SUMMARY.md    # High-level architecture summary (This file)
│   └── GOVERNANCE.md              # RFC Amendment & Governance Workflow
├── CHANGELOG.md                   # Complete architectural release log
├── CONTRIBUTING.md                # Governance contribution guidelines
├── CODE_OF_CONDUCT.md             # Community standards
├── HANDOFF_ARCHITECTURE_v1.0.md   # AI Provider Handoff & Bootstrap Specification
├── LICENSE                        # Apache-2.0 License
├── README.md                      # Primary repository landing page
├── RELEASE_NOTES_v1.0.0.md        # Release notes for GitHub visitors
└── SECURITY.md                    # Security vulnerability reporting protocol
```

---

## 4. RFC Specification Map

| RFC | Title | Level | Core Contribution |
|---|---|---|---|
| [**GLOSSARY**](rfc/GLOSSARY.md) | System Lexicon | Reference | Authoritative vocabulary & canonical data schemas |
| [**RFC-0000**](rfc/RFC-0000_FORGE_CONSTITUTION.md) | Constitution | Level 0 | Supreme legal charter & Non-Derogable Laws |
| [**RFC-0001**](rfc/RFC-0001_KERNEL_CONTRACT.md) | Kernel Contract | Level 1 | Pure policy engine & 13-state deterministic machine |
| [**RFC-0002**](rfc/RFC-0002_MEMORY_GRAPH_PROTOCOL.md) | Memory Graph | Level 1 | 6-tier typed graph fabric ($E_0$–$L_4$) & provenance |
| [**RFC-0003**](rfc/RFC-0003_WORKER_LEASE_PROTOCOL.md) | Worker Lease | Level 1 | Single-writer lease & CAS renewal protocol |
| [**RFC-0004**](rfc/RFC-0004_PROVIDER_CAPABILITY_MATRIX.md) | Capability Matrix | Level 1 | Brand-neutral model routing & critic orthogonality |
| [**RFC-0005**](rfc/RFC-0005_SKILL_PACKAGE_SPEC.md) | Skill Package | Level 1 | Hermetic package spec & universal loopback proxy |
| [**RFC-0006**](rfc/RFC-0006_PROJECT_LEDGER_PROTOCOL.md) | Project Ledger | Level 1 | Append-only Causal Event DAG (17 events) |
| [**RFC-0007**](rfc/RFC-0007_FORGE_RATCHET_PROTOCOL.md) | Forge Ratchet | Level 1 | Two-phase gate & forward progress mechanics |
| [**RFC-0008**](rfc/RFC-0008_VERIFICATION_CONTRACT_PROTOCOL.md) | Verification Contract | Level 1 | Empirical $V_0$–$V_5$ levels & evidence receipts |
| [**RFC-0009**](rfc/RFC-0009_SCOPE_SHIELD_PROTOCOL.md) | Scope Shield | Level 1 | AST diff analysis & automatic replan tripwires |
| [**RFC-0010**](rfc/RFC-0010_TRUST_AND_PROVENANCE_PROTOCOL.md) | Trust Protocol | Level 1 | Cryptographic custody & critic attestations |

---

## 5. Golden Test Suite Overview

The architecture features a 100% verified **Golden Test Suite** located in `docs/golden-tests/`:
- **`COVERAGE_REPORT.md`**: Certifies 100% coverage across RFCs, states, events, verification levels, and memory layers.
- **`FOUNDER_FREEZE_CERTIFICATE.md`**: Formal certificate sealing the architecture version.
- **`FINAL_AUDIT.md`**: Consistency audit report (0 Critical, 0 Major defects).

---

## 6. Future Roadmap & Milestones

- **Milestone v1.0.0-architecture (CURRENT)**: Founder Freeze & Architectural Ratification.
- **Milestone v0.3 (NEXT)**: Runtime Skeleton (`packages/kernel`, `packages/runtime`, `packages/cli`). *Blocked until Founder explicitly authorizes runtime code generation.*
- **Milestone v0.4**: Multi-Provider Distributed Scheduler & Cloud Deployment.
- **Milestone v1.0**: General Production Release.
