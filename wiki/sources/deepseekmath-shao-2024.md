---
title: "DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - deepseekmath-shao-2024.pdf
arxiv_id: "2402.03300"
authors:
  - Zhihong Shao
  - Peiyi Wang
  - Qihao Zhu
  - Runxin Xu
  - Junxiao Song
year: 2024
introduces:
  - [[grpo]]
status: complete
importance: high
tags:
  - 2024
  - paper
---

# DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models

Zhihong Shao and co-authors, [[deepseek|DeepSeek]] and collaborators. [arXiv:2402.03300](https://arxiv.org/abs/2402.03300), first posted 5 February 2024. The author list above gives the first five names from the arXiv record.

## Summary

The paper trains DeepSeekMath 7B from DeepSeek-Coder-Base-v1.5 7B on 120B math-related tokens from Common Crawl, plus language and code data. It reports 51.7% on competition-level MATH without external tools or voting, and 60.9% with self-consistency over 64 samples. It credits two things: a data-selection pipeline, and a new RL algorithm, [[grpo|GRPO]].

## Key points

- **The problem with [[ppo|PPO]].** The value model is usually about the size of the policy model, which costs memory and compute. In LLMs, usually only the last token gets a reward, which makes a per-token value function hard to train.
- **GRPO's baseline.** It drops the value model and uses the average reward of several outputs to the same question.
- **Advantage.** With outcome supervision, each token's advantage is the output's reward minus the group mean, divided by the group standard deviation.
- **KL term.** It adds KL divergence from a reference policy to the loss, instead of penalizing the reward as PPO commonly does for LLMs.
- **Process supervision.** A process reward model scores each reasoning step, and each token's advantage sums the normalized rewards of later steps.
- **Iterative RL.** New samples retrain the reward model, with 10% replayed history, and the policy then becomes the new reference.
- **Comparisons.** The paper reports that GRPO beats online RFT, and that process supervision beats outcome supervision.

## What it does not show

The experiments cover math reasoning on one 7B model family. They say nothing about general agent tasks, where GRPO's later use rests on other sources.

## Concepts & entities

- [[grpo]] · [[ppo]] · [[reinforcement-learning]] · [[policy-optimization-methods]]
- [[deepseek]] · [[deepseek-r1-2025]]

## References

- [arXiv:2402.03300](https://arxiv.org/abs/2402.03300)
