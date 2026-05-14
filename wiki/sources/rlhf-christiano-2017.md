---
title: "Deep Reinforcement Learning from Human Preferences"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - christiano-rlhf-2017.md
arxiv_id: "1706.03741"
venue: NeurIPS 2017
authors:
  - "[[paul-christiano]]"
  - "[[jan-leike]]"
  - "[[tom-brown]]"
  - Miljan Martic
  - Shane Legg
  - "[[dario-amodei]]"
first_author: "[[paul-christiano]]"
introduces:
  - "[[rlhf]]"
  - "[[preference-modeling]]"
year: 2017
tags:
  - 2017
status: complete
importance: high
---

# Deep Reinforcement Learning from Human Preferences

**Source**: `sources/christiano-rlhf-2017.md` (extracted from `christiano-rlhf-2017.pdf`)
**arXiv**: [1706.03741](https://arxiv.org/abs/1706.03741)
**Venue**: NeurIPS 2017
**Authors**: [[paul-christiano]] ([[openai|OpenAI]]), [[jan-leike]] ([[deepmind|DeepMind]]), [[tom-brown]] (independent), Miljan Martic, Shane Legg (DeepMind), [[dario-amodei]] (OpenAI)

## Summary

This paper is the **foundation of [[rlhf|RLHF]]** — five years before it powered InstructGPT and ChatGPT. It tackles a fundamental RL problem: many tasks have goals that are easy for humans to recognize but hard to specify as a reward function. Cleaning a table, scrambling an egg, doing a backflip in a simulator — try writing the reward.

The approach: collect **pairwise preference judgments** from humans over short trajectory segments. Train a **reward model** to predict which segment a human would prefer. Then run standard deep RL (TRPO / A2C) against the reward model. Periodically refresh the reward model with new human comparisons. Crucially, **less than 1% of the agent's interactions need human feedback** — the reward model generalizes.

Empirically, this approach learns:
- **Atari games** (e.g. Pong, Enduro) — comparable to RL with the true game score.
- **Simulated robot locomotion** — a backflip in MuJoCo from about an hour of human time, no hand-crafted reward.

The framing of the paper is *AI safety / specification* — the introduction explicitly cites alignment concerns (Bostrom 2014, Russell 2016, Amodei et al. 2016) and frames RLHF as a way to communicate complex objectives to powerful RL systems. This was years before "alignment" was a mainstream ML concept.

The InstructGPT / ChatGPT pipeline ([[instructgpt-ouyang-2022|Ouyang et al. 2022]]) is a direct application of the Christiano recipe to text: collect human preferences over LLM outputs, train a reward model, RL the policy LM against it. The basic shape is identical.

## Key Points

- **The problem**: RL needs a reward function, but for many real-world tasks the reward is hard to specify; misspecified rewards lead to misaligned behavior.
- **The fix**: learn the reward from pairwise human comparisons over trajectory segments. Don't even bother with a dense reward.
- **Pipeline**:
  1. Run RL with current reward model → produce trajectories.
  2. Sample pairs of segments → ask human which is better.
  3. Update reward model on the labeled pairs.
  4. Repeat.
- **Reward model**: a neural net trained on pairwise preferences with a Bradley-Terry-style logistic loss: `P(σ¹ ≻ σ²) = σ(r̂(σ¹) − r̂(σ²))`.
- **Sample efficiency**: less than 1% of agent interactions need feedback. The reward model generalizes; the RL agent does most of the work.
- **Tasks demonstrated**:
  - Atari (Pong, Qbert, Beamrider, etc.) at comparable performance to standard RL with the true score.
  - Simulated robot locomotion (Hopper, Walker, etc.) with backflips and other novel behaviors learned from ~1 hour of human time.
- **Safety framing** explicit in the introduction — a notable early example of an ML paper motivating itself with alignment concerns.

## Entities Mentioned

- [[paul-christiano]] — first author (founded ARC and later Alignment Research Center)
- [[jan-leike]] — co-author (later led OpenAI Superalignment team; departed in 2024 with public criticism)
- [[tom-brown]], [[dario-amodei]] — early-OpenAI co-authors (both later Anthropic founders)
- Shane Legg — DeepMind co-founder

## Concepts Discussed

- [[rlhf]] — this is the foundational paper
- [[preference-modeling]] — pairwise preference loss
- [[reward-modeling]] — learning reward functions from human signal

## Notable Quotes

> "We explore goals defined in terms of (non-expert) human preferences between pairs of trajectory segments. We show that this approach can effectively solve complex RL tasks without access to the reward function ... while providing feedback on less than 1% of our agent's interactions with the environment." — Abstract

> "This difficulty underlies recent concerns about misalignment between our values and the objectives of our RL systems." — §1

## References

_Original source: `sources/christiano-rlhf-2017.md`_
