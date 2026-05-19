---
title: DeepSeek-R1
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - deepseek-r1-2025.md
status: complete
importance: high
tags:
  - 2025
---

**DeepSeek-R1** refers to both the open-weights reasoning model released by [[deepseek]] in January 2025 and the training recipe that produced it. The work's central empirical claim is that advanced reasoning behaviors — self-reflection, verification, backtracking — can be **incentivized through pure reinforcement learning** with rule-based outcome rewards, without any human-labeled reasoning trajectories. The result is a model that matches OpenAI's o1 series on math, code, and graduate-level QA benchmarks (AIME 2024 pass@1 = 79.8%, MATH-500 = 97.3%, Codeforces 96.3rd percentile) while being open-weights and accompanied by distilled 7B / 32B / 70B variants. Because the recipe and weights are public, R1 became the substrate for essentially every 2025 [[agentic-rl]] paper that followed.

## Core mechanism

- **R1-Zero pure-RL stage**: GRPO applied directly to DeepSeek-V3-Base with no SFT, constrained only by a `<think>...</think><answer>...</answer>` template. AIME 2024 pass@1 climbs from 15.6% to 77.9% over ~10.4k steps; emergent self-verification appears spontaneously, with a documented "aha moment" where the model starts re-evaluating its own work using anthropomorphic phrasing.
- **Multi-stage R1 with SFT cold-start**: (1) SFT on thousands of long-CoT conversational examples → (2) RL with rule-based + language-consistency rewards → (3) rejection-sampling SFT on mixed reasoning/non-reasoning data → (4) final RL combining rule-based and preference rewards. Each stage targets a specific failure of the previous (readability, generality, alignment).
- **[[grpo]] with rule-based rewards**: PPO variant that drops the value network and uses group-relative advantages over G sampled rollouts; rewards are accuracy (boxed-answer match for math, compiler/tests for code) plus format adherence. No neural reward model on reasoning tasks — explicitly to prevent reward hacking.
- **Emergent self-verification / "aha moment"**: chain-of-thought length grows from hundreds to thousands of tokens autonomously; "wait" tokens spike at the moment self-reflection emerges; the model develops systematic exploration of alternative solutions without process-level supervision.
- **Open-weights release with distilled checkpoints**: full R1 series plus distillations into Qwen-7B/32B and Llama-70B dense models, all on Hugging Face. The distilled small models outperform their instruction-tuned baselines, making the recipe accessible at consumer scale.

## What it advances

DeepSeek-R1 is the open-weights inflection point that operationalized reasoning-RL for the public. By showing that pure-RL on a strong base model elicits o1-class reasoning with a minimal reward design, it shifted the field's center of gravity from supervised CoT distillation toward outcome-reward post-training. The released weights and the GRPO recipe seeded a wave of replications and extensions into tool-use ([[search-r1]]), memory ([[memory-r1]], [[mem-alpha]]), computer-use ([[ui-tars-2]]), and self-improving agents — collectively defining the 2025 [[agentic-rl]] paradigm.

## Relation to other concepts

Direct ancestor of [[search-r1]] (extends R1-Zero to retrieval), [[memory-r1]] (extends to memory operations), and most other `*-R1` agentic-RL papers from 2025. Builds on [[chain-of-thought-prompting]] as the surface form inside `<think>` blocks, and on [[reasoning]] as the capability being incentivized. Uses [[grpo]] from DeepSeek's own DeepSeekMath work; contrasts with [[rlhf]] (preference-fitting) and [[constitutional-ai]] (rule-based preferences) by optimizing rule-based correctness directly. The base model [[deepseek-v3]] supplies the parametric prior.

## References

- [[deepseek-r1-2025]] — primary source ([arXiv:2501.12948](https://arxiv.org/abs/2501.12948)).
- Weights: <https://huggingface.co/deepseek-ai>
- GRPO origin: Shao et al. 2024 (DeepSeekMath).
- Implicit comparison target: OpenAI o1 (2024).
