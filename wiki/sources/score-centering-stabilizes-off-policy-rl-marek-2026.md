---
title: "Score Centering Stabilizes Off-policy Reinforcement Learning"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - score-centering-stabilizes-off-policy-rl-marek-2026.md
arxiv_id: "2609.20807"
authors:
  - Martin Marek
  - Max Ryabinin
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Score Centering Stabilizes Off-policy Reinforcement Learning

**Martin Marek, Max Ryabinin** (Together AI). [arXiv:2609.20807](https://arxiv.org/abs/2609.20807), cs.LG, 17 Sep 2026. Preprint with no venue given. Code: github.com/martin-marek/score-centering.

## Summary

This paper explains why LLM [[reinforcement-learning]] is fragile under **training–inference mismatch (TIM)**, meaning a sampler q that differs from the trainer p through kernels, quantization or staleness. By the covariance identity, the per-prefix policy-gradient update splits into **drift** (E_q[R]·s̄, where s̄ is the expected score under the sampler) and **signal** (Cov_q(R, s)). On-policy s̄ = 0, but under TIM drift acts as *distillation of the trainer toward the sampler*. The sampler is a biased copy of the trainer that gets re-synced every step, so the bias compounds in a feedback loop instead of converging. That is why offline SFT on another model's data is stable while online RL breaks on tiny numerical differences: "what matters is not the size of the mismatch, but whether the teacher is fixed or tracks the student". [[grpo|Group centering]] shrinks drift but does not remove it, because expected advantage *at a prefix* is not zero.

**Score centering (SC)** subtracts s̄ from every token's score. It is additive, deterministic given the prefix, has no ratios, clipping or hyperparameters, and does nothing when p = q. In practice it logs the sampler's top-k = 128 logprobs and models the tail with the rescaled trainer distribution; k = 32 already matches exact SC. Because SC is additive, it composes with importance sampling (IS).

## Key points

- **Toy diagnosis (Qwen3-1.7B, Countdown, Fig. 2).** Offline training is stable only with +1/0 rewards, while online training under TIM is *least* stable with +1/0 and most stable with group-centered rewards. The authors take this to mean two separate instability mechanisms.
- **Weight-noise sweep (Qwen3-0.6B, Fig. 3).** Under Δθ ~ N(0, σ²) with σ = 0.01/0.02/0.05, the methods that collapse do so earlier as noise grows. DPPO collapses at about step 160/80/20, and TIS is stable at the smallest noise but collapses at about step 180 and 40 at the larger two. At σ = 0.05, only SC and SC composed with TIS or MIS train stably.
- **INT8 sampler and staleness (Fig. 4).** Under quantization, SC alone or composed performs best. Under severe staleness (the sampler is updated only every 64 steps), **SC + TIS/MIS beats plain SC**, because SC measures the covariance under the sampler rather than the trainer. PPO and DAPO survive staleness but collapse under numerical error, since their clip was designed for ratios caused by policy movement.
- **30B MoE (Qwen3-30B-A3B-Base, INTELLECT-2 math, Fig. 1).** With an FP8 sampler, even plain PG is stable (58% train accuracy). With an FP4 KV cache, PG collapses within 200 steps and MIS collapses late, while SC reaches 52% and TIS 51%. With INT8 W/A and INT4 KV, **SC reaches 30%, TIS 12%, and every other method ends below 5%**. Router indices were *not* replayed.
- **Shared-objective comparison.** Every correction sits on the same REINFORCE plus group-centered objective with one SGD step per batch, so DAPO, IcePop and others are tested as isolated corrections, not as the full recipes.

> [!note] Abstract vs body
> The abstract says SC "matches or outperforms" IS methods. In the body, under severe staleness, plain SC is *beaten* by SC + TIS/MIS (§5.3), and the authors recommend the composition whenever the trainer can move far from the sampler. The abstract only hints at this ("composition outperforms pure importance sampling baselines in our staleness experiments").

## Bearing on the ClawGym II / Agent Lightning stitching dispute

This paper bears on the dispute **indirectly and does not settle it**. [[clawgym-ii-black-box-rl-agent-harness-song-2026|ClawGym II]] and [[agent-lightning-v1-harnessed-agentic-rl-he-2026|Agent Lightning v1.0]] disagree about a *context* mismatch: whether the prompt a stitched turn is trained under differs from the prompt it was sampled under. This paper studies only *parameter-side* mismatch (weight noise, quantization, staleness) on identical prefixes, and it never measures a context mismatch. It does add two points. (1) Its drift mechanism predicts that even a *small* systematic mismatch compounds over long runs, so "the mismatch is tiny" is no defence. (2) ClawGym II's truncated ratio `min(exp(log π_old − log π_rollout), c̄)` is a TIS-family correction, and this paper shows that truncation reintroduces drift and that TIS collapses under severe mismatch where SC does not. The ClawGym page's inference that the truncated ratio "would partly absorb a context mismatch" should therefore be read as *partial at best*. Formally, SC could cancel drift toward any logged sampler distribution, but that is an inference not tested here.

## Concepts & entities

- [[reinforcement-learning]] · [[grpo]] · [[ppo]] · [[agentic-rl]] — the objective family analysed.
- [[quantization]] — FP8/FP4/INT8 samplers are the realistic TIM sources.
- [[mixture-of-experts]] — the 30B run is MoE, and routing mismatch is left uncorrected.
- [[mimo-v2-6-scaling-reinforcement-learning-self-improvement-xiaomi-2026|MiMo-V2.6]] and [[molt-pytorch-native-agentic-rl-framework-hu-2026|Molt]] are engineering-side responses to the same mismatch (routing replay, top-p replay). This paper is the algorithmic alternative.

## What it does not show

- Headline results use *deliberately severe* mismatch on short sequences as a proxy for long runs under mild TIM. The authors admit they could not find natural setups that separate methods within a few GPU-hours.
- There are no downstream evaluation benchmarks at 30B, only training accuracy, and no multi-turn or agentic setting.
- The seed counts are uneven (Appendix B). The 0.6B/1.7B runs use 3 seeds per method. At 30B the FP8 panel is single-seed for everything, and in the FP4/INT4 panels SC uses 5 and 4 seeds, TIS 4 and 3, and most other methods 1.

## References

- [arXiv:2609.20807](https://arxiv.org/abs/2609.20807)
