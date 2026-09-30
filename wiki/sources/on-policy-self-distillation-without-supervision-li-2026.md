---
title: On-Policy Self-Distillation without Any Supervision
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.06296"
authors:
  - Yijiang Li
  - Bingyang Wang
  - Yijun Liang
  - Yunjie Tian
  - Di Fu
  - Nuno Vasconcelos
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# On-Policy Self-Distillation without Any Supervision

**Yijiang Li, Bingyang Wang, Yijun Liang, Yunjie Tian, Di Fu, Nuno Vasconcelos** (UC San Diego; Georgia Institute of Technology; University of Maryland College Park; [[bytedance|ByteDance]]) — [arXiv:2608.06296](https://arxiv.org/abs/2608.06296) v2, cs.LG, 9 Aug 2026. CC BY 4.0. Code: `github.com/williamium3000/u-opsd`.

## The problem

On-policy self-distillation (OPSD) already removes the *stronger teacher*: one model plays both roles, with the teacher conditioned on the ground-truth solution `y*` and the student seeing only the problem. But the paper's framing is sharp — that is "self"-distillation only in the sense that teacher and student **share parameters**; the information that makes the teacher better still comes from outside. The same holds for every OPD variant: [[grpo|GRPO]]/RLVR needs gold answers to verify, OPD needs a stronger model, OPSD needs reference solutions, demonstrations, or environment feedback. The question: *can a model construct its own privileged context?*

## The method

**u-OPSD** answers with [[self-consistency|self-consistency]]. A single rollout is unreliable; **agreement across independently sampled rollouts is an endogenous confidence signal**, and it can supply both the teacher's privileged context and the student's training trajectories.

**(1) Sample.** For each unlabeled prompt, draw `G` rollouts from a stop-gradient copy `π̄` of the current policy at training temperature; parse the boxed final answer with normalization/canonicalization. Unparsable rollouts (usually truncations) are marked **invalid** and excluded from both sets — "an incomplete generation provides evidence of neither a correct nor an incorrect belief."

**(2) Vote.** The pseudo-answer `ã(x)` is the plurality of valid answers (ties broken uniformly at random). Rollouts partition into agreeing `Y⁺` and disagreeing `Y⁻`. Self-consistency score `c(x)` is normalized by **G**, not by the valid count — so truncations actively lower confidence. Prompts with `c(x) < τ` contribute no gradient.

**(3) Distill.** The **longest agreeing rollout** `y⁺` replaces `y*` as the teacher's reference; the disagreeing rollouts are the distillation targets:

```
L_u-OPSD(θ) = (1/|B⁻|) Σ_{y⁻∈B⁻} Σ_t D_β( π̄(· | x, y⁺, y⁻_<t)  ‖  π_θ(· | x, y⁻_<t) )
```

Two classes of prompt are skipped: `c(x) < τ` (vote not trustworthy) and `Y⁻ = ∅` (nothing to correct). What remains is exactly **the model's competence frontier** — prompts where it can name a plausible answer yet still puts substantial mass on conflicting trajectories. The authors note this yields a curriculum "directly from the model's own voting statistics, without external difficulty labels or scheduling."

The key distinction from ordinary majority-vote self-training: u-OPSD does not imitate the winning response under teacher-forced prefixes. It transfers the **solution-conditioned next-token distribution along the model's own disagreeing trajectories** — dense corrective supervision precisely where the model is confidently going wrong.

**Setup.** Four Qwen3 variants: 4B and 8B (each in thinking and non-thinking mode), plus Qwen3-4B-Instruct-2507 and Qwen3-30B-A3B-Instruct-2507. Training data is a 30k subset of OpenThoughts, **problem statements only — the solution field is never read**. Benchmarks: [[aime|AIME]]24, AIME25, HMMT25 at avg@12; MATH500, AMC23 at avg@4; vLLM decoding at temperature 1.0, 38k max generation. Following OPSD: forward KL (β=0) over the full vocabulary with per-token pointwise clipping, teacher **frozen at the initial policy**, LoRA r=64 (α=128) on all attention and MLP projections, LR 5e-6, 150 steps with checkpoints every 25, best checkpoint reported. u-OPSD-specific: **G=8, τ=0.5**, and completion length raised from 1,024 to 4,096 tokens (rollouts must reach a boxed answer). All OPSD numbers are the authors' own rerun under matched conditions.

> [!note] "%" in this paper means percentage points
> Every gain quoted below and in the paper's own text is an **absolute** difference in benchmark average, written with a `%` sign. E.g. "improves by 8.5%" = 40.96 → 49.49.

## Results

### Non-thinking mode (Table 1)

| Model | Method | AIME24 | AIME25 | HMMT25 | MATH500 | AMC23 | **Avg** |
|---|---|---|---|---|---|---|---|
| Qwen3-4B | Base | 25.83 | 17.78 | 10.83 | 84.10 | 66.25 | 40.96 |
| | *w/ GT:* SFT | 26.67 | 19.72 | 13.06 | 84.85 | 69.38 | 42.73 |
| | *w/ GT:* GRPO | 25.00 | 22.50 | 15.00 | 86.20 | 80.62 | 45.86 |
| | *w/ GT:* OPSD | 32.22 | 20.83 | 16.39 | 85.75 | 76.25 | 46.29 |
| | *w/o GT:* TTRL | 25.00 | 20.83 | 11.94 | 83.60 | 67.50 | 41.77 |
| | *w/o GT:* RENT | 22.22 | 20.28 | 11.39 | 84.00 | 74.38 | 42.45 |
| | *w/o GT:* Intuitor | 23.89 | 20.00 | 11.67 | 83.70 | 70.62 | 41.98 |
| | ***w/o GT:* u-OPSD** | **37.50** | **27.78** | 14.44 | **86.50** | **81.25** | **49.49** |
| Qwen3-8B | Base | 27.50 | 23.33 | 13.61 | 84.05 | 69.38 | 43.57 |
| | *w/ GT:* SFT | 26.94 | 21.67 | 11.94 | 84.10 | 72.50 | 43.43 |
| | *w/ GT:* GRPO | 30.56 | 21.94 | 13.06 | 87.85 | 73.75 | 45.43 |
| | *w/ GT:* OPSD | 41.67 | 28.06 | 18.33 | 87.15 | **85.00** | 52.04 |
| | *w/o GT:* TTRL / RENT / Intuitor | ≤28.33 | ≤22.50 | ≤13.06 | ≤84.45 | ≤73.12 | 42.97–43.54 |
| | ***w/o GT:* u-OPSD** | **45.56** | **34.72** | **18.61** | **89.55** | 83.12 | **54.31** |

**+8.5 and +10.7 over base**, and **+3.2 / +2.3 over supervised OPSD** — best on four of five benchmarks at both scales, with no gold labels anywhere. The label-free RL baselines (TTRL, RENT, Intuitor) gain at most 1.5 under the same rollout budget: **consensus is far more useful as a conditioning context for token-level distillation than as a scalar reward for policy optimization** (+7.0 to +11.3 over those methods in non-thinking mode).

### Thinking mode (Table 2) — the gains largely evaporate

| Model | Base | SFT | GRPO | OPSD | TTRL | RENT | Intuitor | **u-OPSD** |
|---|---|---|---|---|---|---|---|---|
| Qwen3-4B | 74.85 | 74.73 | 76.35 | 76.20 | 75.76 | 75.65 | 76.07 | **77.05** |
| Qwen3-8B | 76.09 | 76.16 | 76.92 | 77.97 | 76.95 | 77.16 | 76.64 | **77.99** |

**+2.2 / +1.9 over base**; **+0.9 over OPSD at 4B and +0.02 (a tie) at 8B**; **+0.7 / +1.1 over GRPO.** The paper's own explanation: consensus self-distillation works "when the base model is sufficiently reliable to produce informative consensus while retaining substantial room for improvement," and thinking-mode models at 74.9/76.1 have little headroom left.

### Instruction-tuned models (Table 3)

| Model | Base | OPSD | **u-OPSD** |
|---|---|---|---|
| Qwen3-30B-A3B-Instruct-2507 | 75.77 | 76.33 | **77.46** |
| Qwen3-4B-Instruct-2507 | 67.00 | 67.10 | **68.78** |

The 30B MoE result matters most: the recipe transfers to a substantially larger [[mixture-of-experts|mixture-of-experts]] model **without model-specific hyperparameter tuning**. Note OPSD still leads on HMMT25, MATH500, and AMC23 for the 4B-Instruct model — u-OPSD wins on average, not everywhere.

### Ablations (all Qwen3-8B non-thinking)

**Pseudo-label quality** (probe on 64 training prompts at G=8, 4096-token budget, τ=0.5): **96.3%** of rollouts yield a parsable boxed answer, **94.0%** of prompts clear the threshold, and **86.7%** of those pseudo-labels match the gold answer — i.e. **13.3% of the supervision is wrong**, and it still beats gold-supervised OPSD. Fewer than 10% of valid rollouts disagree with the vote, so the distillation set is small and concentrated.

**The defaults are not optimal, and the paper says so:**

| Knob | Finding |
|---|---|
| Threshold τ | Monotone in favor of looser: **58.59 at τ=0.3**, 57.10 at the default τ=0.5, **44.40 at τ=0.9** — a 14.2 spread. "The default τ=0.5 is therefore not the best setting in this sweep." |
| Rollouts G | Saturating: G=4 (56.99) ≈ G=8 (57.10); **G=12 gains 4.7** over default; G=16 gives back half. G=8 is kept only to cost-match OPSD. |
| Teacher update | Frozen teacher is OPSD's setting; **EMA decay 0.995 gains 2.4 at best checkpoint and 4.1 at step 150.** |

**Teacher reference and distillation target** — both axes favor the **longest** rollout (best setting 59.00: teacher on longest agreeing, distill longest disagreeing), but seven of nine configurations lie within 2.1 of the best. The load-bearing result is the negative one: **stripping the teacher's reference down to just the boxed pseudo-label costs 10.3–15.8 and puts all three label-only variants below the base model's 43.57.** The teacher must condition on the full reasoning trace, not the answer.

**Divergence computation** (best checkpoint):

| Variant | AIME24 | AIME25 | HMMT25 | MATH500 | AMC23 | Avg |
|---|---|---|---|---|---|---|
| Student token only | 29.44 | 19.72 | 12.78 | 84.70 | 70.62 | 43.45 |
| Full vocabulary (default) | 53.89 | 37.50 | 20.00 | 89.75 | 84.38 | 57.10 |
| top-k = 20 | 49.17 | 37.78 | 19.44 | 90.20 | 88.12 | 56.94 |
| top-k = 50 | 48.06 | 34.44 | 24.17 | 89.45 | 83.12 | 55.85 |
| **top-k = 100** | 53.89 | **43.33** | 20.56 | **91.65** | 85.62 | **59.01** |
| top-k = 200 | 51.67 | 40.00 | 22.50 | 90.15 | 86.25 | 58.11 |

Sampled-token distillation is **13.7 below the default** and not competitive under pseudo-label supervision — the preference for full-distribution supervision is *stronger* under pseudo-labels than under gold (+17.8 on AIME25 vs. +2.0 under gold). top-k=100 beats even full vocabulary, so truncation is a free optimization.

**Divergence metric** — forward KL is not merely preferred, it is required. JSD (β=0.5) scores 43.34, level with the untrained model. **Reverse KL does not converge at all**, and the failure mode is documented precisely: generations average 2.7k characters at step 25, 12k at step 50, 76k at step 75, and 99k (the ceiling) from step 125, while parsable-answer rate falls **99% → 33%**. Inspection shows "a loss of termination rather than cohesive reasoning" — a phrase, a LaTeX command pair, or a bracket repeated until the budget runs out.

## Where it breaks

The paper has an unusually honest Limitations section.

- **Scope.** One model family (Qwen3, 4B/8B), one domain (competition math with automatically checkable answers). The voting mechanism **requires an extractable, canonicalizable answer**; open-ended generation would need a softer consensus than exact match.
- **The advantage is regime-dependent, and the authors say not to over-read it.** "Readers should therefore treat consensus as a stronger replacement for gold solutions in the non-thinking regime we measured and as a match for them in the thinking regime, rather than as a uniform improvement on on-policy self-distillation."
- **Bounded by base-model competence.** Majority vote reproduces whatever the model already says most often, so it is capped by that answer's accuracy. 13.3% of pseudo-labels were wrong in-domain, and training under deliberately corrupted votes was not measured. Vote-margin-based down-weighting is flagged as future work.
- **No seed-replicated error bars yet** — "pending and will be added in a revision." Variance is only mitigated by 12-sample evaluation across five benchmarks.
- **Comparison is cost-matched to OPSD, not optimized.** G=8, τ=0.5, and the frozen teacher are all chosen for comparability, and all three are beaten by other settings in the ablations.

## Related

- [[self-consistency]] — the mechanism, repurposed: majority vote becomes an **instance-specific privileged context** for a self-teacher rather than an inference-time aggregation rule
- [[knowledge-distillation]] — the parent; this removes the last external dependency (the reference solution) from on-policy self-distillation
- [[grpo]] · [[reinforcement-learning]] · [[outcome-based-reward]] · [[verifier]] — RLVR is the supervised comparison; the paper's finding is that consensus is worth much more as dense conditioning than as a scalar reward
- [[self-improving-agent]] · [[self-reflection]] — label-free self-evolution; the training set is automatically the model's competence frontier
- [[reasoning]] · [[chain-of-thought]] — the domain, and the reason the *full trace* (not the boxed answer) must be the teacher's context
- [[adaptive-thinking]] · [[reasoning-effort]] — the thinking/non-thinking split is where the entire size of the effect lives
- [[lora]] · [[fine-tuning]] · [[instruction-tuning]] — training setup and the instruct-model results
- [[mixture-of-experts]] — Qwen3-30B-A3B, where the recipe transfers untuned
- [[test-time-compute]] · [[inference-time-compute]] — G rollouts per prompt is the dominant cost term, growing linearly in G
- [[bytedance]]

## References

- [arXiv:2608.06296](https://arxiv.org/abs/2608.06296)
