# 05 — Agent Manager Architecture (`forge-agents`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Package**: `forge-agents` (Heterogeneous Agent Pool Engine)

---

## 1. Purpose

The Agent Manager acts as the HR department and process supervisor for heterogeneous AI models. It abstracts disparate model providers (ChatGPT, Gemini, Grok, Meta Llama, Claude, Codex) into standardized worker interfaces, enforcing role specialization, managing credential pools, and ensuring the human user never has to micromanage agent assignments.

---

## 2. Responsibilities

1. **Provider Normalization**: Convert provider-specific schemas (OpenAI, Anthropic, Google Vertex/GenAI, xAI, Ollama) into a unified Forge Worker API.
2. **Role Specialization Enforcement**: Assign models to roles where their unique capabilities excel.
3. **Context Wiping for Critics**: Enforce clean-slate context isolation when instantiating blind critics.
4. **Credential & Session Pool Management**: Manage rotating API keys and token quotas.
5. **No Human Micro-Management**: Shield the human from agent routing, model prompts, and internal handoffs.

---

## 3. Normalized Worker Interface

```typescript
interface ForgeWorker {
  id: string;
  provider: "openai" | "google" | "xai" | "meta" | "anthropic" | "local";
  model: string;
  role: "architect" | "researcher" | "builder" | "critic" | "auditor";
  
  execute(task: TaskSlice, context: L1MemorySnapshot): Promise<WorkerResult>;
  ping(): Promise<HealthStatus>;
  getContextLimit(): number;
}
```

---

## 4. Role Assignment Matrix

```
┌─────────────────┬───────────────────────────────────────────────────────┐
│ MODEL FAMILY    │ ASSIGNED SPECIALIZATION & RATIONALE                   │
├─────────────────┼───────────────────────────────────────────────────────┤
│ ChatGPT (OpenAI)│ Lead Architect & Wayfinder: Structural planning       │
│ Gemini (Google) │ Research & Knowledge: 1M+ multimodal context ingest   │
│ Grok (xAI)      │ Specialist Builder: High-throughput CLI & code repair │
│ Meta (Llama)    │ Blind Critic: Open-weights adversarial inspection     │
│ Claude / Codex  │ Emergency Fallback: Deep reasoning & precision diffs   │
└─────────────────┴───────────────────────────────────────────────────────┘
```

The user never chooses who implements a slice. The **Scheduler** and **Capability Router** determine the assignment based on task type, active rate limits, and benchmark scores.

---

## 5. Inputs & Outputs
- **Inputs**: Task definitions from `06_SCHEDULER.md`, clean context packets from `04_MEMORY_OS.md`.
- **Outputs**: Standardized `WorkerResult` containing code diffs, command execution logs, and verification assertions.

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Provider API deprecation or breaking schema change*.  
  **Defense**: Normalized adapter interface decouples core logic from vendor SDK changes.
- *Failure Mode 2: Model sycophancy or self-justification*.  
  **Defense**: Blind critic workers are spawned in isolated sub-sessions with zero access to the builder's thought trace.

---

## 7. Interactions with Other Modules
- Managed by `06_SCHEDULER.md` to dispatch tasks.
- Monitored by `03_RUNTIME_ARCHITECTURE.md` for process heartbeats and leases.
- Governed by capability heuristics in `07_CAPABILITY_ROUTER.md`.

---

## 8. Future Extensions
- Support for on-device quantized local models (Apple Silicon MLX, llama.cpp) for offline execution.
- Dynamic fine-tuning adapter injection per task domain.
