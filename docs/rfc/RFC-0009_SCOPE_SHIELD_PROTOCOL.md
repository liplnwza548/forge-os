# RFC-0009: Scope Shield Protocol

```yaml
RFC: 0009
Title: Scope Shield Protocol
Status: FOUNDER_FREEZE_RELEASE_CANDIDATE
Author: AGY (Implementation Agent)
Founder: LiplnwZa
Chief Architect: ChatGPT GPT-5
Target: Forge OS Core Governance (v0.2+)
Created: 2026-09-08
Supersedes: None
Authority: LEVEL 1 (Subordinate to RFC-0000, Peer to RFC-0001)
```

---

## 1. Purpose

This specification establishes the **Scope Shield Protocol** for Forge OS.

In autonomous software development, autonomous agents frequently exhibit "helpful scope creep"—refactoring adjacent files, updating unrelated build scripts, adding unapproved third-party dependencies, or altering core contracts without authorization.

The **Scope Shield** is the definitive defensive boundary enforcement engine of Forge OS. It operates as an impenetrable, zero-bypass tripwire: it confines all worker modifications strictly to the `approved_scope` whitelist defined in the sealed Goal Contract. Any attempt to touch, create, delete, or link an unapproved file is intercepted before git commit, immediately aborting the slice and transitioning the system from `EXECUTING` to `REPLAN_REQUIRED`.

---

## 2. Scope

1. **In-Scope**:
   - Boundary validation algorithms: path AST analysis, glob matching, symlink traversal detection.
   - Verification of `approved_scope` confinement prior to Critic dispatch.
   - Dynamic Scope Hash computation and tripwire mechanics.
   - Immediate exception routing: triggering `EXECUTING -> REPLAN_REQUIRED`.
2. **Out-of-Scope**:
   - Host OS kernel-level seccomp or AppArmor profile generation (managed by host deployment infrastructure).

---

## 3. Definitions

All terms conform to [`GLOSSARY.md`](GLOSSARY.md). Key terms:
- **Scope Shield**: The defensive boundary validation engine intercepting out-of-scope modifications.
- **Approved Scope**: The whitelist array of glob patterns defined in `GoalContract.constraints.approved_scope`.
- **Scope Tripwire**: The kernel-level exception mechanism triggered when a modification violates approved boundaries.
- **Path AST Analysis**: Normalization and lexical deconstruction of target filepaths to defeat directory traversal attacks.

---

## 4. Invariants

1. **Strict Confinement Invariant**: No worker modification may touch, modify, create, or delete any file or directory not explicitly matched by `approved_scope`.
2. **Deterministic Pre-Dispatch Interception**: The Scope Shield must validate the target file list before a task slice is dispatched to a Builder, and re-validate the physical diff before dispatching to a Critic.
3. **Instant Tripwire Reversion**: A Scope Shield violation unconditionally triggers an immediate state transition from `EXECUTING` to `REPLAN_REQUIRED` and resets the workspace to the last Champion Commit.
4. **Symlink Traversal Ban**: Any symlink targeting a path outside the repository root or resolving to an unapproved directory trips the Scope Shield as a high-severity security violation.

---

## 5. Interfaces & Schemas

### 5.1 Scope Shield Verification Pipeline

```mermaid
flowchart TD
    DIFF["Worker Emits Diff / Modified File List"] --> NORM["Normalize Paths (filepath.Clean & RelPath)"]
    NORM --> TRAV{"Check Symlink / Traversal Escape?"}
    
    TRAV -->|YES (Escape Detected)| TRIP["TRIPWIRE: Emit V-03 Violation"]
    TRAV -->|NO (Contained in Repo)| GLOB["Match Paths against approved_scope Globs"]
    
    GLOB --> MATCH{"All Files Matched in approved_scope?"}
    MATCH -->|YES| PASS["SCOPE SHIELD PASS -> Proceed to Critic Verification"]
    MATCH -->|NO (Unapproved Touch)| TRIP

    TRIP --> KERNEL["Kernel State Transition: EXECUTING -> REPLAN_REQUIRED"]
    KERNEL --> PURGE["Revert Workspace: git reset --hard champion_commit"]
    PURGE --> LOG["Emit SCOPE_SHIELD_TRIPPED Event to Ledger"]

    classDef proc fill:#2563eb,stroke:#1d4ed8,stroke-width:2px,color:#fff;
    classDef branch fill:#d97706,stroke:#b45309,stroke-width:2px,color:#fff;
    classDef pass fill:#059669,stroke:#047857,stroke-width:2px,color:#fff;
    classDef trip fill:#b91c1c,stroke:#7f1d1d,stroke-width:2px,color:#fff;

    class DIFF,NORM,GLOB proc;
    class TRAV,MATCH branch;
    class PASS pass;
    class TRIP,KERNEL,PURGE,LOG trip;
```

