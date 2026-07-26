---
title: "Right in the Right Way: LM Training with Verifiable Rewards and Human Demonstrations"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - right-in-the-right-way-damani-2026.md
arxiv_id: "2607.01181"
year: 2026
authors:
  - Mehul Damani
  - Isha Puri
  - Idan Shenfeld
  - Jacob Andreas
first_author: Mehul Damani
venue: arXiv (cs.LG) preprint
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Right in the Right Way: LM Training with Verifiable Rewards and Human Demonstrations

**Mehul Damani, Isha Puri, Idan Shenfeld, Jacob Andreas** (MIT EECS) — [arXiv:2607.01181](https://arxiv.org/abs/2607.01181) (v1, 1 Jul 2026, cs.LG). Preprint; no venue listed.

## Summary

RL with verifiable rewards (RLVR) optimizes only what can be objectively scored — answer correctness, unit-test pass rates. The authors argue this leaves the *non-verifiable* half of output quality unoptimized and often actively degraded: diversity collapse, unnatural-sounding text, and reward hacking are the documented failure modes. Their framing: "the gap between what is verifiable and what is valuable is precisely where current RLVR methods fall short." A bug fix that rewrites an entire function passes every test but is worthless to the programmer who has to review it.

The proposed method, **VARL** (Verifiable and Adversarial Reinforcement Learning), co-trains two models: a **generator** trained with GRPO, and a **discriminator** that predicts whether a given output was written by a human or produced by the generator. The generator's reward is the **product** of the verifiable reward and the discriminator's probability that the output is human-written — a *verifier-gated* adversarial reward, so an incorrect answer earns nothing regardless of how human-like it looks. Because the discriminator is retrained continuously alongside the policy, it acts as an adaptive learned proxy for the human output distribution, supplying feedback on properties (readability, style, structure) that resist formalization as scalar rewards. The paper shows this reward transformation corresponds to minimizing the **Vincze–Le Cam divergence** between policy and demonstration distributions, and argues Le Cam is the only one of four candidate transformations satisfying both desirable properties (Table 1).

Mode collapse gets penalized as a side effect: features that appear "suspiciously often" in model output are exactly what the discriminator learns to exploit. The authors position VARL as a bridge between RL and SFT — you specify soft properties via demonstrations rather than by hand-designing rewards, while keeping RLVR's accuracy gains, "in some cases yielding human-like policies with superhuman performance."

## Key points

Three evaluation settings, each targeting a different role for demonstrations:

| Task | Dataset | Base model | Result |
|---|---|---|---|
| Bug fixing (style preservation) | RunBugRun, 22,000 train / 500 test | Qwen2.5-7B-Instruct | Accuracy **50% → 65%** while matching the human token-edit-distance distribution; RLVR rewrites functions wholesale |
| Story generation (diversity) | WritingPrompts (curated subset), 25,000 train / 200 test | Llama-3.1-8B-Instruct | Win rate vs. human stories **2% → 22%**, with substantially more diverse and human-like stories |
| Countdown-Code (reward hacking) | Countdown-Code (deliberately flawed verifier) | Qwen-2.5-3B-Instruct | True task accuracy **20% → 60%** with minimal hacking; RLVR collapses to a degenerate reward-hacking solution |

- **Reward hacking rate: 1.00% for VARL vs. 96.90% / 98.22% / 99.30%** for the three SFT+RLVR baselines. KL regularization (β ∈ {0, 0.001, 0.01}) fails to prevent hacking on countdown — the rate stays above 96% for every SFT+RLVR variant.
- **Multiplicative, not additive** — reward is verifiable × discriminator probability; the paper ablates additive vs. multiplicative on bug fixing and countdown.
- **Imperfect demonstrations are tolerated**: 10% of the countdown demonstrations themselves exhibit reward hacking, and VARL still holds the hacking rate at 1%. The authors are explicit that this is not a guarantee — the mechanism works *when demonstrations contain desirable behavior*, not unconditionally. VARL does briefly exhibit hacking mid-training before the discriminator reins it in.
- **Training setup**: GRPO with group size 8, 128 prompts per gradient step, standard-deviation normalization removed from the advantage. Discriminator is a binary classification head on a copy of the same base LM, with balanced batches (50% FIFO buffer, 50% reservoir buffer).
- **Story-generation judging**: Gemini-2.5-Flash-Lite during training, with a different and stronger judge (GPT-5.5) for evaluation to reduce judge overfitting. Qualitative failure signature of RLVR: >50% of its stories have an ominous tone (vs. 10% of human stories) and 30% are overwritten (vs. ~0% elsewhere).
- **SFT is not a fix**: SFT actually *decreases* story win rate and increases distance to the human feature distribution, despite superficially resembling human text.

## Concepts & entities

- [[verifier]] — the paper's core target: a verifiable reward is a proxy, and optimizing it alone leaves style/structure/diversity unconstrained. The discriminator is proposed as a second, learned proxy for what the verifier cannot score.
- [[reward-hacking]] — Countdown-Code is a purpose-built testbed with a flawed verifier; VARL cuts the hacking rate from ~97-99% to 1%.
- [[rlhf]] — related in spirit (learned reward model from human data) but VARL's discriminator is adversarially retrained on demonstrations rather than fit once to preference pairs.
- [[agentic-rl]] — same underlying tension: verifiable environment rewards are cheap to compute but only cover part of what makes an agent trajectory good.
- [[ppo]] / [[policy-optimization-methods]] — VARL is implemented on GRPO; the contribution is the reward, not the optimizer.

## References

- [arXiv:2607.01181](https://arxiv.org/abs/2607.01181)
- Original source text: `sources/right-in-the-right-way-damani-2026.md`
