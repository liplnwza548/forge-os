# RFC-0004: Provider Capability Matrix & Routing Protocol

```yaml
RFC: 0004
Title: Provider Capability Matrix & Routing Protocol
Status: FOUNDER_FREEZE_RELEASE_CANDIDATE
Author: AGY (Implementation Agent)
Founder: LiplnwZa
Chief Architect: ChatGPT GPT-5
Target: Forge OS Core Governance (v0.2+)
Created: 2026-09-08
Supersedes: docs/architecture/07_CAPABILITY_ROUTER.md
Authority: LEVEL 1 (Subordinate to RFC-0000, Peer to RFC-0001)
```

---

## 1. Purpose

This specification establishes the **Provider Capability Matrix and Routing Protocol** for Forge OS.

In compliance with **Article V of the Forge Constitution (RFC-0000)**, Forge OS mandates absolute **Provider Neutrality**. It eliminates arbitrary commercial vendor rankings and brittle benchmark scores, replacing them with a standardized ontology of **Semantic Capability Descriptors**. 

The system treats frontier cloud APIs, local open-weight engines, and specialized reasoning models as interchangeable, untrusted cognitive backends. Scheduling decisions are resolved through formal capability matching, dynamic health telemetry, and strict **Builder/Critic Orthogonality**.

---

## 2. Scope

1. **In-Scope**:
   - The Semantic Capability Descriptor schema (`level`, `confidence`, `determinism`, `tool_access`, `context_capacity`, `multimodal_support`).
   - Abstract capability dimensions: reasoning, coding, multimodal, planning, verification, memory, tools.
   - Provider Lineage tracking and the Builder/Critic Orthogonality Law.
   - Algorithmic Routing Decision Protocol and dynamic Capability Negotiation.
   - Real-time Health Telemetry Interface and Provider Trust Scoring.
2. **Out-of-Scope**:
   - Vendor-specific SDK integration code (implemented in Runtime Provider Adapters).
   - Commercial billing account management and payment processing.

---

## 3. Definitions

All terms conform to [`GLOSSARY.md`](GLOSSARY.md). Key terms:
- **Semantic Descriptor**: A qualitative, verified classification of capability (`basic`, `competent`, `proficient`, `expert`).
- **Provider Lineage**: The architectural ancestor family (e.g., `lineage:llama-family`, `lineage:gpt-family`) preventing shared blind-spot collusion.
- **Builder/Critic Orthogonality**: The constitutional rule that a Builder and Critic on a single slice must belong to distinct lineage families.
- **Provider Trust Score**: A dynamic metric $[0.0, 1.0]$ based on real-time availability, latency stability, and schema conformance.

---

## 4. Invariants

1. **Brand Neutrality Invariant**: No scheduling decision or code path may reference commercial brand names or vendor labels. Dispatch is governed exclusively by capability profiles and lineage tokens.
2. **Builder/Critic Orthogonality Law**: A task slice whose Builder is of lineage $A$ **MUST NEVER** be assigned to a Critic of lineage $A$. If only one lineage exists in the environment, the Critic must run under an adversarial zero-shot configuration with an explicit ledger warning.
3. **Minimum Capability Assertion**: A task requiring capability level `expert` in `coding` cannot be dispatched to a worker whose profile indicates `competent` or lower.

---

## 5. Interfaces & Schemas

### 5.1 Semantic Capability Descriptor Schema

Each registered capability is defined using a standardized semantic tuple:

```yaml
# Example: High-Assurance Reasoning Descriptor
reasoning:
  level: expert               # basic | competent | proficient | expert
  determinism: high           # low | medium | high
  confidence: verified        # speculative | empirically_tested | verified
  tool_access: true           # native structured tool invocation support
  context_capacity: 1048576   # usable token window size before degradation
  multimodal_support:
    vision: true
    audio: false
```

#### Standard Levels Defined:
- `basic`: Capable of minor formatting, summarization, and simple boilerplate.
- `competent`: Capable of writing standard functions, single-file scripts, and localized bug fixes.
- `proficient`: Capable of multi-file refactoring, complex type systems, and integration test generation.
- `expert`: Capable of formal architectural synthesis, invariant validation, and adversarial vulnerability detection.

