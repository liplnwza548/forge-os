# Canonical Glossary of Forge OS

```yaml
Document: GLOSSARY.md
Status: FOUNDER_FREEZE_RELEASE_CANDIDATE
Version: 1.1.0
Authority: Derived from RFC-0000 (Forge Constitution)
Last Updated: 2026-09-08
```

---

## 1. Purpose & Precedence

This Glossary establishes the single, authoritative vocabulary for Forge OS. All RFCs (RFC-0000 through RFC-0010), architecture specifications, schemas, interfaces, and operational manuals must conform to these definitions. In any case of lexical ambiguity or semantic drift, the definitions herein—anchored by the [Forge Constitution](RFC-0000_FORGE_CONSTITUTION.md)—strictly govern.

---

## 2. Core System Governance

### Constitution
The supreme legal and architectural charter of Forge OS ([RFC-0000](RFC-0000_FORGE_CONSTITUTION.md)). It defines the non-negotiable operating laws, human sovereignty, governance tiers, and amendment rules. It outranks all other specifications, code, and agent consensus.

### Non-Derogable Constitutional Laws
The foundational set of constitutional principles that can never be suspended, bypassed, or weakened under any circumstances, including emergency operations or constitutional amendments:
1. Founder Sovereignty (Absolute human primacy)
2. Independent Verification (Zero self-certification)
3. Ratchet Invariance (No state advancement without physical evidence)
4. Total Worker Isolation (Zero ambient privileges)
5. Emergency Break-Glass (Instant human termination)

### Founder
The human principal (`LiplnwZa`) who holds ultimate sovereignty over system objectives, budget, capital, permissions, and life-cycle progression.

### Founder Sovereignty
The absolute constitutional law that no autonomous agent, model swarm, or algorithmic heuristic may override human intent, mutate an approved Goal Contract, or advance past approval gates without explicit, authenticated human authorization.

### Founder Freeze
A formal governance state in which an RFC, specification, or codebase is locked against all further modifications, awaiting the Founder's explicit ratification before code generation or deployment.

### Constitutional Authority Hierarchy
The strict six-tier precedence structure:
- **Level 0**: Constitution ([RFC-0000](RFC-0000_FORGE_CONSTITUTION.md))
- **Level 1**: RFC Contracts ([RFC-0001](RFC-0001_KERNEL_CONTRACT.md) to [RFC-0010](RFC-0010_TRUST_AND_PROVENANCE_PROTOCOL.md))
- **Level 2**: Kernel (`packages/kernel`)
- **Level 3**: Runtime (`packages/runtime`, `scheduler`, `agent-manager`, `storage`)
- **Level 4**: Skills (`packages/skills/*`)
- **Level 5**: Projects (Working repositories managed by Forge OS)
*Rule*: Lower authority may extend higher authority, but may never contradict or weaken it.

---

## 3. Kernel & Execution Architecture

### Kernel
The mathematically pure, referentially transparent policy engine ([RFC-0001](RFC-0001_KERNEL_CONTRACT.md)). The Kernel owns state legality, goal contract validation, ratchet rules, and verification criteria. It contains zero execution mechanics, zero network sockets, zero filesystem I/O, and zero vendor SDKs.

### Runtime
The user-space operational platform that implements execution mechanics: spawning workers, managing timeouts, hosting local network listeners, streaming logs, and coordinating disk I/O. The Runtime imports the Kernel; the Kernel **never** imports the Runtime.

### Project
A distinct software repository managed under the Forge OS lifecycle containing an on-disk `.forge/` ledger, local codebase, test suite, and configuration.

### State Machine
The formal 13-state deterministic lifecycle (`UNINITIALIZED` $\to$ `GRILLING` $\to$ `GRILL_CONFIRMED` $\to$ `PLANNING` $\to$ `PLAN_GENERATED` $\to$ `AWAITING_APPROVAL` $\to$ `EXECUTING` $\to$ `VERIFYING` $\to$ `SCRUTINIZING` $\to$ `DONE`, with exception states `REPLAN_REQUIRED` and `HALTED`). Governed exclusively by the Kernel.

### Decision Frontier
The boundary between settled architectural choices and remaining open questions during the `GRILL` stage. Questioning terminates immediately once this frontier is exhausted.

---

## 4. Contracts, Hashes & Scope Shield

### Goal Contract
The binding, machine-readable covenant (`.forge/goal_contract.json`) that defines the project's objective, desired outcome, success criteria, constraints, and non-goals. Governed by the canonical 11-field schema ([RFC-0001](RFC-0001_KERNEL_CONTRACT.md)).

### Immutable Founder Goal Law
The constitutional invariant that a Goal Contract, once approved, cannot change unless the Founder explicitly authorizes an amendment, the amendment is recorded as a `GOAL_AMENDED` event in the Ledger, and a new Goal Hash is generated.

