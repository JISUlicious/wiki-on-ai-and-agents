---
title: "DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via RL"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - deepseek-r1-2025.md
arxiv_id: "2501.12948"
authors:
  - DeepSeek-AI
year: 2025
introduces:
  - "[[deepseek-r1]]"
  - "[[agentic-rl]]"
tags:
  - 2025
status: complete
importance: high
---

# DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via RL

**Source file**: [[deepseek-r1-2025]]
**arXiv**: [arXiv:2501.12948](https://arxiv.org/abs/2501.12948)
**Authors**: DeepSeek-AI team
**Year**: 2025 (Jan)

## Summary

DeepSeek-R1 demonstrates that advanced reasoning capabilities in LLMs can be **incentivized through pure reinforcement learning**, without any human-annotated reasoning trajectories. Starting from DeepSeek-V3-Base, the authors apply Group Relative Policy Optimization (GRPO) with purely rule-based outcome rewards (correctness + format) and obtain **DeepSeek-R1-Zero**, a model whose AIME 2024 pass@1 climbs from 15.6% to 77.9% over the course of training (86.7% with self-consistency). Critically, the model is never explicitly taught how to reason — instead, sophisticated patterns like self-reflection, verification, backtracking, and exploration of alternative solutions emerge spontaneously, accompanied by a steady increase in chain-of-thought length. The paper highlights an "aha moment" where the intermediate checkpoint suddenly starts re-evaluating its own work using anthropomorphic language ("Wait, wait. Wait.").

Because R1-Zero suffers from poor readability and English/Chinese language mixing, the authors then build the production-quality **DeepSeek-R1** via a multi-stage pipeline: (1) SFT cold-start on thousands of conversational long-CoT examples, (2) first RL stage with rule-based rewards plus a language-consistency reward, (3) rejection-sampling + SFT on mixed reasoning and non-reasoning data (Dev3), and (4) a final RL stage combining rule-based and model-based preference rewards for helpfulness/harmlessness. The resulting model reaches **AIME 2024 pass@1 = 79.8%, MATH-500 = 97.3%, Codeforces 96.3rd percentile (rating 2029)**, putting it in the same class as OpenAI's o1 series.

DeepSeek released the R1 family under open weights on Hugging Face, alongside distilled checkpoints at 7B/32B/70B scales that inherit much of R1's reasoning ability. The release became the substrate for essentially every 2025 agentic-RL paper that followed — open weights plus a recipe that demonstrably elicits reasoning from RL alone reshaped how the field thinks about post-training and triggered a wave of replications, ablations, and extensions into agentic / tool-use settings.

## Key Points

- **Pure-RL training (R1-Zero)**: GRPO applied directly to DeepSeek-V3-Base with no SFT phase. Outputs are constrained only by a structural template (`<think>...</think><answer>...</answer>`); content is unconstrained to allow the model to find its own reasoning style.
- **GRPO algorithm**: PPO variant that drops the value network — for each question, sample G outputs from the old policy, compute group-relative advantages by normalizing rewards within the group, then clip-update with a KL penalty against a reference policy. Cheaper than PPO at scale.
- **Rule-based rewards only (for reasoning)**: Accuracy reward (boxed-answer match for math, compiler/test-cases for code) + format reward (must use the think/answer tags). No neural reward model on reasoning tasks — the authors explicitly avoid these to prevent reward hacking.
- **Emergent behaviors**: Self-verification, reflective reasoning ("let me reevaluate"), systematic exploration of alternatives, and a measurable spike in "wait" tokens marking the aha moment. CoT length grows from hundreds to thousands of tokens autonomously.
- **R1 multi-stage pipeline**: cold-start SFT → reasoning RL with language-consistency reward → rejection-sampling SFT on mixed data → final RL with rule-based + preference rewards. Each stage targets a specific failure of the previous one (readability, generality, alignment).
- **Benchmark headline numbers**: AIME 2024 pass@1 = 79.8% (Cons@16 even higher); MATH-500 = 97.3%; CNMO 2024 = 78.8%; Codeforces 96.3rd percentile, rating 2029; LiveCodeBench 65.9%; SWE-Bench Verified 49.2%; GPQA Diamond 71.5%; MMLU-Pro 84.0%.
- **Open-weights release**: full R1 series plus distillations into 7B / 32B / 70B dense checkpoints, available at `huggingface.co/deepseek-ai`. The distilled small models outperform their instruction-tuned baselines.
- **Training infrastructure**: ~10,400 RL steps for R1-Zero, batch size 512, 16 rollouts per question, context grown from 32k to 65k tokens at step 8.2k (which coincides with the performance jump).

## Entities Mentioned

- [[deepseek]] (org) — Chinese lab that authored and released the model.
- [[deepseek-v3]] / [[deepseek-v3-base]] — the base model that R1 / R1-Zero are trained from.
- [[deepseek-r1-zero]] — the pure-RL intermediate model.
- [[deepseek-r1]] — the production multi-stage model.
- [[openai]] / [[o1]] — implicit comparison target (o1-class reasoning).

## Concepts Discussed

- [[agentic-rl]] — this paper is the de-facto substrate for the 2025 agentic-RL wave.
- [[reinforcement-learning]]
- [[grpo]] — Group Relative Policy Optimization.
- [[ppo]] — referenced as the algorithm GRPO simplifies.
- [[chain-of-thought-prompting]] / long-CoT reasoning.
- [[self-reflection]] / self-verification / backtracking.
- [[reasoning]]
- [[rule-based-rewards]] vs. neural reward models.
- [[reward-hacking]] — explicit motivation for avoiding neural reward models on reasoning data.
- [[knowledge-distillation]] — for the 7B / 32B / 70B distilled variants.
- [[sft]] (cold-start) and [[rejection-sampling]] within the multi-stage pipeline.
- Benchmarks: [[aime]], [[math-500]], [[codeforces]], [[livecodebench]], [[swe-bench]], [[gpqa]], [[mmlu-pro]].

## Notable Quotes

> "We show that the reasoning abilities of LLMs can be incentivized through pure reinforcement learning (RL), obviating the need for human-labeled reasoning trajectories. The proposed RL framework facilitates the emergent development of advanced reasoning patterns, such as self-reflection, verification, and dynamic strategy adaptation." (Abstract)

> "Rather than explicitly teaching the model how to solve a problem, we simply provide it with the right incentives, and it autonomously develops advanced problem-solving strategies. This serves as a reminder of the potential of RL to unlock higher levels of capabilities in LLMs." (§2.3)

## References

- arXiv: https://arxiv.org/abs/2501.12948
- Model weights: https://huggingface.co/deepseek-ai
- GRPO origin: Shao et al., 2024 (DeepSeekMath).
- Base model: DeepSeek-V3 (DeepSeek-AI, 2024b).
- Related prior art on CoT: Wei et al. 2022b; Kojima et al. 2022.
- Implicit comparison: OpenAI o1 (2024).
