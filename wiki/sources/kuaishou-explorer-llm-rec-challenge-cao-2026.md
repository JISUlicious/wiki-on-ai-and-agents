---
title: "KUAISHOU Explorer LLM-Rec Challenge 2026: Reasoning Generative Recommendation"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - kuaishou-explorer-llm-rec-challenge-cao-2026.md
arxiv_id: "2609.39828"
venue: SIGIR 2026 challenge report
authors:
  - Jiangxia Cao
  - Hao Peng
  - Wenlong Xu
  - Jiaxin Deng
  - Kun Gai
year: 2026
status: complete
importance: low
tags:
  - 2026
  - paper
---

# KUAISHOU Explorer LLM-Rec Challenge 2026: Reasoning Generative Recommendation

**OneRec Team, Kuaishou.** The arXiv record lists about 110 authors, starting with Jiangxia Cao, Hao Peng and Wenlong Xu and ending with Kun Gai; academic co-authors include Xiangnan He and Jirong Wen. [arXiv:2609.39828](https://arxiv.org/abs/2609.39828), v1, cs.IR, 30 Sep 2026. Challenge report, organized with the SIGIR 2026 community.

## Summary

Kuaishou's OneRec / OneRec-V2 models are deployed generative recommenders. They predict the next item autoregressively over **Semantic IDs** (SIDs): a domain marker plus three RQ-KMeans codes with 8,192 codewords per level. The follow-ups OneRec-Think, OpenOneRec and **OneReason** tie SIDs to natural language so the model can reason in [[chain-of-thought]] before recommending.

The report's starting premise is that **CoT reasoning did not reliably improve recommendation**: thinking mode could underperform direct generation. OneReason responds with stronger SID–language alignment, templated interest-reasoning supervision and RL. It is deployed in Kuaishou advertising. The challenge asked teams to post-train one OneReason model (a [[qwen|Qwen3]] base, about 578B pre-training tokens over three stages) on four capabilities: understanding items, users, recommendation and the world.

## Key points

- **Released resources:** OneReason-0.8B/8B-pretrain checkpoints; 944,802 SFT seed examples (the intro says 792,797, of which 412,438 have CoT); 500,000 anonymized users with 733M interactions across short video, e-commerce, ads and live streaming; SID/caption alignment for 35.9M items.
- **Tasks:**
  - SID → caption (weighted F1);
  - selecting topic-relevant behaviour (F1);
  - interest-evolution chains;
  - cross-domain recommendation, scored by Pass@32 on the **union** of think and no-think candidates, which deliberately rewards thinking for finding *different* items;
  - multiple-choice general knowledge.
- **Scale:** 2,031 participants in 1,206 teams; 812 valid in the preliminary round (33,059 submissions); 50 semi-finalists on 4×80 GB GPUs; 20 finalists after reproducibility checks within 72 hours. The organizers also ran leakage checks on user histories, SIDs and semantic pairs.
- **Champion:** self-sampled reasoning trajectories that hit the target, with the **target-SID loss masked**. Supervising the answer collapsed think/no-think diversity. Self-sampled traces beat traces from a larger teacher, and adding more traces reduced the gains.
- **Runner-up:** SFT data only. Product descriptions were rewritten under constraints. Think mode was kept for e-commerce and no-think used for the other three domains, which beat a uniform format.
- **Third place:** staged SFT → RL → [[dpo|DPO]]. DPO uses log-derived positives and SID-hierarchy negatives, because online sampling gave all-zero reward groups.

## What it does not show

- **No ablation for the central claim.** That reasoning helps recommendation is shown only through the challenge metric, and that metric is built to reward think-mode diversity. Absolute hit rates are low: full-SID hit rates in the third-place team's training curves (Figure 4) are about 2–4%.
- It is a challenge report. Winning-team results are summarized from their reports, not reproduced as tables.

## Concepts & entities

- [[chain-of-thought]] · [[dpo]] · [[grpo]] · [[supervised-fine-tuning]] · [[qwen]]
- [[genrec-llm-recommendation-ranker-netflix-li-2026]]: Netflix's prefill-only scoring head as the alternative to SID decoding.
- [[vela-user-behavior-as-supervision-xiao-2026]] · [[coral-llm-native-harness-production-recommender-azhar-2026]]
- [[generative-recommendation]]: concept page comparing LLM recommendation designs (Semantic IDs vs catalog scoring) and LLMs around recommenders.

## References

- [arXiv:2609.39828](https://arxiv.org/abs/2609.39828)
- Challenge site: ks-llmrec.streamlake.com
