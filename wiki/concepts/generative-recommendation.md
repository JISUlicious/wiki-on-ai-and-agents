---
title: Generative Recommendation
type: concept
created: 2026-10-29
updated: 2026-10-29
sources:
  - genrec-llm-recommendation-ranker-netflix-li-2026.md
  - kuaishou-explorer-llm-rec-challenge-cao-2026.md
  - vela-user-behavior-as-supervision-xiao-2026.md
  - coral-llm-native-harness-production-recommender-azhar-2026.md
  - lifecycle-of-llm-as-a-judge-recommendation-explanations-kong-2026.md
status: draft
importance: medium
tags:
  - 2026
---

# Generative Recommendation

**Generative recommendation** replaces (or wraps) the classic recommender pipeline of engineered features and discriminative rankers with a language model that reads a user's history and produces the recommendation. The wiki's sources show two main ways of doing it, plus LLMs used *around* a conventional recommender. All production results come from very large platforms and are small relative effects.

## Two output designs

| | Semantic-ID decoding | Catalog scoring head |
|---|---|---|
| Example | Kuaishou OneRec / OneReason ([[kuaishou-explorer-llm-rec-challenge-cao-2026]]) | Netflix GenRec ([[genrec-llm-recommendation-ranker-netflix-li-2026]]) |
| Item representation | Semantic IDs: a domain marker plus three RQ-KMeans codes (8,192 codewords per level) | Learned item embeddings |
| Inference | Autoregressive next-item generation, optionally after [[chain-of-thought]] | One prefill-only forward pass on [[vllm]], softmax over the catalog |
| Out-of-catalog items | Possible if decoding wanders | Impossible by construction |
| Training signal | SFT, RL, DPO (challenge teams) | Ranking cross-entropy plus LM loss, reweighted by reward models in place of RL |

GenRec explicitly contrasts itself with the SID approach. The Kuaishou report starts from the premise that CoT reasoning "did not reliably improve recommendation", and its central claim that reasoning helps is shown only through a challenge metric built to reward think-mode diversity (full-SID hit rates in one team's curves are about 2–4%).

## Results so far

- **GenRec (Netflix):** about +1.6% relative offline MRR over production with ~40× fewer Phase-2 labelled examples. Online A/B: short-term engagement +0.115%, core long-term metric +0.006% (p = 0.025), batch surfaces only. [[context-engineering]] cut user context from ~5,000 to ~1,700 tokens with negligible loss, and the authors frame the shift as "feature engineering to context engineering".
- **Kuaishou challenge:** the champion masked the target-SID loss on self-sampled reasoning traces, because supervising the answer collapsed think/no-think diversity; self-sampled traces beat a larger teacher's.
- **Vela:** uses **user behaviour as supervision**, splitting long histories into visible past and hidden future (72K–120K-token segments). VelaBench 51.5 → 63.0. The source page warns that the +4.0 general-capability gain appears only with teacher-generated trajectories (self-generated RFT data gives 0.0), so it is partly distillation from a 2.4T-parameter teacher.

## LLMs around the recommender

- **[[coral-llm-native-harness-production-recommender-azhar-2026|CORAL]] (Meta):** an LLM agent tunes a production recommender's hand-set parameters (retrieval budgets, serving tiers) in an observe → reason → optimize → measure loop, with a constrained optimizer that guarantees the budget. Live A/B: +0.16% sessions and +0.15% watch time at no extra serving cost.
- **[[lifecycle-of-llm-as-a-judge-recommendation-explanations-kong-2026|LLM-as-a-judge lifecycle]] (Netflix):** an [[llm-as-a-judge]] gates and critiques generated recommendation explanations; a five-week A/B test gave +0.2% relative novel-content viewing and +0.3% sessions with a successful play.

## Open questions

- Absolute numbers are mostly withheld (GenRec reports only relative metrics), and online effects are fractions of a percent.
- Whether explicit reasoning helps ranking is unsettled; the only evidence is metric-dependent.
- Vela and GenRec both mix behaviour signal with distillation or reward-model weighting, so the contribution of the behaviour data alone is hard to isolate.

## Related

- [[context-engineering]] · [[agentic-harness-engineering]] · [[llm-as-a-judge]] · [[knowledge-distillation]] · [[reward-modeling]] · [[meta]]