---

### 5.2 Canonical Scope Shield Interface

```typescript
export interface ScopeShieldEvaluator {
  /** Asserts whether all modified paths fall strictly within approved_scope */
  assertConfinement(
    modifiedPaths: string[], 
    approvedScope: string[], 
    repoRoot: string
  ): ScopeShieldVerdict;

  /** Validates that a proposed Goal Contract amendment has valid scope declarations */
  validateScopeDeclaration(proposedScope: string[]): boolean;
}

export interface ScopeShieldVerdict {
  passed: boolean;
  violations: Array<{
    path: string;
    violation_type: 'TRAVERSAL_ATTEMPT' | 'UNAPPROVED_PATH' | 'DISALLOWED_SYMLINK';
    matched_rule: string | null;
  }>;
  tripped_event_payload: {
    event_type: 'SCOPE_SHIELD_TRIPPED';
    offending_worker_id: string;
    champion_commit_sha: string;
  } | null;
}
```

---

## 6. Failure Cases

1. **Directory Traversal via Relative Paths**: Worker submits edits to `docs/../../etc/shadow`.  
   *Defense*: `filepath.Clean` resolves canonical relative path; detection of leading `../` triggers `TRAVERSAL_ATTEMPT`.
2. **Stealth Modification via Symlink**: Worker creates symlink `docs/link.md -> src/auth/keys.ts`.  
   *Defense*: Scope Shield resolves real target path via `realpath`; target fails `approved_scope` match.
3. **Implicit Dependency Injection**: Worker modifies `package.json` to add an external dependency when only `src/modules/*.ts` is in scope.  
   *Defense*: `package.json` is not matched by `approved_scope`; slice rejected, workspace purged.

---

## 7. Security Considerations

1. **AST Path Sanitization**: Path comparisons must normalize case sensitivity on Windows/macOS (`Path.toLowerCase()`) to prevent case-mismatch bypasses on case-insensitive filesystems.
2. **Pre-Commit Hook Integration**: The Scope Shield is integrated into the local Git pre-commit hook, ensuring that even if a supervisor is compromised, raw git commits cannot be finalized without Scope Shield certification.

---

## 8. Examples

### Example: Scope Shield Intercepting Unauthorized File Touch
```json
{
  "slice_id": "slice-14",
  "worker_id": "worker-builder-alpha-01",
  "modified_files": [
    "docs/architecture/02_KERNEL_ARCHITECTURE.md",
    "package.json"
  ],
  "approved_scope": [
    "docs/architecture/*.md"
  ],
  "evaluation": {
    "passed": false,
    "violations": [
      {
        "path": "package.json",
        "violation_type": "UNAPPROVED_PATH",
        "matched_rule": null
      }
    ]
  },
  "action": "TRIPWIRE_ACTIVATED",
  "kernel_transition": "EXECUTING -> REPLAN_REQUIRED",
  "workspace_reset_sha": "c4b9e1"
}
```

---

## 9. Architecture Corrections

1. **Formalized Scope Shield as Standalone Protocol**: Elevated scope containment from an informal rule into an authoritative Level 1 protocol with strict AST normalization.
2. **Direct State Transition Invariant**: Codified the automatic `EXECUTING -> REPLAN_REQUIRED` transition upon scope breach.

---

## 10. References to Related RFCs

- [**RFC-0000: The Forge Constitution**](RFC-0000_FORGE_CONSTITUTION.md) — Law I (Founder Sovereignty) and Violation V-03 (Silent Scope Expansion).
- [**RFC-0001: The Kernel Contract**](RFC-0001_KERNEL_CONTRACT.md) — Implements `assertScopeConfinement` and `REPLAN_REQUIRED` state transition.
- [**RFC-0006: Project Ledger Protocol**](RFC-0006_PROJECT_LEDGER_PROTOCOL.md) — Records `ROLLBACK` and `TASK_FAILED` upon tripwire activation.
- [**RFC-0007: Forge Ratchet Protocol**](RFC-0007_FORGE_RATCHET_PROTOCOL.md) — Scope confinement as Phase 1 Entry Condition.
