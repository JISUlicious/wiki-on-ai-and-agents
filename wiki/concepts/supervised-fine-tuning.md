---
title: Supervised Fine-Tuning (SFT)
type: concept
created: 2026-09-29
updated: 2026-09-30
sources:
  - embarrassingly-simple-self-distillation-zhang-2026.md
  - pivotrl-high-accuracy-agentic-post-training-yi-2026.md
  - instructgpt-ouyang-2022.md
  - deepseek-r1-2025.md
status: draft
importance: medium
---

# Supervised Fine-Tuning (SFT)

**Supervised fine-tuning** trains a pre-trained model by maximum likelihood on (input, target-output) demonstrations. It is ordinary next-token cross-entropy, applied to curated responses instead of raw text. [[fine-tuning]] is the broader term: any continued training of a pre-trained model, including RL-based and parameter-efficient methods. SFT is its *supervised imitation* branch. In modern LLM post-training, SFT names a specific stage: the step that turns a base model into an instruction follower ([[instruction-tuning]]) and that usually comes before preference or RL training.

## In this wiki

- **Stage one of the classic pipeline.** [[instructgpt-ouyang-2022|InstructGPT]] runs SFT, then a [[reward-modeling|reward model]], then [[ppo|PPO]]. The SFT model also serves as the reference policy in [[rlhf]] and [[dpo]]. [[constitutional-ai]] opens with a critique-and-revision SFT phase.
- **Cold start, or skip it.** [[deepseek-r1]] uses a small long-CoT SFT cold start before RL. R1-Zero drops SFT entirely, and [[search-r1]] follows that no-SFT choice.
- **SFT versus RL.** [[pivotrl-high-accuracy-agentic-post-training-yi-2026|PivotRL]] names the tension: SFT is compute-efficient but degrades out of domain, while end-to-end RL keeps OOD ability at high rollout cost. PivotRL runs local RL on existing SFT trajectories. [[sycophancy]] records that SFT raised over-conceding while PPO lowered it.
- **SFT on the model's own outputs.** [[embarrassingly-simple-self-distillation-zhang-2026|Simple Self-Distillation]] samples the model at a tuned temperature and runs plain SFT on those samples, with no verifier, teacher or RL. It lifts Qwen3-30B-Instruct from 42.4% to 55.3% on LiveCodeBench v6. [[on-policy-distillation]] sits between SFT and [[grpo|GRPO]] in cost and behaviour.
- **SFT as a stage in a data flywheel.** In [[ui-tars-2]] and [[computer-use-agent|computer-use]] pipelines, SFT sits between continual pre-training and multi-turn RL. [[illicit-distillation]] documents SFT on harvested frontier-model traces.

## Related

- [[fine-tuning]] · [[instruction-tuning]] · [[pre-training]] · [[lora]] · [[knowledge-distillation]] · [[on-policy-distillation]]
- [[rlhf]] · [[dpo]] · [[ppo]] · [[grpo]] · [[reinforcement-learning]] · [[agentic-rl]]

## References

- [[instructgpt-ouyang-2022]] · [[deepseek-r1-2025]] · [[pivotrl-high-accuracy-agentic-post-training-yi-2026]] · [[embarrassingly-simple-self-distillation-zhang-2026]]

## Related 2026 sources

- [[salesforce-koa-enterprise-agentic-tool-use-salesforce-2026]] — Salesforce Koa: GRPO on environments built from agent configs; small gains, and its SFT checkpoint beats the RL release on 2 of 3
