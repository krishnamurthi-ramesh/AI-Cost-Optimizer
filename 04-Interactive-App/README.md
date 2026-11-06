# 📈 AI Inference Cost & Dynamic Routing Simulator

> An institutional-grade financial simulator and dynamic model router demonstrating how cascading semantic tiering reduces AI inference costs by up to 68% while preserving 99%+ of frontier model quality.

🔗 **Live Production App:** [https://ai-cost-optimizer-iota.vercel.app](https://ai-cost-optimizer-iota.vercel.app)

---

## 🎯 The Problem
Software companies face a **gross margin collapse** when deploying generative AI features:
- Defaulting to monolithic frontier models (GPT-4o, Claude 3.5 Sonnet) costs upwards of **$137,000/month per 100k DAU**.
- 70%–80% of real-world user queries are low-complexity requests where specialized Small Language Models (Gemini 2.5 Flash, Llama 3.3 70B) match frontier accuracy at 1/15th the token cost.

---

## 📊 Live Financial Model Overview

| Strategy | Monthly Inference Bill (100k DAU) | P95 Latency | Quality Retention | Gross Margin ($20/mo SaaS) |
|---|---|---|---|---|
| **Strategy A: Frontier Monolith** | $137,116 / mo | 2.80s | 94.2% | 54.3% |
| **Strategy B: SLM Monolith** | $18,696 / mo | 0.65s | 88.5% | 93.8% |
| **Strategy C: Cascading Router** | **$45,553 / mo** | **1.20s** | **93.4% (99.2% of frontier)** | **84.8%** |

### Key Business Results
- **-$91,564 / month** in net cash savings ($1.1M annualized run-rate savings).
- **57.1% lower response latency** through intelligent SLM routing.
- **+30.5% gross margin expansion** on subscriber revenue.

---

## 🛠️ Features

1. **Interactive Sensitivity Controls:** Sliders for DAU (10k to 500k), Queries/Day, Prompt Cache Hit %, and Escalation %.
2. **Strategy Comparison Matrix:** Side-by-side economics (cost per query, cost per active user, P95 latency, gross margin).
3. **Live Semantic Classifier Playground:** Test real queries to see automated complexity scoring, tier assignment, and cost attribution in real time.
4. **Zero-Overhead Static Architecture:** Built with clean, accessible vanilla Web Standards and hosted on Vercel's Edge Network.

---

## 📄 License
MIT License. Part of the AI Product Management Portfolio Series.

