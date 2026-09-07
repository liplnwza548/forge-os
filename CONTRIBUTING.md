# Contributing to Forge OS Architecture

Thank you for your interest in contributing to **Forge OS**!

---

## 1. Architectural Governance First

Forge OS is governed strictly by its **Constitution ([RFC-0000](docs/rfc/RFC-0000_FORGE_CONSTITUTION.md))** and **Non-Derogable Laws**:
1. **Founder Sovereignty** (`LiplnwZa` is the sole ratifying authority).
2. **Independent Verification** (No self-certification allowed).
3. **Empirical Verification Precedence** (Proofs require reproducible exit code 0).
4. **Strict Boundary Isolation** (Path AST whitelisting and 4D sandboxing).
5. **Unconditional Revocation** (Human break-glass control).

---

## 2. RFC Amendment Workflow

Any proposed change to the architecture, state machine, memory graph, ledger schemas, or ratchet gates MUST follow the RFC Amendment Workflow outlined in [**`docs/GOVERNANCE.md`**](docs/GOVERNANCE.md):

1. **Submit RFC Amendment Proposal**: Open an issue using the `RFC Amendment Proposal` template.
2. **Architectural Review**: Evaluated against Constitutional Invariants and Golden Tests.
3. **Founder Ratification**: Requires explicit signature from Founder `LiplnwZa`.

---

## 3. Code Contributions (Milestone v0.3+)

Runtime code contributions will begin during Milestone v0.3. All pull requests must pass:
- Level $V_0$ syntax & linter checks.
- Level $V_1$ strict compilation.
- Level $V_2$ unit tests (100% pass rate).
- Independent Critic review (disjoint provider lineage).