---

### 5.2 Canonical Provider Manifest Schema (`providers/<provider_id>.json`)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "ForgeProviderManifest",
  "type": "object",
  "required": [
    "provider_id",
    "lineage",
    "endpoint_type",
    "capabilities",
    "cost_class",
    "availability_class"
  ],
  "properties": {
    "provider_id": { "type": "string", "pattern": "^provider-[a-z0-9-]+$" },
    "lineage": { 
      "type": "string", 
      "description": "Model family lineage token for orthogonality enforcement (e.g., lineage:alpha, lineage:beta)" 
    },
    "endpoint_type": { 
      "type": "string", 
      "enum": ["OPENAI_COMPATIBLE", "ANTHROPIC_NATIVE", "GOOGLE_GENAI", "CUSTOM_RPC"] 
    },
    "capabilities": {
      "type": "object",
      "required": ["reasoning", "coding", "planning", "verification", "memory", "tools"],
      "properties": {
        "reasoning": { "$ref": "#/definitions/CapabilityDescriptor" },
        "coding": { "$ref": "#/definitions/CapabilityDescriptor" },
        "planning": { "$ref": "#/definitions/CapabilityDescriptor" },
        "verification": { "$ref": "#/definitions/CapabilityDescriptor" },
        "memory": { "$ref": "#/definitions/CapabilityDescriptor" },
        "tools": { "$ref": "#/definitions/CapabilityDescriptor" }
      }
    },
    "cost_class": { "type": "string", "enum": ["free", "economic", "standard", "premium"] },
    "availability_class": { "type": "string", "enum": ["best_effort", "tier_2", "tier_1"] }
  },
  "definitions": {
    "CapabilityDescriptor": {
      "type": "object",
      "required": ["level", "determinism", "confidence", "tool_access", "context_capacity"],
      "properties": {
        "level": { "type": "string", "enum": ["basic", "competent", "proficient", "expert"] },
        "determinism": { "type": "string", "enum": ["low", "medium", "high"] },
        "confidence": { "type": "string", "enum": ["speculative", "empirically_tested", "verified"] },
        "tool_access": { "type": "boolean" },
        "context_capacity": { "type": "integer", "minimum": 2048 },
        "multimodal_support": {
          "type": "object",
          "properties": {
            "vision": { "type": "boolean" },
            "audio": { "type": "boolean" }
          }
        }
      }
    }
  }
}
```

---

### 5.3 Health Telemetry Interface & Provider Trust Score

The Runtime continuously computes the **Provider Trust Score** via a 5-minute Exponentially Weighted Moving Average (EWMA):

```typescript
export interface ProviderHealthTelemetry {
  provider_id: string;
  sliding_window_seconds: 300;
  total_requests: number;
  rate_limit_errors_429: number;
  server_errors_5xx: number;
  schema_validation_failures: number;
  avg_time_to_first_token_ms: number;
  tokens_per_second: number;
  trust_score: number; // [0.0, 1.0]
}
```

$$\text{ErrorPenalty} = 3 \times \text{Rate}(429) + 5 \times \text{Rate}(5\text{xx}) + 10 \times \text{Rate}(\text{SchemaFailures})$$
$$\text{TrustScore}(P) = \max\left(0.0, 1.0 - \text{ErrorPenalty}\right) \times \min\left(1.0, \frac{\text{TargetLatency}}{\text{ActualLatency}}\right)$$

---

### 5.4 Routing Decision Flow

```mermaid
flowchart TD
    TASK["Task Slice Ready\n(Requirements: Coding >= proficient, Verification >= expert)"] --> REQ["Capability Requirement Vector"]
    REQ --> FILTER["Filter Providers by Minimum Required Levels"]
    
    FILTER --> ORTHO{"Is Role == CRITIC?"}
    ORTHO -->|YES| EXCLUDE["Exclude Providers Matching Builder Lineage"]
    ORTHO -->|NO| RANK["Proceed to Scoring"]
    EXCLUDE --> RANK

    RANK --> SCORE["Compute Composite Score:\nScore = CapabilityLevelScore * TrustScore / CostWeight"]
    SCORE --> SORT["Rank Candidate Providers"]

    SORT --> DISPATCH{"Primary Provider Healthy?\n(TrustScore > 0.6 & Rate Limit Clear)"}
    DISPATCH -->|YES| ASSIGN["Grant Task Lease to Primary"]
    DISPATCH -->|NO (Degraded / 429)| CASCADE["Cascade to Next Highest Ranked Candidate"]

    classDef proc fill:#2563eb,stroke:#1d4ed8,stroke-width:2px,color:#fff;
    classDef branch fill:#d97706,stroke:#b45309,stroke-width:2px,color:#fff;
    classDef success fill:#059669,stroke:#047857,stroke-width:2px,color:#fff;
    classDef fail fill:#b91c1c,stroke:#7f1d1d,stroke-width:2px,color:#fff;

    class TASK,REQ,FILTER,EXCLUDE,SCORE,SORT proc;
    class ORTHO,DISPATCH branch;
    class ASSIGN success;
    class CASCADE fail;
