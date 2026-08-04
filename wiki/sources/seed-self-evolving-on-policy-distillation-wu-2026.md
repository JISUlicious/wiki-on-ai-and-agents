---
title: "SEED: Self-Evolving On-Policy Distillation for Agentic Reinforcement Learning"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - seed-self-evolving-on-policy-distillation-wu-2026.md
arxiv_id: "2607.14777"
year: 2026
authors:
  - Jinyang Wu
  - Shuo Yang
  - Zhengxi Lu
  - Fan Zhang
  - Jianhua Tao
tags:
  - 2026
  - paper
status: complete
importance: high
---

# SEED: Self-Evolving On-Policy Distillation for Agentic Reinforcement Learning

**Jinyang Wu, Shuo Yang, Zhengxi Lu et al.** (Tsinghua University; Zhejiang University; CUHK; NTU; Tongji University) — [arXiv:2607.14777](https://arxiv.org/abs/2607.14777). v1, 16 Jul 2026, cs.CL. Code at `jinyangwu/SEED`.

## Summary

SEED targets the **supervision gap** in agentic RL: outcome-based rewards are sparse, delayed and assigned at the trajectory level, so they say whether an episode succeeded but not which intermediate observations, actions or tool calls deserve credit. The observation SEED exploits is that a *completed* trajectory contains rich hindsight information — reusable workflows, decisive observations, failure causes — that simply was not available at the intermediate decision steps. SEED converts this into natural-language **hindsight skills** and then distills their *behavioral effect* back into the policy, so that at inference nothing extra is needed.

The mechanism is a self-teacher trick. Stage 1 (**hindsight-skill SFT**) fine-tunes the backbone to read a serialized completed trajectory and emit a hindsight skill; the skill annotations for this bootstrap come from an *external* analyzer (GLM-5.2) over 1,440 offline trajectories (180 tasks × 8 rollouts). Stage 2 (**self-evolving on-policy distillation**) drops the external teacher: the frozen current policy `π_θold` both collects the rollout group *and* plays analyzer, producing a hindsight skill per trajectory. The **same** trainable policy then re-scores the **same sampled action tokens** twice — once under the ordinary interaction history (student branch) and once under a skill-augmented history (teacher branch). The detached log-probability shift `Δ = sg[ℓ_skill − ℓ_θ]` is passed through a sigmoid **confidence gate** `g = σ(β·Δ)` (following SDAR), and the OPD loss is gradient-equivalent to a gate-weighted negative log-likelihood: it raises the ordinary policy's likelihood on exactly the tokens the hindsight skill endorses. Positive shift → stronger auxiliary supervision on that token; negative shift → attenuated.

Two properties matter. First, this is **auxiliary, not a replacement**: `L_SEED = L_rl + λ_opd · L_opd`, with `L_rl` a clipped GRPO objective carrying the ordinary trajectory-level group-relative advantage broadcast to all valid tokens. The dense token-level signal sits *on top of* outcome RL rather than displacing it. Second, because actor and analyzer share parameters and are refreshed each update, **the experience distribution and its hindsight supervision evolve together** — that is the "self-evolving" claim, contrasted against static skill libraries and fixed external teachers. At deployment SEED "requires no analyzer, no skill bank, no retrieval module, and no augmented decision prompt" — a notable contrast with retrieval-based skill systems.

## Key points

- **The headline number checks out.** Against GRPO (outcome-only RL) SEED improves the **ALFWorld macro-average by 14.9–45.9 points** across the three backbones. Decomposed: Qwen2.5-7B 81.2 → **96.1** (+14.9); Qwen2.5-3B 75.0 → **91.8** (+16.8); Qwen3-1.7B **46.1 → 92.0 (+45.9)**. The 45.9 end of the range is a single small backbone where GRPO simply fails to get off the ground, so the range is driven by that outlier rather than being uniform.
- Other margins over GRPO: Search-based QA **+1.4 to +9.3**, WebShop task-completion score **+8.7 to +19.8**, WebShop success rate **+5.5 to +39.0**.
- **ALFWorld per-family under Qwen2.5-3B**: SEED scores 100.0 on Pick, Look, Clean and Heat, 70.6 Cool, 80.0 Pick2. Under 7B: 100.0 on Pick, Look, Cool and Pick2; 96.3 Clean, 80.0 Heat.
- **SEED is not uniformly best.** On Search-based QA at 7B it reaches 48.6 vs. RLSD and SDAR at 49.0. The paper's own framing is honest about this: best or tied-best in **10 of 12** aggregate comparisons, and it beats Skill-GRPO* (which gets privileged skill context at test time) in **11 of 12** — while itself using no skill context at evaluation.
- **Internalizing beats prompting.** Skill-Prompt, which injects skills only at evaluation, is *worse than the vanilla backbone* on several aggregates (e.g. Search QA 31.7 → 23.9 at 3B; ALFWorld 12.5 → 9.4 at Qwen3-1.7B). Skills supplied as context are not the same as skills distilled into weights.
- **Ablations** (ALFWorld avg., full SEED = 91.8): removing **on-policy skills** and substituting a static offline skill library is the most damaging → **84.4** (−7.4); removing **hindsight-skill SFT** → **86.0** (−5.8); removing **self-evolving OPD** (keeping only the one-time SFT skill supervision) → **87.0** (−4.8). The ordering is the paper's core claim in miniature: freshness of the supervision matters more than either of the other two pieces.
- **Sample efficiency**: with **60%** of the training instances SEED scores **80.7**, above GRPO's **75.0** on the *full* set; with 40% it reaches 58.9, matching GRPO at 80% (58.6).
- **Training dynamics** (3B, ALFWorld): the curves separate early — by step 40 SEED is at ~57% while GRPO is near 35%. Mean episode length falls from ~28 turns to **13** for SEED vs. ~16 for GRPO, and since shorter episodes coincide with higher success this reads as efficiency, not premature termination.
- **Generalization**: on the ALFWorld *unseen* split the 3B checkpoint goes 70.9 → **86.2** macro-average (**+15.3** over GRPO), winning five of six families — largest gains Heat **+35.0**, Look +18.3, Pick +16.5, with Clean the one regression (**−2.9**).
- **Multimodal extension** (Qwen2.5-VL-3B): Sokoban 6×6 67.1 → **82.0** (+14.9), EZPoints 86.9 → **100.0** (+13.1), average 77.0 → **91.0**. ReAct prompting manages only 7.4% average, so these are environments where policy learning is required rather than optional.
- **Setup**: 150 policy updates, rollout group N = 8, batch 16 on ALFWorld/WebShop and 128 on Search-based QA; WebShop uses the standard 128 test tasks; Search-based QA follows the Search-R1 protocol over NQ, TriviaQA, PopQA, HotpotQA, 2WikiMultiHopQA, MuSiQue, Bamboogle.

## Concepts & entities

- [[outcome-based-reward]] — the explicit foil. GRPO's terminal reward is "broadcast to all valid tokens"; SEED's contribution is a *token-level* signal layered on top, and every headline number is a delta over that outcome-only baseline. Note it is an addition to outcome RL, not a replacement for it.
- [[agentic-rl]] — long-horizon multi-turn RL over ALFWorld / WebShop / search-QA, formalized as a POMDP with sparse episode-level `R(τ)`.
- [[skill-acquisition]] — skills are acquired from the agent's *own* completed trajectories by the agent itself acting as analyzer; successful trajectories yield reusable workflows, failed ones yield corrective/avoidance rules.
- [[skill-optimization]] — the self-evolving loop means the skill supervision is regenerated each update to track the policy's current failure modes; the static-library ablation quantifies what that freshness is worth (7.4 points).
- [[agent-skills]] — a sharp contrast with skill-library architectures: SEED's skills exist only at training time and leave no artifact at deployment.
- [[self-improving-agent]] — actor and analyzer are the same weights, so improving the policy improves the supervision that improves the policy.
- [[reflexion]] — same hindsight-from-a-finished-episode intuition, but here the reflection is converted into a gradient signal on token log-probabilities rather than re-injected as context.
- [[agent-evaluation]] — evaluated on ALFWorld (six task families plus an unseen split), WebShop (128 tasks), seven search-QA datasets, and vision-based Sokoban/EZPoints.

## References

- [arXiv:2607.14777](https://arxiv.org/abs/2607.14777)
- Source text: [[seed-self-evolving-on-policy-distillation-wu-2026]]
- Code: `https://github.com/jinyangwu/SEED`
