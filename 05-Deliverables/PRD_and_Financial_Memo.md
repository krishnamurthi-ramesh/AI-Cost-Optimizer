# Product Requirements Document (PRD) & Financial Architecture Memo
## Semantic Cascading Router: AI Inference Unit Economics & Dynamic Tiering Platform

- **Document Version:** 2.1 (Production Standard)
- **Author:** R. Krishnamurthi (AI Product Manager)
- **Target Audience:** Chief Technology Officer, VP of Engineering, Head of AI Platform, VP Finance
- **Live Deployment:** [https://ai-cost-optimizer-iota.vercel.app](https://ai-cost-optimizer-iota.vercel.app)
- **Status:** Approved for Phased Production Rollout

---

## 1. Executive Summary & Problem Space

### 1.1 The Context & The "Gross Margin Trap"
Between 2024 and 2026, software companies rushed generative AI copilots and autonomous multi-agent systems into production. While top-line user adoption and interaction volume expanded rapidly, SaaS unit economics deteriorated sharply. 

In traditional cloud software, hosting and compute represent amortized, fixed infrastructure with gross margins of **75% to 85%**. In contrast, generative AI features incur direct, variable token consumption costs that sit directly within **Cost of Goods Sold (COGS)**. 

### 1.2 The Root Causes
1. **Monolithic Over-Provisioning:** Engineering teams routinely route 100% of incoming prompts to flagship frontier models (e.g., Anthropic Claude 3.5 Sonnet, OpenAI GPT-4o) out of fear of edge-case failure.
2. **Margin Deterioration:** At 100,000 Daily Active Users (DAU) generating 8 queries/day, a monolithic frontier architecture incurs **~$137,116/month in raw LLM token fees**, crushing gross margins on a standard $20/month subscription from 85% down to **54.3%**.
3. **Asymmetric Entropy Distribution:** Real-world usage shows that **70% to 80% of daily user requests are low-entropy tasks** (formatting, schema extraction, simple entity lookup, short summarization) where small language models (Gemini 2.5 Flash, Llama 3.3 70B) achieve statistical parity with frontier models at **1/10th the token cost**.
4. **Latency Penalties:** Frontier reasoning models impose a P95 latency penalty of 2.5s–4.0s, reducing user interaction frequency and retention.

### 1.3 The Solution: The Semantic Cascading Router
An intelligent, drop-in inference proxy that:
1. Evaluates incoming query complexity in real-time (<80ms) via a lightweight classifier.
2. Automatically routes low-to-medium complexity tasks (70%–80% of volume) to high-speed SLMs (Gemini 2.5 Flash, Llama 3.3 70B).
3. Reserves frontier models (Claude 3.5 Sonnet, GPT-4o) for high-entropy multi-step reasoning.
4. Enforces strict prompt caching discounts (saving 50%–90% on input prefixes).
5. Provides an automated confidence-based fallback safety net to guarantee zero user-visible degradation.

---

## 2. User Personas & Jobs-To-Be-Done (JTBD)

| Persona | Role & Context | Primary Pain Point | Job-To-Be-Done (JTBD) |
|---|---|---|---|
| **Marcus Vance** | VP of Engineering | Cloud infrastructure budget is bleeding cash; board is demanding 30% reduction in LLM inference spend. | *"Help me slash monthly model spend without requiring my engineering team to rewrite hundreds of API integrations or sacrifice user uptime."* |
| **Elena Rostova** | Head of AI Platform / Staff ML Eng | Under pressure to migrate to smaller models, but terrified of regression, hallucination, or customer escalation. | *"Guarantee that low-tier model errors automatically fall back to frontier models so my team never causes an enterprise client outage."* |
| **David Chen** | VP Finance / FP&A | Gross margins dropped from 82% to 54%; cannot accurately forecast unit economics as DAU grows. | *"Provide a predictable model where inference cost per user scales linearly and gross margins remain comfortably above 80%."* |

---

## 3. Product Architecture & System Mechanics

```
                         [ Incoming Client Prompt ]
                                     │
                                     ▼
                     ┌──────────────────────────────┐
                     │   OpenAI / Anthropic Proxy   │
                     │  (Zero Code Changes for Devs)│
                     └───────────────┬──────────────┘
                                     │
                                     ▼
                     ┌──────────────────────────────┐
                     │   Semantic Complexity        │
                     │      Classifier (SLM)        │
                     │  Latency: <80ms | Cost: $0   │
                     └───────────────┬──────────────┘
                                     │
             ┌───────────────────────┴───────────────────────┐
             ▼ Complexity < 0.65                             ▼ Complexity >= 0.65
   ┌───────────────────────┐                       ┌───────────────────────┐
   │    SLM Tier (78%)     │                       │  Frontier Tier (22%)  │
   │  • Gemini 2.5 Flash   │                       │  • Claude 3.5 Sonnet  │
   │  • Llama 3.3 70B      │                       │  • OpenAI GPT-4o      │
   │  Latency: 0.65s       │                       │  Latency: 2.80s       │
   │  Cost: $0.00078/q     │                       │  Cost: $0.00571/q     │
   └───────────┬───────────┘                       └───────────┬───────────┘
               │                                               │
               ▼                                               │
   ┌────────────────────────┐                                  │
   │ Confidence & Schema    │                                  │
   │      Verification      │                                  │
   └───────────┬────────────┘                                  │
         Pass  │       │ Fail (Entropy > 2.5 or Parse Error)   │
               │       └───────────────────┐                   │
               │                           ▼                   │
               │               ┌───────────────────────┐       │
               │               │ Automated Transparent │       │
               │               │   Frontier Escalation │───────┘
               │               └───────────────────────┘
               ▼
     [ Verified Response Delivered to User ]
     Mean Latency: 1.20s | Blended Cost: $0.00190/q
```

---

## 4. Functional Requirements

### FR-1: Real-Time Semantic Complexity Classification
* **Description:** The system must classify incoming prompts within **<80ms** across a normalized complexity spectrum ($0.0 \le c \le 1.0$).
* **Model:** Default to Gemini Flash Lite / fine-tuned embedding model.
* **Threshold:** Queries scoring $<0.65$ route to SLM tier; $\ge 0.65$ route to Frontier tier.
* **Acceptance Criteria:** Classification accuracy $\ge 96.5\%$ on benchmark dataset of 5,000 historical enterprise prompts.

### FR-2: Multi-Model Gateway & Dynamic Routing
* **Description:** Support dynamic dispatching across multiple upstream providers:
  - Anthropic (`claude-3-5-sonnet`)
  - OpenAI (`gpt-4o`)
  - Google (`gemini-2.5-flash`, `gemini-flash-lite`)
  - Groq (`llama-3.3-70b-versatile`)
* **Acceptance Criteria:** OpenAI-compatible API spec (`/v1/chat/completions`). Any application can change its `baseURL` and operate with zero code changes.

### FR-3: Automated Confidence-Based Fallback Cascade
* **Description:** When an SLM handles a query, the response is inspected before streaming to the client:
  1. **Schema Check:** Strict JSON validation against user-supplied JSON schemas.
  2. **Entropy Check:** Output token perplexity must be $<2.5$.
  3. **Hedging Detection:** Immediate rejection if phrases indicating failure appear ("I cannot answer", "I am uncertain").
* **Acceptance Criteria:** If check fails, the query instantly re-routes to Claude 3.5 Sonnet / GPT-4o. Total fallback rate targeted at $<4.5\%$.

### FR-4: Strict Prompt Prefix Caching Optimization
* **Description:** The router normalizes system prompts and tool declarations into ordered prefix blocks to trigger upstream KV-cache discounts (saving up to 90% on input tokens).
* **Acceptance Criteria:** System maintains a cache hit rate $\ge 35\%$ across all active production sessions.

### FR-5: Shadow Routing & Evaluation Mode
* **Description:** Capability to run the router in "Shadow Mode" where 100% of user responses are still served by Frontier, but SLM predictions run asynchronously in the background.
* **Acceptance Criteria:** Automated daily report measuring exact semantic similarity, cosine distance, and latency differences before live production cutover.

### FR-6: Real-Time Telemetry & Financial Attribution
* **Description:** Emits Prometheus/Datadog metrics for every query:
  - `query_cost_dollars`
  - `input_tokens_cached` vs `uncached`
  - `assigned_tier` (`slm` | `frontier` | `fallback`)
  - `e2e_latency_ms`

---

## 5. Non-Functional Requirements & Service Level Objectives (SLOs)

| Metric | Target SLO | Measurement Mechanism |
|---|---|---|
| **Classifier Overhead Latency** | P95 $\le 85\text{ms}$ | Measured from ingress to model dispatch |
| **End-to-End Availability** | $\ge 99.99\%$ Uptime | Multi-region redundant deployment |
| **Fallback Cascade Latency** | P95 $\le 3.4\text{s}$ (when fallback triggered) | Total turnaround time for dual-hop retry |
| **Data Privacy & Compliance** | Zero PII persistence | Transient in-memory routing; SOC2 and GDPR compliant |
| **Throughput Capacity** | 2,500 requests/sec | Horizontal autoscaling Kubernetes deployment |

---

## 6. Financial Architecture & ROI Comparison

### 6.1 Baseline Economics (100k DAU, 8 Queries/User/Day, 30 Days)

| Dimension | Strategy A: Monolithic Frontier | Strategy B: Monolithic SLM | Strategy C: Cascading Router | Strategic Delta (C vs. A) |
|---|---|---|---|---|
| **Monthly Spend** | **$137,116 / mo** | $18,696 / mo | **$45,553 / mo** | **-$91,564 / mo (-66.8%)** |
| **Annualized Run-Rate** | $1,645,392 / yr | $224,352 / yr | **$546,636 / yr** | **-$1,098,756 / yr in Cash Savings** |
| **Blended Cost per Query** | $0.00571 | $0.00078 | **$0.00190** | **-66.7%** |
| **P95 Latency** | 2.80s | 0.65s | **1.20s** | **57.1% Faster** |
| **Quality Score** | 94.2 / 100 | 88.5 / 100 | **93.4 / 100** | **99.2% Quality Retention** |
| **Gross Margin ($20/mo SaaS)** | 54.3% | 93.8% | **84.8%** | **+30.5% Gross Margin Expansion** |

### 6.2 Unit Economic Walkdown per Query
* **Monolithic Frontier Query:**
  - Input (650 tokens, 35% cached): $(227.5 \times \$0.775 + 422.5 \times \$2.75) / 1\text{M} = \$0.00134$
  - Output (350 tokens): $(350 \times \$12.50) / 1\text{M} = \$0.00438$
  - Total: **$0.00571 / query**
* **Cascading Router Query (Blended):**
  - Classifier Overhead: **$0.00003**
  - 78% SLM Resolution: $0.78 \times \$0.00078 = \$0.00061$
  - 22% Frontier Escalation: $0.22 \times \$0.00571 = \$0.00126$
  - Total: **$0.00190 / query** (Net reduction: **66.7%**)

---

## 7. Competitive Differentiation Matrix

| Capability | AWS Bedrock Routing | Martian Router | Unify.ai | In-House Cascading Router (This Project) |
|---|---|---|---|---|
| **Cross-Cloud Vendor Flexibility** | ❌ AWS Locked | ✅ Yes | ✅ Yes | ✅ Complete (Anthropic, OpenAI, Google, Groq) |
| **Deterministic Data Privacy** | ✅ High | ⚠️ Third-Party Gateway | ⚠️ Third-Party Gateway | ✅ 100% In-VPC / In-House Data Control |
| **Cost per Routing Call** | Bundled in AWS fee | Commission on token delta | Monthly SaaS tier | **$0.00003/query (Pure raw tokens)** |
| **Custom Schema Fallback Gate** | ❌ No | ❌ General fallback only | ❌ Latency/Error based | ✅ Domain-specific schema & entropy gate |
| **Interactive Executive Modeling** | ❌ CloudWatch only | ❌ Dashboard only | ❌ No P&L simulator | ✅ Full P&L interactive simulator deployed |

---

## 8. Phased Rollout & Migration Plan

```
Phase 1: Shadow Mode (Days 1–14)
├── Route 100% of user traffic to Frontier (Zero user risk)
├── Run Semantic Classifier & SLM asynchronously in background
└── Measure: Discordance rate, classification precision, theoretical savings

Phase 2: Canary Deployment (Days 15–28)
├── Route 10% of production traffic through Cascading Router
├── Enable automated fallback to Frontier on schema failure
└── Measure: CSAT, ticket escalation, latency delta, P&L delta

Phase 3: Full Production Cutover (Days 29–45)
├── Ramp to 50% → 100% traffic allocation
├── Activate KV-cache enforcement across all standardized templates
└── Final Review: Confirm 66%+ inference spend reduction with VP Finance
```

---

## 9. Comprehensive Risk Matrix & Mitigations

| Risk Event | Severity | Probability | Automated Mitigation |
|---|---|---|---|
| **Routing Misclassification (False Negative)**: Complex prompt misrouted to SLM. | High | Medium (3%–5%) | Automated schema validator and entropy check detects degraded reasoning and instantly re-routes to Frontier. |
| **Upstream Provider Rate Limit / Outage** (e.g., Anthropic API outage). | High | Low | Router features cross-provider failover: if Claude 3.5 Sonnet fails, seamlessly fails over to OpenAI GPT-4o in <200ms. |
| **Classifier Latency Spike** (>150ms). | Medium | Low | Timeout circuit breaker: if classifier fails to respond in 80ms, query bypasses classifier directly to primary model. |
| **User Frustration on Fallback Latency** (dual hop takes 3.4s). | Low | Low (<2%) | Progressive streaming: UI indicates "Performing comprehensive verification..." during rare fallback executions. |
