---
title: "Hybrid Reasoning Agents"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - hybrid-reasoning-agents-zou-2026.md
arxiv_id: "2610.18861"
authors:
  - Jiaru Zou
  - Yunzhe Qi
  - Sirui Chen
  - Federico Bianchi
  - Shang Zhu
  - Pan Lu
  - Jingrui He
  - James Zou
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Hybrid Reasoning Agents

**Jiaru Zou, Yunzhe Qi, Sirui Chen, Federico Bianchi, Shang Zhu, Pan Lu, Jingrui He, James Zou** ([[stanford-university|Stanford]]; Together AI; [[uiuc|UIUC]]; [[meta|Meta]]): [arXiv:2610.18861](https://arxiv.org/abs/2610.18861), v1, cs.CL, 13 Oct 2026. Preprint; no venue.

## Summary

[[latent-reasoning|Latent reasoning]] (Coconut-style continuous thoughts) is efficient but cannot call tools and is hard to inspect. Text agents can act but must verbalize every step. **HybridAgent** switches between the two within a single generation. **HybridLink** is a lightweight residual adapter (Linear → GELU → Linear, plus a residual) that maps last-layer hidden states back into the input-embedding space. Between `<latent>` and `</latent>` the model recurs on its own hidden states. In text mode it emits tool calls and reads observations. A continuous *working memory* carries the mixed text/latent/observation context across turns without loss. The authors report that latent steps can be **decoded as readable pseudo-text** through the LM head, so the latent thoughts can be inspected.

Training has two stages. **HybridSFT** uses 54K automatically annotated hybrid trajectories, with *self-conditioned supervision* so the backbone learns to coordinate with HybridLink. **HybridRL** uses 12K deep-reasoning and 1K agentic-search examples, 512-trajectory rollout batches, and a composite reward; it learns *when* to think latently and when to act in text.

## Key points

- **Setup.** Qwen3-Base backbones at **4B, 8B and 14B**. 11 benchmarks:
  - Math: AIME25, AIME26, MATH500.
  - Knowledge: GPQA-Diamond, MedQA, MuSiQue.
  - Agentic: [[gaia|GAIA]], HLE, WebWalker, SealQA, DeepSearchQA.
- **Baselines.** Text agentic methods (SFT, GRPO, ARPO). Latent methods (SoftThinking, Coconut, SoftCoT). Frontier agents (GPT-5.6-Luna, Kimi-K3, GLM-5.3-Flash, DeepSeek-V4.1-Flash, QwQ-32B). Frameworks ([[react|ReAct]], Search-o1, WebThinker).
- **Headline gains** (relative %):
  - HybridAgent-14B beats the strongest agentic baseline by an average of **17.4%** on the agentic benchmarks.
  - At matched size and training budget, it beats text/latent baselines by **18.5%** at the SFT stage and **10.4%** at the RL stage.
  - At 4B, HybridRL beats ARPO by **2.0 percentage points** on average across four benchmarks.
- **Efficiency vs matched text agents.** **35.5–54.7%** fewer tokens, up to 54% fewer generation steps, **1.8–2.6×** faster inference (2.1× in the body figure), and 2.7–3.7× lower cost.
- **Ablations.** Self-conditioned supervision adds 30.5% on average over HybridLink activation alone. Each reward component contributes (removing one costs up to 11.2% on GAIA). HybridAgent's pass@k curves rise more steeply with k than text-only RL agents' curves.

## What it does not show

> [!warning] Newsletter vs paper
> - The newsletter's "+10.4% avg over 11 benchmarks" mixes two comparisons. The abstract frames 10.4% as an improvement over "frontier LLM Agents and agentic frameworks". The body defines it as the **RL-stage gain over matched text/latent baselines**.
> - All the % gains appear to be **relative**. The one absolute figure given, at 4B, is +2.0 pp over ARPO.

- Agentic absolute scores are low for every model (for example, Qwen3-14B-thinking scores single digits on several HLE categories), so relative gains are inflated by small denominators.
- "Inspectable" latent thoughts are judged on readability and context-specificity over 200 samples. Whether the pseudo-text is *faithful* to the computation is not established. Compare [[how-transparent-is-diffusiongemma-engels-2026]].
- Frontier baselines use their own default settings. The speedups are measured against matched text-mode agents, not against the frontier systems.

## Concepts & entities

- [[latent-reasoning]] · [[chain-of-thought]] · [[recurrence]]: the latent mode is sequence-wise recurrence on hidden states. See [[the-latent-space-yu-2026]] and [[latent-reasoning-with-normalizing-flows-tu-2026]].
- [[tool-use]] · [[agentic-search]] · [[agentic-rl]] · [[grpo]].
- [[test-time-compute]] · [[reasoning-effort]]: fewer tokens at higher accuracy.
- [[logit-lens]]: decoding hidden states through the LM head is the readability mechanism.
- [[stanford-university]] · [[qwen]].

## References

- [arXiv:2610.18861](https://arxiv.org/abs/2610.18861) · Code: togethercomputer/HybridAgent
