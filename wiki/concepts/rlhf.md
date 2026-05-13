---
title: RLHF (Reinforcement Learning from Human Feedback)
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-4-openai-2023.md
status: complete
importance: high
---

# RLHF (Reinforcement Learning from Human Feedback)

RLHF is the dominant post-training procedure for aligning large language models to human preferences. The canonical pipeline (Christiano et al. 2017 in RL; Stiennon et al. 2020 and Ouyang et al. 2022, "InstructGPT", for LLMs):

1. **Supervised fine-tuning (SFT)**: fine-tune the pre-trained base model on a curated dataset of (instruction, response) pairs written by human annotators or selected from existing examples. This gives a "policy" model that follows instructions.
2. **Reward model (RM) training**: human raters rank multiple candidate responses to the same prompt. Train a reward model — typically a fine-tuned LM with a scalar head — to predict the human-preferred ranking. The RM is the learned proxy for "human preference."
3. **Policy optimization**: fine-tune the SFT policy with reinforcement learning (PPO is the canonical choice) to maximize the reward model's score, with a KL-divergence penalty against the SFT policy to prevent reward hacking.

## Why RLHF matters

Pre-trained LMs are trained to maximize next-token likelihood — they imitate the data distribution, which includes ~unlimited unhelpful, harmful, or off-target continuations. RLHF turns them into **helpful assistants** by directly optimizing for what humans prefer. The transformation from "GPT-3 base" (capable but uncontrolled) to "InstructGPT" (capable and follows instructions) is the canonical example.

Every major deployed LLM uses some form of RLHF or close cousin: [[gpt-4|GPT-4]], Claude, Gemini, LLaMA-2-Chat, Mistral-Instruct, etc.

## Variants and successors

- **DPO (Direct Preference Optimization)**, Rafailov et al. 2023: skip the reward model and PPO; directly optimize the policy on preference data via a contrastive loss. Simpler, often as effective.
- **IPO, KTO, ORPO, SimPO**: variants of DPO with different loss formulations.
- **RLAIF (RL from AI Feedback)**: an AI model (constitutional or critic) provides preferences instead of (or alongside) humans. Used in Anthropic's Claude (constitutional AI).
- **Iterated / online RLHF**: re-collect preferences and update the RM/policy in iterations.
- **Process supervision**: reward intermediate reasoning steps, not just final answers (used in math reasoning models).

## Limitations

- **Reward hacking**: the policy finds ways to maximize the proxy reward without satisfying the underlying objective.
- **Inconsistent / biased preferences**: human raters disagree; aggregate "human preference" is fuzzy.
- **Mode collapse / sycophancy**: RLHF'd models can become bland, agreeable, or evasive.
- **Capability tax**: some research suggests RLHF can hurt raw capability (vs. base model) on certain tasks even as it improves usefulness.

## References

- [[gpt-4-openai-2023]]
