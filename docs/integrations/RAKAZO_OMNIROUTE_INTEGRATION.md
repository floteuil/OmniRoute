# RAKAZO ⇄ OMNIROUTE INTEGRATION SPECIFICATION

> **IMPORTANT SCOPE NOTICE:**  
> This document applies **exclusively to the Rakazo integration** (`rakazo-*` namespace) and must **not** be treated as a default or global policy for other OmniRoute consumers (e.g. n8n, Flowise, or direct API clients).

---

## 1. Overview & Architecture

Rakazo connects to OmniRoute as a multi-provider sovereign inference gateway for its **Free Track** execution runtime.

```
┌─────────────────────────────────────────────────────────────┐
│                      RAKAZO PLATFORM                        │
│  • Sovereign Agents (Pi Runtime, Graphile Workers)          │
│  • Prompt Compiler & Deterministic 4-Block Cache (A,B,C,D)  │
│  • Modular MCP Tooling & Anti-Loop Circuit Breakers         │
└──────────────────────────────┬──────────────────────────────┘
                               │ HTTP POST /v1/chat/completions
                               │ Header: Authorization: Bearer <RAKAZO_API_KEY>
                               │ Header: x-session-id: <fnv1a_hash>
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                      OMNIROUTE GATEWAY                      │
│  • Priority Cascades scoped to Rakazo (`combo/rakazo-*`)    │
│  • Strict Zero-Cost ($0.00) Enforcement (Fail-Closed)       │
│  • Native MCP Tool Calling Dispatcher                       │
│  • Upstream Session Affinity (KV Cache reuse)               │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Interface Contract

* **Base URL:** `https://omniroute.workspacegroupefloteuil.eu/v1`
* **Chat Endpoint:** `POST /v1/chat/completions`
* **Authorization Header:** `Authorization: Bearer <OMNIROUTE_RAKAZO_KEY>`
* **Session Sticky Header:** `x-session-id: <session_id>`

### Scoped Combo Routes (`model` parameter)

| Rakazo Intent | Resolved Combo | Priority Cascade (Verified Free Tier) | Capabilities |
| :--- | :--- | :--- | :--- |
| `coding` | `combo/rakazo-coding` | 1. `mistral/codestral-latest`<br>2. `groq/openai/gpt-oss-120b`<br>3. `groq/qwen/qwen3.6-27b`<br>4. `gemini/gemini-3.1-flash-lite`<br>5. `cohere/command-r-plus-08-2024` | Native Tool Calling (MCP), Contexte $\ge 32$k. |
| `reasoning` | `combo/rakazo-reasoning` | 1. `groq/qwen/qwen3.6-27b`<br>2. `groq/openai/gpt-oss-120b`<br>3. `gemini/gemini-3.1-flash-lite`<br>4. `cohere/command-a-reasoning-08-2025` | Preservation of thinking tokens (`<think>`). |
| `fast` | `combo/rakazo-fast` | 1. `groq/openai/gpt-oss-20b`<br>2. `mistral/mistral-small-latest`<br>3. `cohere/command-r7b-12-2024`<br>4. `gemini/gemini-3.1-flash-lite` | Ultra-low TTFT (~300–600 ms). |
| `writing` | `combo/rakazo-writing` | 1. `mistral/mistral-small-latest`<br>2. `cohere/command-r-plus-08-2024`<br>3. `gemini/gemini-3.1-flash-lite`<br>4. `groq/openai/gpt-oss-120b` | High French editorial quality. |
| `analysis` | `combo/rakazo-analysis` | 1. `gemini/gemini-3.1-flash-lite`<br>2. `groq/openai/gpt-oss-120b`<br>3. `groq/qwen/qwen3.6-27b`<br>4. `cohere/command-r-plus-08-2024` | 1M Context window, structured JSON. |

---

## 3. Live Telemetry Headers

OmniRoute attaches diagnostic response headers on each response:

```http
x-omniroute-provider: groq | mistral | gemini | cohere
x-omniroute-model: <resolved_model_id>
x-omniroute-decision: strategy=priority; provider=<prov>; latency_ms=<ms>
x-omniroute-response-cost: 0.0000000000
x-omniroute-session-id: ext:<session_id>
x-omniroute-version: 3.8.51
x-omniroute-latency-ms: <duration_ms>
```

---

## 4. Onboarding & Deprecation Lifecycle

1. **New Candidate Model Onboarding:**
   `Review Provider ToS` ➔ `Verify $0 Cost Tier` ➔ `Certify Native Tool Calling (40-Tool Benchmark)` ➔ `Manual Add to Scoped Combo`.
2. **Model Deprecation / Degradation:**
   If a model introduces billing, breaks tool calling, or revokes its free tier, it is immediately removed from the `rakazo-*` combo pool without impacting other client combos.
