# 07 — Capability Router Architecture (`forge-router`)

> **Status**: APPROVED ARCHITECTURE DRAFT | **Target**: Forge OS v0.2+  
> **Package**: `forge-router` (Model Intelligence & Provider Routing)

---

## 1. Purpose

The Capability Router provides dynamic, telemetry-driven intelligence to match tasks with the optimal foundation model provider. It monitors live token quotas, API latencies, context window limits, tool capabilities, and pricing to deliver resilient execution without manual configuration.

---

## 2. Responsibilities

1. **Multi-Dimensional Provider Scoring**: Score available models across 8 dimensions (Context size, Latency, Cost, Tool support, Multimodal, Reasoning, Coding, Reliability).
2. **Dynamic Fallback Cascades**: Reroute requests automatically when a provider encounters HTTP 429 (rate limit), 503 (service outage), or token exhaustion.
3. **Budget Optimization**: Direct routine tasks to cost-effective models while reserving frontier reasoning models for architectural planning and complex debugging.
4. **Offline & Edge Routing**: Fall back to local models (e.g. Ollama / Llama 3) when internet connectivity is severed.

---

## 3. Capability Scoring Matrix

| Provider / Model | Context Window | Coding Strength | Reasoning Depth | Cost per 1M In/Out | Primary Dispatch Target |
|---|---|---|---|---|---|
| **ChatGPT (o1 / o3)** | 200k tokens | 9.5 / 10 | 9.8 / 10 | High | Wayfinder Planning & Architecture |
| **Gemini (2.0 Pro/Flash)**| 1M–2M tokens | 8.8 / 10 | 9.0 / 10 | Low–Medium | Large Codebase Recon & Media Sync |
| **Grok (xAI Grok-2/3)** | 128k tokens | 9.3 / 10 | 8.9 / 10 | Medium | High-Speed Builder & Bug Fixing |
| **Meta Llama (3.3 70B/405B)**| 128k tokens | 9.0 / 10 | 9.1 / 10 | Low / Self-hosted | Clean-Slate Blind Critic & Security Audit |
| **Claude (3.5 Sonnet)** | 200k tokens | 9.6 / 10 | 9.4 / 10 | Medium | Precision Refactoring & Final Scrutiny |

---

## 4. Formal Routing Rules

```
RULE 1: IF task == "ARCHITECTURE_PLANNING" 
        THEN assign PRIMARY: ChatGPT (o1/o3) | FALLBACK: Claude 3.5 Sonnet

RULE 2: IF task == "DEEP_REPO_INSPECTION" OR input_size > 100k_tokens
        THEN assign PRIMARY: Gemini 2.0 Pro | FALLBACK: Gemini 2.0 Flash

RULE 3: IF task == "BUILDER_CODE_SLICE"
        THEN assign PRIMARY: Grok-2 / Claude 3.5 | FALLBACK: GPT-4o

RULE 4: IF task == "BLIND_CRITIC_REVIEW"
        THEN assign PRIMARY: Meta Llama 3.3 (Clean Slate) | FALLBACK: Claude 3.5 (Wiped Context)
        INVARIANT: Critic model MUST NOT be the same instance as Builder

RULE 5: IF provider.error == HTTP_429 OR provider.health == OFFLINE
        THEN trigger IMMEDIATE_FALLBACK_CASCADE and record in .forge/ledger.jsonl
```

---

## 5. Inputs & Outputs
- **Inputs**: Task complexity profile, input token volume, live provider health telemetry, budget constraints.
- **Outputs**: Resolved `ModelEndpoint` configuration (Base URL, Auth Header, Model ID, Max Output Tokens).

---

## 6. Failure Modes & Defenses
- *Failure Mode 1: Global multi-provider outage*.  
  **Defense**: Route to local on-premise model (Ollama / llama.cpp) in degraded mode; set state to `BLOCKED` if task requires unavailable capability.
- *Failure Mode 2: Flapping provider connection*.  
  **Defense**: Circuit breaker trips provider into cooldown for 120 seconds after 3 consecutive connection drops.

---

## 7. Interactions with Other Modules
- Directly consulted by `06_SCHEDULER.md`.
- Telemetry updated by `03_RUNTIME_ARCHITECTURE.md`.
- Credentials managed via `12_SECURITY_MODEL.md`.

---

## 8. Future Extensions
- Automated prompt rewriting adapted to vendor-specific tokenizers and fine-tuned system instructions.
- Cost-performance Pareto curve dashboard.
