---
title: "Harness-Zero: Harness Distillation via Agent-as-Harness"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - harness-zero-harness-distillation-agent-as-harness-ye-2026.md
arxiv_id: "2609.24974"
authors:
  - Haoran Ye
  - Yuxing Lu
  - Haonan Dong
  - Zhaochen Su
  - Guojie Song
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Harness-Zero: Harness Distillation via Agent-as-Harness

**Haoran Ye, Yuxing Lu, Haonan Dong, Zhaochen Su, Guojie Song** ([[peking-university|Peking University]]; [[google|Google]]; HKUST) — [arXiv:2609.24974](https://arxiv.org/abs/2609.24974), v1, cs.AI, 21 Sep 2026. Preprint; no venue. Code: `github.com/metaevo-ai/harness-zero`.

## Summary

Harness optimization leaves its gains in the harness. Because the best harness differs by domain and model, a general agent must either accept a weaker shared harness or route among many specialized ones. Harness-Zero moves the gains **into the weights**. First, an optimized harness h⋆ is evolved on training tasks, here with a simple three-round skill-guided loop on DeepAgents-style tools, middleware, skills and memory. h⋆ is then adapted into a *private reference harness* K. The student runs under a minimal **target harness** h (a mini-SWE-agent with one Bash tool). A separate **harnessing agent** reads K and reviews each proposed response before it executes. It either passes the response or substitutes the smallest coherent correction, written in the student's own action space. The harnessing agent cannot see the sandbox, the hidden solutions or the verifier. The reviewed trajectories are then used for [[supervised-fine-tuning|SFT]] ([[lora|LoRA]], 2 epochs, Qwen3.5-9B). At deployment h⋆, K and the harnessing agent are all removed.

The authors call the method **agent-as-harness**: guidance from one harness, re-expressed one response at a time as supervision for another harness with a different action space.

## Key points

- **Inference only, no training** (Table 1; 3 benchmarks × DeepSeek-V4-Pro and GPT-5.6 Sol; the same model plays student and harnessing agent). mini-SWE-agent averages 68.6, h⋆ mounted as code 78.1, agent-as-harness with an **empty** K 69.2, and agent-as-harness with the evolved K **81.1**. Review alone buys ~0.6; the evolved guidance buys the rest. On AppWorld with GPT-5.6 Sol nothing helps (96.4 baseline, already near ceiling).
- **Distillation** (Table 2; Qwen3.5-9B deployed under h alone):

| Setting | SpreadsheetBench | [[appworld\|AppWorld]] | USPTO | Avg |
|---|---|---|---|---|
| Base, h | 31.0 | 26.8 | 12.0 | 23.3 |
| Base, h⋆ mounted | 39.0 | 48.2 | **38.0** | 41.7 |
| Base, DeepAgents | 35.0 | 19.6 | 7.0 | 20.5 |
| Base, [[claude-code\|Claude Code]] | 31.0 | 10.7 | 6.0 | 15.9 |
| **Harness-Zero, h** | **44.0** | **58.9** | 30.0 | **44.3** |

  With the harness removed, the distilled 9B beats itself with the harness attached on two of three domains. Generic heavyweight harnesses *hurt* the 9B model.
- **Procedure transfers; domain knowledge does not fully transfer.** On USPTO retrosynthesis the harness supplies reaction priors and SMILES validation tools. Distillation reaches 30.0 against h⋆'s 38.0.
- **What makes the supervision work** (Table 3, USPTO, 500 collection tasks): SFT on teacher (GPT-5.6 Sol) rollouts under h gives 12.0, no gain, despite 52% collection success. Teacher under h⋆: 3.0. Student under h⋆: 12.0. Review with empty K: 11.0. **Review with the oracle answer: 15.0, despite 98.6% collection success.** Harness-Zero's K: **30.0 at only 59.4% collection success.** Collection success does not predict distillation value, and answer access teaches shortcuts.
- **Behavior recovery.** 82.3% average recovery across 28 harness-exclusive patterns (e.g., "reload saved workbook" 17/17, "execute final script" 52/75).

## What it does not show

- **No matched-compute comparison.** Agent-as-harness adds a second full model call at every turn, and code-as-harness has no such overhead. The 81.1 vs 78.1 comparison is therefore not cost-matched, and no paper arm gives the base student equivalent extra sampling. For distillation the relevant control is the choice of supervision source, and Table 3 covers that well.
- Every number is **single-run pass@1** on 100-task test splits (AppWorld: 56 scenarios). One student model (9B), and one harnessing model (GPT-5.6 Sol) for distillation.

## Concepts & entities

- [[agentic-harness-engineering]] — Harness-Zero is a *consumer* of harness evolution: any optimizer can produce h⋆.
- [[code-as-harness]] — the explicit foil. The paper argues an agent reading guidance adapts to model updates where code assumptions go stale.
- [[knowledge-distillation]] · [[on-policy-distillation]] — privileged-guidance distillation that stays on the student's own trajectory. It succeeds where off-policy teacher rollouts fail (12.0).
- [[harness-vs-model-attribution]] — a mechanism for turning a harness effect into a model effect.
- [[co-evolving-harnesses-and-models-on-policy-correction-yu-2026]] — the alternating co-evolution line this contrasts with: there, gains stay coupled to the harness.
- [[meta-harness-lee-2026]] · [[agentic-harness-engineering-lin-2026]] — named as interchangeable sources of h⋆.
- [[appworld]] · [[lora]] · [[supervised-fine-tuning]].

## References

- [arXiv:2609.24974](https://arxiv.org/abs/2609.24974)