```

---

## 6. Failure Cases

1. **Provider Outage During Task Execution**: Primary provider encounters persistent 500 errors mid-slice.  
   *Defense*: Heartbeat expires; supervisor evicts lease; Scheduler cascades to the next lineage provider with matching capabilities.
2. **Collusion Attempt (Same Lineage Reseller)**: Two different API endpoints host fine-tunes of the same base model.  
   *Defense*: Manifest records root `lineage` token; orthogonality checker catches the shared ancestry and blocks Critic assignment.
3. **No Eligible Orthogonal Provider Available**: Single-model environment configured.  
   *Defense*: Explicit Capability Negotiation fallback mode: Critic spawned with strict zero-shot adversarial system prompt, emitting a non-blocking `COMPLIANCE_WARNING` to the Ledger.

---

## 7. Security Considerations

1. **Credential Isolation**: Provider manifests never contain raw API keys or passwords. They declare credential aliases, and the Runtime Security Vault injects tokens directly into isolated child process environments at spawn time.
2. **Schema Conformance Filtering**: Malformed tool responses or JSON injection attempts are counted as schema validation failures, directly penalizing the provider's Trust Score.

---

## 8. Examples

### Example: Capability Negotiation for Verification Task
```yaml
# Task Requirement: Critical Cryptographic Verification
required_capabilities:
  verification:
    min_level: expert
    min_determinism: high
  coding:
    min_level: proficient
lineage_exclusion: "lineage:alpha-models"

# Scheduler Evaluator Matches:
# Candidate 1: provider-beta-01 (Lineage: beta, Verif: expert, Determinism: high, Trust: 0.98) -> MATCHED (Primary)
# Candidate 2: provider-alpha-02 (Lineage: alpha) -> EXCLUDED (Orthogonality Violation)
# Candidate 3: provider-gamma-01 (Lineage: gamma, Verif: competent) -> REJECTED (Level Insufficient)
```

---

## 9. Architecture Corrections

1. **Full CapabilityDescriptor Schema Alignment**: Reconciled the JSON schema in Section 5.2 with Section 5.1 by adding `tool_access`, `context_capacity`, and `multimodal_support` to the descriptor definition.
2. **Replacement of Numeric Benchmark Scores**: Removed arbitrary 0-100 scalar scores. Replaced with discrete semantic descriptors (`level`, `determinism`, `confidence`) preventing micro-benchmark drift.
3. **Builder/Critic Orthogonality Law**: Codified lineage-based isolation as a strict mathematical filter rather than an optional scheduling heuristic.

---

## 10. References to Related RFCs

- [**RFC-0000: The Forge Constitution**](RFC-0000_FORGE_CONSTITUTION.md) — Article V (Provider Neutrality) and Law II (Independent Verification).
- [**RFC-0003: Worker Lease Protocol**](RFC-0003_WORKER_LEASE_PROTOCOL.md) — Grants execution leases based on routing decisions.
- [**RFC-0006: Project Ledger Protocol**](RFC-0006_PROJECT_LEDGER_PROTOCOL.md) — Records `TASK_ASSIGNED` with provider and lineage metadata.
- [**RFC-0010: Trust and Provenance Protocol**](RFC-0010_TRUST_AND_PROVENANCE_PROTOCOL.md) — Implements provider trust scoring and attestation.
