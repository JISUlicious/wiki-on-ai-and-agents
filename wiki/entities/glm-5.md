---
title: GLM-5
type: entity
created: 2026-09-29
updated: 2026-09-29
sources:
  - skillmoo-gong-2026.md
  - misa-mixture-of-indexer-sparse-attention-zhou-2026.md
  - skillreducer-gao-2026.md
family: glm
successors:
  - [[glm-5-2]]
parameters: 744B total / 40B active
status: draft
importance: medium
tags:
  - model
---

# GLM-5

Open-weight [[mixture-of-experts|MoE]] foundation model from [[z-ai|Z.ai]] (Zhipu AI), described in "GLM-5: from Vibe Coding to Agentic Engineering" (arXiv, February 2026). It has **744B total parameters with 40B active** and was pre-trained on 28.5T tokens. It adopts **DeepSeek Sparse Attention (DSA)** to cut training and inference cost at long context, and it introduces an asynchronous RL infrastructure that decouples generation from training. It is the direct predecessor of [[glm-5-2|GLM-5.2]].

## In this wiki

- **Sparse-attention retrofit target.** [[misa-mixture-of-indexer-sparse-attention-zhou-2026|MISA]] is a training-free retrofit that applies only to models already trained with DSA: DeepSeek-V3.2 and GLM-5. With GLM-5's 32 indexer heads, routing to 8 is a 4× indexer cut, and MISA scores slightly *above* DSA there. The ingest notes this is the GLM-5 base model, not [[glm-5-2]], whose IndexShare saves along the layer axis instead. See [[trained-sparse-attention]].
- **Cost-efficient agent backbone.** [[skillmoo-gong-2026|SkillMOO]] runs both its solver and optimizer agents on GLM-5 for its cost-to-capability ratio and API availability. Per-candidate evaluation cost $0.10–$14. The paper flags single-model results as a threat to validity.
- **Evaluated model.** One of five models across which [[skillreducer-gao-2026|SkillReducer]]'s compressed skills keep 0.939–0.986 retention.

## Lineage

It extends the GLM-4.5 line (355B / 32B active) by roughly doubling total parameters and raising pre-training data from 23T to 28.5T tokens. It adds DSA-style [[sparse-attention]] borrowed from [[deepseek]]'s V3.2. [[glm-5-2|GLM-5.2]] (June 2026) keeps the 744B / 40B-active scale, extends context to 1M and adds IndexShare.

## Related

- [[glm-5-2]] · [[z-ai]] · [[mixture-of-experts]] · [[trained-sparse-attention]] · [[deepseek-v3]] · [[agentic-rl]]

## References

- [GLM-5: from Vibe Coding to Agentic Engineering, arXiv:2602.15763](https://arxiv.org/abs/2602.15763) — DSA, asynchronous RL infrastructure
- [zai-org/GLM-5 README](https://github.com/zai-org/GLM-5) — 744B / 40B active, 28.5T tokens, scaling from GLM-4.5
- [[skillmoo-gong-2026]] · [[misa-mixture-of-indexer-sparse-attention-zhou-2026]] · [[skillreducer-gao-2026]]
