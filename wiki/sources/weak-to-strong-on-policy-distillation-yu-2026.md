---
title: Weak-to-Strong On-Policy Distillation
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2607.26246"
venue: Technical Report
authors:
  - Fangxu Yu
  - Weijia Xu
  - Michael Xu
  - Tianyi Zhou
  - Zinan Lin
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Weak-to-Strong On-Policy Distillation

**Fangxu Yu, Weijia Xu, Michael Xu, Tianyi Zhou, Zinan Lin** (University of Maryland College Park; [[microsoft-research|Microsoft Research]]; MBZUAI) — [arXiv:2607.26246](https://arxiv.org/abs/2607.26246) v2, cs.LG, 28 Jul 2026 (rev. 3 Aug 2026). Technical Report. Code: `github.com/Yu-Fangxu/W2S-OPD`.

## The problem

[[on-policy-distillation|On-policy distillation]] (OPD) supervises a student with a teacher's token-level distribution **on the student's own rollouts**, combining RL's on-policy credit assignment with distillation's dense signal. Its premise is a teacher at least as capable as the student, and current practice satisfies that premise two ways, both of which break:

1. **Strong-to-weak** — distill a larger model into a smaller one (Qwen3 distills 235B into 8B/14B). At the frontier there is no larger teacher, so the ceiling is capped.
2. **Same-origin consolidation** — train several domain experts from the student's base by RL and merge them back (MOPD; DeepSeek-V4 at trillion scale). This requires training experts *at the student's scale*, which is expensive.

Naively pointing OPD at a *weaker* teacher fails twice over: the weak model's distribution is far from the student's, which weakens the signal, and distilling toward it **bounds the student at the teacher's level** while eroding general capability.

## The method

The reframing is the contribution: **stop imitating the weak supervisor, and instead isolate the capability *direction* that separates two weak models.** What the two weak models agree on is their shared limitation; subtracting their logits cancels that common component and leaves only the direction along which the positive improves on the negative — a signal the paper argues is largely disentangled from scale, and therefore transferable upward.

That direction is then **re-anchored onto the student's own base model**, producing a proxy teacher that carries the capability while staying distributionally adjacent to the student (borrowing the construction from decoding-time-experts / contrastive decoding):

```
π_T,α(· | s_t) = softmax( z_base(s_t) + α·( z⁺(s_t) − z⁻(s_t) ) )
```

The student minimizes per-token **reverse KL** to this proxy teacher on its own rollouts (Top-k for efficiency):

```
L_W2S-OPD(π_S) = E_{x∼D, y∼π_S} [ Σ_t KL( π_S(·|s_t) ‖ π_T,α(·|s_t) ) ]
```

`α ≥ 0` trades signal strength against distributional proximity. All three source models are **frozen**; only their logits are needed. Multi-teacher is a trivial extension: sum the K direction terms with per-direction coefficients α_k (setting α_k ∈ {0,1} routes each query to its matching expert pair).

**Three instantiations of the contrast pair** (all Qwen3; student and anchor are Qwen3-8B base, non-thinking mode):

| Setting | Positive m⁺ | Negative m⁻ | What the difference isolates |
|---|---|---|---|
| Pre-RL / Post-RL | 4B-RL domain expert | 4B pre-RL init | the skill RL instills — *without ever running RL at 8B* |
| Smaller / Larger | 4B base | 0.6B base | the capability that emerges from scale — **free, no training at all** |
| Correct / Wrong Hints | 4B conditioned on a correct hint | same 4B on a wrong hint | the instance-level direction toward the solution; shared conditioning cancels the style shift |

**Setup.** Student Qwen3-8B; both W2S-OPD and OPD trained 100 steps under identical config. The RL experts are GRPO on Qwen3-4B — 500 steps on the level-6 subset of DeepMath-103K (math), 300 steps on Eurus-RL-Code (code). Evaluation at temperature 1.0, top-p 1.0; 32 samples/problem on math, 4 on code; Math-Verify for answer checking. Benchmarks: [[aime|AIME]]24, AIME25, HMMT25 (Feb.), HMMT25 (Nov.) for math; HumanEval+, MBPP+, LiveCodeBench-V6 for code; [[gpqa|GPQA]]-Diamond and IFBench for out-of-domain.

## Results

### Pre-RL / Post-RL contrast

| Method | AIME24 | AIME25 | HMMT25 Feb | HMMT25 Nov | **Math Avg** | HumanEval+ | MBPP+ | LCB | **Code Avg** |
|---|---|---|---|---|---|---|---|---|---|
| Neg. model (4B) | 21.6 | 19.9 | 10.0 | 8.2 | 14.9 | 79.9 | 63.6 | 20.9 | 54.8 |
| **Pos. model (4B-RL)** | 62.0 | 57.9 | 34.8 | 40.4 | **48.8** | 86.3 | 71.0 | 27.1 | **61.5** |
| Student base (8B) | 26.1 | 21.1 | 11.6 | 9.0 | 17.0 | 81.2 | 70.0 | 21.7 | 57.6 |
| *Single-teacher:* SFT | 60.2 | 54.4 | 30.7 | 37.6 | 45.7 | 82.5 | 71.8 | 24.8 | 59.7 |
| *Single-teacher:* OPD | 62.1 | 54.8 | 30.6 | 38.3 | 46.5 | 83.1 | 71.2 | 22.0 | 58.7 |
| *Single-teacher:* **W2S-OPD** | **68.9** | **60.1** | **35.1** | **43.1** | **51.8** | 84.8 | 72.4 | 26.8 | 60.9 |
| Δ vs OPD | +6.8 | +5.3 | +4.5 | +4.8 | **+5.3 (11.4% rel.)** | +1.7 | +1.2 | +5.0 | **+2.2 (3.7% rel.)** |
| *Multi-teacher:* OPD | 64.3 | 54.9 | 29.6 | 37.4 | 46.5 | 83.4 | 69.5 | 25.6 | 59.5 |
| *Multi-teacher:* **W2S-OPD** | 67.4 | **61.0** | 34.7 | **45.2** | **52.1** | 85.2 | 71.0 | 27.1 | 61.1 |
| Δ vs OPD | +3.1 | +6.1 | +5.1 | +7.8 | **+5.6 (12.0% rel.)** | +1.8 | +1.5 | +1.5 | +1.6 |

On math the student ends at **51.8 vs. the 4B expert's 48.8** — it passes the teacher it learned from, while plain OPD (46.5) stays below it. On code W2S-OPD narrows but does not close the gap (60.9 vs. 61.5). The multi-teacher run merges a math expert and a code expert into one student in a single distillation pass.

### Purely-weaker sources

**Smaller / Larger** (Qwen3-4B minus Qwen3-0.6B, both off-the-shelf, *no training whatsoever*, both weaker than the student on every benchmark):

| Method | Math Avg | Code Avg |
|---|---|---|
| Neg. model (0.6B) | 1.7 | 18.8 |
| Pos. model (4B) | 14.9 | 54.8 |
| Student base (8B) | 17.0 | 57.6 |
| **W2S-OPD** | **23.0 (+6.0)** | **58.8 (+1.2)** |

**Correct / Wrong Hints** (one 4B model, two conditionings):

| Method | Math Avg | Code Avg |
|---|---|---|
| Pos./Neg. model (4B) | 14.9 | 54.8 |
| Student base (8B) | 17.0 | 57.6 |
| **W2S-OPD** | **18.4 (+1.4)** | **58.7 (+1.1)** |

These two settings are the load-bearing result: **every supervision source is strictly weaker than the student, and the student still improves.** The gains are much smaller than in the RL-expert setting (+6.0 and +1.4 on math vs. +5.3 *over an already-strong OPD baseline*), but they are positive.

The gain scales with the **capability gap inside the pair**: fixing the 8B student and 4B positive, swapping the negative from 0.6B to 1.7B drops the math gain from **+6.0 to +4.7** — "instantiat[e] W2S-OPD with the most separated base-model pair available."

### Out-of-domain (trained on math only)

| Method | GPQA-Diamond | IFBench |
|---|---|---|
| Positive model (4B-RL) | 51.8 | 26.0 |
| Student base (8B) | 38.9 | 26.3 |
| OPD | 54.4 | **25.9 (below base)** |
| **W2S-OPD** | **56.5** | **27.0** |

This is the cleanest demonstration of the "directional not imitative" claim. On GPQA-Diamond W2S-OPD lifts the student **38.9 → 56.5**, above both OPD and the 4B expert. On IFBench, OPD **degrades the student below its own initialization** (26.3 → 25.9) — "consistent with the student absorbing the small expert's limitations" — while W2S-OPD improves it to 27.0.

### α, cost, and what gets reinforced

- **α is a real knob but a forgiving one.** Small α leaves the proxy teacher ≈ the base model, the signal is too weak, and W2S-OPD *falls behind* OPD. Large α distorts the distribution and pushes the teacher away from the student, and performance declines on most benchmarks. W2S-OPD beats OPD "over a wide range of α," with a moderate value best.
- **Cost is modest.** 1043 s vs. 868 s per training step — **+20% over OPD** despite forwarding three frozen models (8B anchor + the 4B pair) instead of one 4B teacher, because per-step cost is dominated by the shared on-policy rollout generation and the extra scoring passes are forward-only and parallelizable.
- **Different contrasts reinforce different tokens.** Taking the top-1% highest Δ_t = log π⁺(x_t) − log π⁻(x_t) tokens over 200 math traces, labeled into the eight Schoenfeld problem-solving episodes by ThinkARM:

| Contrast | Read | Analyze | Plan | Implement | Explore | Verify | Monitor | Answer |
|---|---|---|---|---|---|---|---|---|
| *All tokens (base rate)* | 10.3 | 33.2 | 8.6 | 40.7 | 0.5 | 2.4 | 0.5 | 3.8 |
| Pre-RL / Post-RL | 9.6 | 40.5 | **15.7** | 24.0 | 1.2 | 4.3 | **1.8** | 2.9 |
| Smaller / Larger | 6.5 | **44.4** | 12.6 | **27.7** | 0.8 | 4.2 | 0.9 | 2.9 |
| Correct / Wrong Hints | 5.2 | 29.8 | **20.2** | 19.6 | 1.1 | **5.4** | **2.3** | **16.4** |

Read against the base rate: the RL and hint contrasts up-weight **Plan** and **Monitor** — the reasoning *framework* — while the scale contrast keeps more mass on **Analyze/Implement**, the solving procedure. The hint contrast's 16.4% on **Answer** (vs. 3.8% base rate) is a direct artifact of conditioning on a correct versus a wrong answer.

## Where it breaks

There is no explicit limitations section; the Discussion frames the open questions.

- **Saturation is unknown.** "How far [weak supervision sources] can push a stronger student before weak-to-strong supervision saturates... remain[s] to be explored." All runs are 100 steps.
- **The purely-weak gains are small.** +1.4 math points in the hint setting; +1.2 code points in the scale setting. The large numbers all come from the pre-RL/post-RL setting, where the positive model is a domain expert that is *in-domain stronger* than the student (48.8 vs. 17.0 on math).
- **One student, one family, one scale.** Everything is Qwen3, student fixed at 8B, contrast models at 0.6B–4B. Nothing tests whether the "capability direction is scale-disentangled" premise holds across a wider gap or across model families — and cross-family use would require shared tokenizer/vocabulary for logit arithmetic to be defined at all.
- **Two domains.** Math and code, both verifiable. The out-of-domain check is two benchmarks.
- **α needs tuning per setting**, and at low α the method is worse than the baseline it replaces.
- **Code is the weaker story**: W2S-OPD never surpasses the code teacher (60.9 vs. 61.5), only math.

> [!note] Digest qualifier
> The framing "a strong student distilling from *multiple demonstrably weaker* teachers" merges two distinct results. In the **scale** and **hint** settings every source really is weaker than the student on every benchmark. In the headline **pre-RL/post-RL** setting the positive model is a small *domain expert* that beats the 8B student badly in-domain; what is weak there is its *general* capability and its parameter count, not its math skill. Both results are real; only the first two are "every supervision source is weaker."

## Related

- [[knowledge-distillation]] — the direct parent; this inverts its usual capability ordering
- [[grpo]] · [[reinforcement-learning]] · [[outcome-based-reward]] · [[verifier]] — RLVR is the alternative paradigm being combined with, and the source of the post-RL expert
- [[reasoning]] · [[chain-of-thought]] — the transferred capability, analyzed at the level of reasoning episodes
- [[fine-tuning]] · [[instruction-tuning]] — SFT is the weakest baseline in every table
- [[rlhf]] · [[dpo]] — post-training alternatives; Appendix B recasts the W2S-OPD objective as reward maximization under a KL trust region
- [[steering-vectors]] · [[activation-addition]] · [[contrastive-activation-addition]] — closest conceptual neighbors: a difference between two models/conditionings isolates a direction which is then added elsewhere. W2S-OPD does this in **logit** space across models, and then *distills* the result rather than using it at decoding time.
- [[self-improving-agent]] — the broader agenda: frontier models improving from supervision cheaper and weaker than themselves
- [[code-generation]] · [[humaneval]] — evaluation
- [[microsoft-research]]

## References

- [arXiv:2607.26246](https://arxiv.org/abs/2607.26246)