### Goal Hash (`goal_hash`)
The SHA-256 cryptographic digest uniquely identifying the immutable intent, success definitions, constraints, and scope of a Goal Contract:
$$\text{goal\_hash} = \operatorname{SHA256}(\text{CanonicalJSON}(\{\text{goal\_id}, \text{founder\_intent}, \text{founder\_id}, \text{success\_definition}, \text{constraints}\}))$$

### Approval Hash (`approval_hash`)
The cryptographic signature or HMAC token generated upon Founder authorization at Stage 4 (`APPROVE`). It binds the `goal_hash`, `approved_at` timestamp, and `founder_id`, authorizing the transition into `EXECUTING`:
$$\text{approval\_hash} = \operatorname{HMAC-SHA256}(\text{FounderKey}, \text{goal\_hash} \mathbin{\Vert} \text{approved\_at} \mathbin{\Vert} \text{founder\_id})$$

### Scope Shield
The defensive runtime boundary enforcement engine ([RFC-0009](RFC-0009_SCOPE_SHIELD_PROTOCOL.md)) that confines all worker file modifications strictly to `approved_scope`. Any unapproved file touch trips the Scope Shield and transitions the system from `EXECUTING` directly to `REPLAN_REQUIRED`.

### Scope Hash (`scope_hash`)
The cryptographic digest sealing the approved target files and whitelist directories (`approved_scope`) permitted to be modified during execution.

---

## 5. Execution, Scheduling & Workers

### Worker
An isolated operating system process instantiated by the Runtime to perform a bounded computational task (such as code editing, test running, or verification).

### Builder
A worker role responsible for implementing code, documentation, or configuration modifications for a specific Task Slice ($\le 100$ lines).

### Critic
An isolated worker role running in a separate process with a distinct model family and clean context, tasked with adversarial review, compilation verification, and test execution of a Builder's output.

### Builder/Critic Orthogonality Law
The absolute rule that a Builder and Critic assigned to the same task slice must never share the same provider family, process memory, or conversational context ([RFC-0004](RFC-0004_PROVIDER_CAPABILITY_MATRIX.md)).

### Task Slice
An atomic, indivisible unit of engineering work restricted to no more than 100 contiguous modified lines of code ($\le 100$ lines changed), complete with its own dedicated verification contract.

### Worker Lease
A time-bounded, single-writer reservation granted by the Scheduler to a worker for a specific task slice, recorded in `.forge/workers/leases/<slice_id>.lock` ([RFC-0003](RFC-0003_WORKER_LEASE_PROTOCOL.md)).

### Lease Policy
The structured configuration object defined by the Kernel and supplied by the Runtime governing lease lifecycles:
```yaml
lease_policy:
  heartbeat_interval: duration
  lease_timeout: duration
  max_runtime: duration
  retry_budget: integer
  checkpoint_interval: duration
```

### Compare-And-Swap (CAS) Lease Renewal
The concurrency protocol where a worker must verify that the active `.lock` file holds its expected `nonce` before extending lease timestamps, preventing stale zombie workers from overwriting reclaimed locks.

---

## 6. Verification, Ratchet & Ledger

### Evidence
Physical, reproducible artifacts generated during execution—such as test exit codes, AST parse trees, compiler error streams, benchmark logs, and cryptographic diff hashes. Natural language claims have zero evidential value.

### Verification
The deterministic validation of evidence against an approved verification contract, performed by an independent Critic and certified by the Kernel ([RFC-0008](RFC-0008_VERIFICATION_CONTRACT_PROTOCOL.md)).

### Verification Contract
A machine-readable specification detailing commands, constraints, timeouts, required levels ($V_0$–$V_5$), and critic rubrics necessary to prove an artifact's correctness before advancing across the Ratchet ([RFC-0008](RFC-0008_VERIFICATION_CONTRACT_PROTOCOL.md)).

### Verification Receipt
An immutable, signed data structure recording execution outputs, duration, exit code, stdout/stderr SHA-256 hashes, and critic endorsements of a verification pass ([RFC-0008](RFC-0008_VERIFICATION_CONTRACT_PROTOCOL.md)).

### Verification Levels (V0 to V5)
The strictness tiers applied during evaluation (prefixed with `V_` to prevent collision with Memory Layers $L_0$–$L_4$):
- `V0_SYNTAX`: Valid AST parse, zero syntax errors.
- `V1_COMPILE`: Clean compilation / typecheck exit code ($0$).
- `V2_UNIT`: Isolated unit test suite passing 100%.
- `V3_INTEGRATION`: Cross-component integration tests passing.
- `V4_INVARIANT`: Formal invariant / fuzz testing pass.
- `V5_OUTSIDER`: Clean-slate outsider scrutinize audit.

### Flaky Test Quarantine
The formal isolation state where tests with non-deterministic outcomes across identical commits are isolated to `.forge/quarantine/` and prevented from deadlocking build pipelines ([RFC-0008](RFC-0008_VERIFICATION_CONTRACT_PROTOCOL.md)).

