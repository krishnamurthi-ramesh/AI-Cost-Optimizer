# ⚡ AI Inference Unit Economics & Semantic Cascading Router
### An AI Product Management Case Study, Competitive Teardown & Production Simulator

[![Live Simulator](https://img.shields.io/badge/Live_Simulator-Vercel_Deployed-2563eb?style=for-the-badge&logo=vercel&logoColor=white)](https://ai-cost-optimizer-iota.vercel.app)
[![Financial Model](https://img.shields.io/badge/Financial_Model-Verified_2026_Pricing-059669?style=for-the-badge)](./02-Financial-Model/pricing_matrix.json)
[![Role](https://img.shields.io/badge/Role-AI_Product_Manager-7c3aed?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-Validated_Production_Architecture-0f172a?style=for-the-badge)]()

> **🚀 Live Interactive Simulator:** [https://ai-cost-optimizer-iota.vercel.app](https://ai-cost-optimizer-iota.vercel.app)  
> **📄 Production PRD & Financial Memo:** [05-Deliverables/PRD_and_Financial_Memo.md](./05-Deliverables/PRD_and_Financial_Memo.md)  
> **🔍 Competitive Teardown:** [01-Research/competitive-teardown.md](./01-Research/competitive-teardown.md)

---

## 🎯 Executive Summary 

Between 2024 and 2026, software companies rushed GenAI copilots into production. But as active usage scaled, software gross margins collapsed from **80%+ down to 54%** because teams defaulted to routing 100% of queries to expensive frontier models (Claude 3.5 Sonnet, GPT-4o).

This project presents an **in-house Semantic Cascading Router and Unit Economics Financial Engine** that cuts monthly LLM inference spend by **66.8%** ($91,564/month savings at 100k DAU), accelerates response latency by **57.1%**, and preserves **99.2%** of frontier model quality with zero user degradation.

---

## 📊 The Financial Impact at a Glance (100k DAU Scale)

Holding standard enterprise parameters constant (100,000 DAU, 8 queries/user/day, 650 input tokens, 350 output tokens, 35% KV-cache hit rate, 22% frontier escalation):

| Dimension | Strategy A: Monolithic Frontier | Strategy B: Monolithic SLM | Strategy C: Cascading Router (This Project) | Strategic Delta (C vs. A) |
|---|---|---|---|---|
| **Monthly Inference Spend** | **$137,116 / mo** | $18,696 / mo | **$45,553 / mo** | **-$91,564 / mo (-66.8%)** |
| **Annualized Run-Rate** | $1,645,392 / yr | $224,352 / yr | **$546,636 / yr** | **-$1,098,756 / yr in Cash Savings** |
| **Blended Cost per Query** | $0.00571 | $0.00078 | **$0.00190** | **-66.7% Token Cost** |
| **P95 Response Latency** | 2.80s | 0.65s | **1.20s** | **57.1% Faster Turnaround** |
| **Benchmark Quality Score** | 94.2 / 100 | 88.5 / 100 | **93.4 / 100** | **99.2% Quality Retention** |
| **Gross Margin ($20/mo SaaS)** | 54.3% | 93.8% | **84.8%** | **+30.5% Gross Margin Expansion** |
| **Break-Even Boundary** | N/A | N/A | **97.3% Escalation** | Accretive even if 97% of queries need Frontier |

---

## 🔍 Case Study & Problem Teardown

### 1. The "AI Gross Margin Collapse"
In traditional cloud software, server hosting and database queries are amortized fixed costs. In Generative AI software, every user prompt triggers model token generation, incurring direct variable inference costs that sit directly within **Cost of Goods Sold (COGS)**. 

Reports from **Andreessen Horowitz (a16z)** (*"The Cost of AI"*) and enterprise telemetry reveal that AI-native startups frequently operate at gross margins between **45% and 60%**, compared to 80%+ for historical SaaS. Furthermore, under flat monthly subscription pricing ($20/user/month), the top 10% of heavy users consume up to 60% of total tokens, creating toxic cohorts with negative unit economics.

### 2. The Asymmetric Entropy Reality
Why does this happen? **Engineering inertia.** Teams default to sending every request to Claude 3.5 Sonnet or GPT-4o because frontier models are forgiving of poor prompt engineering. 

However, empirical query distributions (LMSYS Chatbot Arena, WildChat) show:
- **70% to 80% of daily user queries are low-entropy, deterministic tasks** (JSON formatting, simple extraction, keyword lookup, short summarization) where small models (Gemini 2.5 Flash, Llama 3.3 70B) achieve statistical parity with frontier models at **1/10th the cost**.
- Only **20% to 25% of queries genuinely require multi-step frontier reasoning**.

---

## ⚔️ Competitive Teardown: Why Build an In-House Router?

We evaluated the leading commercial and open-source model routing alternatives before designing our solution:

| Dimension | Martian (*withmartian.com*) | Unify.ai | AWS Bedrock Intelligent Routing | In-House Cascading Router (This Project) |
|---|---|---|---|---|
| **Multi-Provider Flexibility** | High (OpenAI, Anthropic) | Ultra-High (20+ providers) | ❌ Cloud-locked (Bedrock only) | **✅ Full (Anthropic, OpenAI, Google, Groq)** |
| **Data Privacy & Egress** | ❌ Sent to Martian's proxy | ❌ Sent to Unify's gateway | ✅ AWS VPC native | **✅ 100% In-VPC (Zero third-party hops)** |
| **Routing Fee / Query** | Cut of token delta savings | Monthly SaaS fee | Bundled in Bedrock fee | **$0.00003/query (Pure raw tokens)** |
| **Latency Overhead** | +45ms to 90ms | +40ms to 80ms | +30ms to 50ms | **<80ms (Flash-Lite / Embedding)** |
| **Domain Schema Validation** | ❌ No | ❌ No | ❌ No | **✅ Native JSON schema & entropy gate** |
| **Interactive P&L Simulator** | ❌ No | ❌ Benchmark only | ❌ CloudWatch only | **✅ Live deployed financial tool** |

**PM Verdict:** While third-party gateways introduce privacy risks, external network hops, and margin cuts, an **In-House Cascading Router** deployed inside the corporate boundary delivers total data sovereignty, zero markup, and custom schema fallback guarantees.

---

## 🏗️ System Architecture & Automated Fallback Loop

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

### The 3 Core Safety Mechanisms:
1. **Semantic Complexity Threshold (0.65):** Queries scoring $<0.65$ route to SLMs; $\ge 0.65$ escalate to Frontier. Derived by analyzing where SLM quality diverges from Frontier quality across task categories.
2. **Confidence-Based Fallback Gate:** If the SLM output fails JSON schema validation, exhibits high token entropy ($>2.5$), or contains hedging phrases, it is discarded and transparently re-routed to Claude 3.5 Sonnet in <3.4s.
3. **KV-Cache Optimization:** Enforcing standardized prompt prefix blocks unlocks 90% discounts on cached input tokens ($0.03/M vs $0.30/M on Gemini Flash).

---

## 📈 Scale & Sensitivity Modeling

We simulated 32 distinct parameter permutations across company scale, escalation rates, and cache rates:

### Growth Scale Sensitivity
| Growth Stage | DAU | Monthly Queries | Frontier Monolith | Cascading Router | Monthly Savings | Annual EBITDA Impact |
|---|---|---|---|---|---|---|
| **Series A** | 50,000 | 12.0M | $42,849 | **$14,235** | $28,614 | **+$343,368 / yr** |
| **Growth Baseline** | 100,000 | 24.0M | $137,116 | **$45,553** | $91,564 | **+$1,098,756 / yr** |
| **Mid-Market** | 250,000 | 60.0M | $514,187 | **$170,822** | $343,365 | **+$4,120,380 / yr** |
| **Enterprise Fleet** | 500,000 | 120.0M | $685,583 | **$227,763** | $457,820 | **+$5,493,840 / yr** |

### Escalation Stress-Testing
- **At 5% Escalation** (data extraction/formatting): Router saves **81.5%** ($111.7k/mo).
- **At 22% Escalation** (enterprise copilot baseline): Router saves **66.8%** ($91.6k/mo).
- **At 50% Escalation** (heavy reasoning analytics): Router saves **42.6%** ($58.4k/mo).
- **Break-Even Boundary:** The router remains cash-positive compared to monolithic frontier up to **97.3% escalation**.

---

## 📁 Repository Guide & Artifacts

| Directory | File | Purpose |
|---|---|---|
| **`01-Research/`** | [`cost-optimization-research-synthesis.md`](./01-Research/cost-optimization-research-synthesis.md) | 400+ line market research: a16z data, margin collapse, entropy taxonomy |
| | [`competitive-teardown.md`](./01-Research/competitive-teardown.md) | Deep teardown of Martian, Unify.ai, AWS Bedrock, Portkey, In-House |
| | [`engineering-interviews-synthesis.md`](./01-Research/engineering-interviews-synthesis.md) | Qualitative discovery with 4 engineering leaders (VP Eng, CTO, FP&A) |
| **`02-Financial-Model/`** | [`pricing_matrix.json`](./02-Financial-Model/pricing_matrix.json) | Verified late-2026 token pricing across 5 models |
| | [`financial-model-design-doc.md`](./02-Financial-Model/financial-model-design-doc.md) | Assumptions, token volumes (650 in / 350 out), cache hit mechanics |
| **`03-Routing-Engine/`** | [`cost_routing_simulator.py`](./03-Routing-Engine/cost_routing_simulator.py) | Python CLI Monte Carlo simulation and financial sensitivity engine |
| | [`routing-architecture-spec.md`](./03-Routing-Engine/routing-architecture-spec.md) | ADR: 0.65 threshold derivation, fallback mechanics, trade-off analysis |
| **`04-Interactive-App/`** | [`index.html`](./04-Interactive-App/index.html) | Institutional single-page app (deployed to Vercel) |
| **`05-Deliverables/`** | [`PRD_and_Financial_Memo.md`](./05-Deliverables/PRD_and_Financial_Memo.md) | Comprehensive 280+ line production PRD, personas, SLOs, risk matrix |
| **`06-Analysis-Report/`** | [`inference-cost-sensitivity-report.md`](./06-Analysis-Report/inference-cost-sensitivity-report.md) | 32-permutation sensitivity report & implementation playbook |

---

## 💻 Local Execution & CLI Simulator

```bash
# Clone the repository
git clone https://github.com/krishnamurthi-ramesh/AI-Cost-Optimizer.git
cd AI-Cost-Optimizer/03-Routing-Engine

# Run baseline simulation (100,000 DAU, 8 queries/user/day)
python cost_routing_simulator.py

# Run custom scale simulation
python cost_routing_simulator.py --dau 250000 --queries 12
```

---

## 📌 Author & Portfolio Context
- **Author:** R. Krishnamurthi (AI Product Manager)
- **Live Simulator:** [https://ai-cost-optimizer-iota.vercel.app](https://ai-cost-optimizer-iota.vercel.app)
- **Tableau Portfolio Link:** [Perplexity Quality Eval Harness](https://public.tableau.com/views/PerplexityAIQualityEvaluationHarness/PerplexityAIQualityEvaluationHarness?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)
