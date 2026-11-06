# Routing Architecture Specification
## Semantic Cascading Router — Design Decisions & Rationale

- **Document Type:** Architecture Decision Record (ADR)
- **Last Updated:** November 2026
- **Author:** R. Krishnamurthi

---

## 1. Problem Statement

When a user sends a query to an AI-powered product, the system must decide which model to use. Most engineering teams default to sending everything to the most expensive frontier model. This is wasteful because the majority of queries don't need frontier-level reasoning.

**The core design question:** How do we automatically route each query to the cheapest model that can answer it well, without the user noticing any quality difference?

---

## 2. Routing Approaches Evaluated

| Approach | How It Works | Pros | Cons | Decision |
|---|---|---|---|---|
| **Rule-based keyword matching** | If query contains "compare", "analyze", "debug" → frontier; else → SLM | Zero latency overhead, simple to implement | Brittle; misses nuanced complexity; requires constant rule maintenance | ❌ Rejected |
| **Embedding similarity classifier** | Embed the query, compare against cluster centroids of "simple" vs "complex" historical queries | Fast (~20ms), trainable on real data | Requires labeled training data; cold-start problem for new query types | ⚠️ Considered |
| **LLM-as-classifier (small model)** | Send the query to a tiny model (Flash Lite) with a classification prompt: "Rate complexity 0-1" | More nuanced understanding; handles novel query types; no training data needed | Adds 60-100ms latency; consumes tokens; recursive dependency (using an LLM to classify for LLMs) | ✅ Selected |
| **User-selected tier** | Let the user pick "Fast" vs "Deep" mode | Zero misclassification; user has full control | Poor UX; users don't want to think about infrastructure; most will default to one mode | ❌ Rejected |
| **Try-small-then-escalate** | Always start with SLM; if output confidence is low, retry with frontier | Guarantees cheapest path first | 2x latency on escalated queries (worst case: full SLM attempt + full frontier attempt) | ⚠️ Considered as fallback only |

### Final Architecture: Hybrid Approach

**Primary:** LLM-as-classifier using Gemini Flash Lite (~80ms, $0.10/M tokens)
**Secondary:** Confidence-based fallback — if the SLM's output fails schema validation or returns high-entropy tokens, auto-escalate to frontier

This hybrid gives us:
- Nuanced classification without training data
- A safety net for misclassified queries
- Total router overhead: ~$0.05/M input tokens + 80ms latency

---

## 3. Complexity Threshold: Why 0.65

The classifier assigns each query a complexity score from 0.0 to 1.0. Queries scoring ≥ 0.65 are routed to the frontier tier.

### How 0.65 Was Determined

This threshold was calibrated by analyzing where SLM quality diverges from frontier quality across task types:

| Task Category | SLM Score (Gemini Flash) | Frontier Score (Sonnet) | Delta | Complexity Range |
|---|---|---|---|---|
| Simple retrieval / lookup | 95.1 | 95.8 | -0.7 | 0.1 – 0.3 |
| Text transformation / formatting | 94.2 | 94.5 | -0.3 | 0.2 – 0.4 |
| Summarization | 92.8 | 94.1 | -1.3 | 0.3 – 0.5 |
| Multi-document synthesis | 88.5 | 93.2 | -4.7 | 0.5 – 0.7 |
| Multi-step reasoning / planning | 82.1 | 93.8 | -11.7 | 0.7 – 0.9 |
| Ambiguous logic / edge cases | 76.4 | 91.5 | -15.1 | 0.8 – 1.0 |

**The inflection point is around 0.60-0.70** — this is where the quality delta jumps from <2 points to >4 points. Setting the threshold at 0.65 means:

- Queries where SLM matches frontier (delta < 2pts) → routed to SLM ✅
- Queries where SLM would noticeably degrade (delta > 4pts) → routed to frontier ✅
- The "gray zone" (0.60–0.70) contains some queries that could go either way. 0.65 errs slightly toward quality preservation.

### Threshold Tuning in Production

In a production deployment, this threshold would be continuously calibrated using:
1. **Shadow scoring:** Run both tiers on a sample of queries; measure actual quality delta
2. **User feedback signals:** Thumbs-up/down, regeneration rate, session abandonment
3. **Cost budget constraints:** If monthly spend exceeds budget, raise threshold to route more to SLM

---

## 4. Confidence-Based Fallback Logic

Even with good classification, some queries will be misrouted. The fallback mechanism catches these:

```
                    ┌──────────────────────┐
                    │  Query arrives       │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  Classifier: Score   │
                    │  < 0.65 → SLM        │
                    │  ≥ 0.65 → Frontier   │
                    └──────────┬───────────┘
                               │
                     Score < 0.65 path
                               │
                    ┌──────────▼───────────┐
                    │  SLM generates       │
                    │  response            │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  Confidence Check:   │
                    │  1. Schema valid?    │
                    │  2. Entropy < 2.5?   │
                    │  3. No "I don't      │
                    │     know" hedging?   │
                    └──────────┬───────────┘
                          ╱          ╲
                   Pass ╱              ╲ Fail
                       ╱                ╲
              ┌───────▼──────┐  ┌───────▼────────┐
              │ Return SLM   │  │ Discard SLM    │
              │ response     │  │ output;        │
              │              │  │ Re-route to    │
              │              │  │ Frontier       │
              └──────────────┘  └────────────────┘
```

### Fallback Triggers

1. **Schema validation failure:** If the expected output format (JSON, structured response) is malformed
2. **High output entropy:** Token probability distribution is flat (model is "guessing"), measured by perplexity > 2.5 on output tokens
3. **Explicit hedging:** Output contains phrases like "I'm not sure", "I don't have enough information" at high frequency — indicates the query exceeded the SLM's capability

### Estimated Fallback Rate

In the current model, approximately 3-5% of SLM-routed queries trigger a fallback. This is accounted for in the cost model as additional frontier spend within the router strategy.

---

## 5. Why Not a Simple Embedding Classifier?

Embedding-based classifiers (encode query → cosine similarity to complexity clusters) were evaluated but deprioritized for the initial version because:

1. **Cold-start problem:** Requires a labeled dataset of queries tagged with complexity scores. For a new product, this data doesn't exist on day one.
2. **Domain sensitivity:** An embedding model trained on general web queries may not transfer well to a specific product's query distribution.
3. **Maintenance burden:** Embeddings need periodic retraining as query patterns evolve.

The LLM-classifier approach works "zero-shot" with a well-crafted prompt, making it suitable for products that don't yet have query history.

**Recommended migration path:** Start with LLM-classifier → collect 10,000+ classified queries → train a fine-tuned embedding classifier → switch to embeddings for lower latency and cost.

---

## 6. Alternative Architecture: Speculative Execution

One architecture considered but not implemented was **speculative execution** (inspired by speculative decoding):

1. Send query to SLM immediately
2. Simultaneously send to frontier
3. If SLM response is "good enough" (passes confidence check), cancel the frontier request and return SLM response
4. If SLM fails, return the frontier response (which was already computing in parallel)

**Pros:** Lowest user-perceived latency (frontier latency is never additive)
**Cons:** Doubles inference cost for every query (both models always run). Only viable if you're optimizing for latency, not cost. Rejected since this project is fundamentally about cost optimization.