### Ratchet
The unidirectional execution protocol guaranteeing that system progress moves strictly forward through verified evidence ([RFC-0007](RFC-0007_FORGE_RATCHET_PROTOCOL.md)). State never advances without verified evidence, and unverified changes are immediately rolled back.

### Ratchet Levels
The five progressive tiers of the Forge Ratchet:
1. `Plan Ratchet`: Freezes requirements into a sealed Goal Contract.
2. `Build Ratchet`: Advances code state by committing verified slices ($\le 100$ lines).
3. `Verify Ratchet`: Certifies test suites and independent critic audits against $V_0$–$V_5$.
4. `Knowledge Ratchet`: Distills verified facts and schemas into long-term graphs.
5. `Experience Ratchet`: Encodes post-mortems and reusable patterns into global memory.

### Ratchet Gate
The two-phase validation gate:
1. **Entry Condition**: Checks whether physical proofs, critic orthogonality, line count ($\le 100$), and scope confinement are valid.
2. **Advancement Transaction**: Executes atomic commit, memory promotion, and checkpoint logging in sequential lockstep.

### Champion Commit
The most recent Git commit that has successfully passed all verification gates and received critic certification. It serves as the immutable rollback anchor.

### Rollback
The instantaneous reversion of the working directory to the last Champion Commit following a failed verification, without deleting the failure history from the Ledger.

### Ledger & Causal Event DAG
The append-only, immutable event log stored at `.forge/ledger.jsonl` ([RFC-0006](RFC-0006_PROJECT_LEDGER_PROTOCOL.md)). Formed as a causal Directed Acyclic Graph (DAG) using explicit `parent_event_id` pointers, tracking 17 canonical event types. Overwriting, truncating, or modifying past ledger entries is strictly prohibited.

---

## 7. Memory Graph & Knowledge Architecture

### Memory Graph Fabric
The multi-layered, typed graph database protocol replacing flat filesystem memory ([RFC-0002](RFC-0002_MEMORY_GRAPH_PROTOCOL.md)).

### Memory Layers
- **$E_0$ Evidence**: Ephemeral physical artifacts (raw test outputs, linter streams, compiler exit codes).
- **$L_0$ Session**: Ephemeral worker context, active prompt scratchpads, turn memory. Wiped on task completion.
- **$L_1$ Project**: Active project state, local decision graph, task queues, and working file graph.
- **$L_2$ Skill**: Procedural memory, tool calling patterns, validated AST transformation recipes.
- **$L_3$ Knowledge**: Declarative truths, ingested API specifications, system architecture documentation.
- **$L_4$ Experience**: Long-term episodic memory, cross-project post-mortems, failure signatures, and architectural records.

### Provenance Chain
The cryptographic audit record embedded in every Memory Node ([RFC-0010](RFC-0010_TRUST_AND_PROVENANCE_PROTOCOL.md)), tracking author identity, prompt hash, parent dependencies, and critic attestation.

### Memory Promotion
The formal process by which verified insights advance from lower layers to higher layers ($E_0 \to L_0 \to L_1 \to L_2/L_3 \to L_4$), requiring physical proof and cryptographic hash verification.

### Memory Demotion
The process of downgrading or invalidating a memory node when a subsequent post-mortem or verification failure proves that a previously held assumption or pattern is defective.

### Checkpoint
An atomic, point-in-time serialization of the working codebase, Memory Graph, and Ledger state, bound cryptographically to a Git commit hash.

### Memory Trust Score
A calculated metric $[0.0, 1.0]$ based on historical verification frequency, critic independence, and reproduction success, determining how prominently a memory node is weighted during planning.

---

## 8. Providers & Capabilities

### Provider
An external or local cognitive engine service (e.g., cloud API or self-hosted open-weight model) providing completion, embedding, or reasoning capabilities ([RFC-0004](RFC-0004_PROVIDER_CAPABILITY_MATRIX.md)).

### Provider Neutrality
The constitutional law that Forge OS must treat all AI providers without brand affinity, commercial bias, or hardcoded favoritism, evaluating them purely through quantitative semantic capability descriptors.

### Capability Descriptor
A structured evaluation of a provider's competence in a specific dimension, consisting of `level`, `confidence`, `determinism`, `tool_access`, `context_capacity`, and `multimodal_support`.

### Provider Lineage
The model family ancestry token (e.g., `lineage:llama-family`, `lineage:claude-family`, `lineage:gpt-family`) used to enforce Builder/Critic orthogonality regardless of host provider or reseller.

### Provider Trust Score
A dynamic, exponentially weighted moving average (EWMA) reflecting a provider's real-time error rate, latency stability, schema conformance, and availability.

---

## 9. Skills & Extensions

### Skill
A hermetically isolated, versioned package containing procedural heuristics, reference documentation, and declarative verification hooks for a specific domain ([RFC-0005](RFC-0005_SKILL_PACKAGE_SPEC.md)).

### Skill Sandbox
The strict 4D containment boundary enforcing zero ambient privileges across Memory, Storage, Provider, and Network domains.
