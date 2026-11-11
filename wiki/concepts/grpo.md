---
title: GRPO (Group Relative Policy Optimization)
type: concept
created: 2026-05-16
updated: 2026-11-11
sources:
  - deepseekmath-shao-2024.pdf
  - memory-r1-yan-2025.md
  - why-is-ai-generated-language-so-hard-to-understand-clark-2026.md
  - the-surprising-effectiveness-of-deleting-weights-lu-2026.md
status: draft
importance: medium
---

# GRPO (Group Relative Policy Optimization)

GRPO is a [[reinforcement-learning|reinforcement learning]] algorithm introduced in [[deepseekmath-shao-2024|DeepSeekMath]] (Shao et al., February 2024) and made widely known by [[deepseek-r1-2025|DeepSeek-R1]]. It is a variant of [[ppo|PPO]] that removes the value model.

## How it works

For each question:

1. Sample a group of `G` outputs from the current policy.
2. Score each output with a reward model or a rule-based check.
3. Normalize within the group: `(rᵢ − mean(r)) / std(r)`. With outcome supervision, every token in an output gets that output's value as its advantage.
4. Update with PPO's clipped objective, minus a KL term against a reference policy.

The original paper gives two reasons for dropping the critic. A value model is usually about as large as the policy, and a per-token value function is hard to train when usually only the last token is rewarded. It also argues that within-group comparison suits reward models trained on comparisons between outputs.

## Limits

- **Sampling cost.** The saved value model is paid for with `G` rollouts per question. [[gepa-agrawal-2025|GEPA]] reports beating GRPO's 24K rollouts on 5 of 6 tasks with far fewer, although GRPO wins on AIME.
- **Reward quality still decides the outcome.** Removing the critic doesn't address [[reward-hacking]], and a group whose rewards are all equal has no contrast to learn from. The second point is this wiki's reading of the formula, not a claim from the paper.

## Where it appears

- [[memory-r1]] — Yan et al. (2025) train an agent's memory-write/edit/delete policy with GRPO, treating each memory operation as an action whose reward is the downstream answer correctness.
- [[ppo]] — the immediate predecessor and conceptual contrast point; GRPO swaps PPO's critic for a group-relative baseline.
- [[reinforcement-learning]] — sits in the policy-gradient family alongside [[ppo]] and [[dpo]].
- [[rl-memory-policy]] — RL formulations where the action space consists of memory operations rather than tokens.

## References

- [[deepseekmath-shao-2024]]
- [[deepseek-r1-2025]]
- [[memory-r1-yan-2025]]

## Related 2026 sources

- [[score-centering-stabilizes-off-policy-rl-marek-2026]] — Sampler–trainer mismatch compounds into drift; subtracting the expected score cancels it where truncated IS fails
- [[salesforce-koa-enterprise-agentic-tool-use-salesforce-2026]] — Salesforce Koa: GRPO on environments built from agent configs; small gains, and its SFT checkpoint beats the RL release on 2 of 3

## Lineage sources

- [[gepa-agrawal-2025]] — GEPA beats GRPO (24K rollouts) on 5 of 6 tasks with up to 35× fewer rollouts; GRPO wins AIME

## Newer evidence (2026-11-10 sweep)

- GRPO with a length penalty on Qwen2.5-1.5B makes the model pick the shorter, less human-like construction over 98% of the time (without KL). ([[why-is-ai-generated-language-so-hard-to-understand-clark-2026]])

## Newer evidence (2026-11-11)

- GRPO with only a learned binary mask over gate projections ([[deletion-tuning|Deletion Tuning]]) matches full-fine-tuning GRPO at Llama3-8B, Qwen2.5-7B/14B and Qwen3-30B-A3B, and raises pass@128 over the base model. ([[the-surprising-effectiveness-of-deleting-weights-lu-2026]])
