---
title: "Understanding Reasoning from Pretraining to Post-Training"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - understanding-reasoning-pretraining-posttraining-shen-2026.md
arxiv_id: "2607.16097"
year: 2026
authors:
  - Jingyan Shen
  - Ang Li
  - Salman Rahman
  - Micah Goldblum
  - Pavel Izmailov
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Understanding Reasoning from Pretraining to Post-Training

**Jingyan Shen, Ang Li, Salman Rahman, Yifan Sun, Micah Goldblum, Matus Telgarsky, Pavel Izmailov** (New York University; Modal Labs; UCLA; UIUC; Columbia University) — [arXiv:2607.16097](https://arxiv.org/abs/2607.16097).

## Summary

RL post-training is usually studied in isolation from the [[pre-training]] that precedes it, leaving two questions open: how pretraining choices shape the returns to RL compute, and what RL actually does to the inherited policy. Both are hard to answer at LLM scale — pretraining corpora are uncontrolled and joint compute sweeps are prohibitively expensive. The authors therefore build a **controlled chess testbed** that mirrors the standard LLM pipeline: autoregressive pretraining on tokenized human Lichess games (54B-token corpus, 2022 Blitz/Rapid), SFT on *synthetic reasoning traces* (serialized game-continuation search trees, so the chain of thought is written in chess move tokens rather than natural language), then GRPO RL with a binary verifiable reward on Lichess puzzles. Models use the dense Qwen3 architecture at 10 sizes from 5M to 1B parameters; the paper sweeps 36 pretraining–RL combinations.

The headline result is a **joint pretraining–RL scaling law**. Rather than fitting the full sigmoid RL-compute curve of Khatri et al. (2025) — whose ceiling is unidentifiable without runs long enough to saturate — the authors take a first-order Taylor expansion of that sigmoid in the non-saturated regime, giving a law that is *log-linear in RL compute*. Two pretraining quantities drive it: the intercept (post-RL pass@1 at a reference RL compute) is predicted by **pretraining validation loss**, and the slope (pass@1 gain per decade of RL compute) grows approximately linearly with **log pretraining tokens**, with model size a weaker positive correction. Composing this with a [[chinchilla]]-style loss law `L(N,T)` lets them score hypothetical recipes `(N, T, C_RL)` without training them, and trace a compute-optimal frontier.

Mechanistically, RL is *not* just sharpening. Fitting `π_RL ∝ π_SFT^α` gives a global slope above 1 (sharpening on average) but only moderate R², with per-state slopes varying widely. Categorizing per-state policy updates shows three regimes whose mix depends on puzzle difficulty: **ground-truth amplification** dominates on easy puzzles, while on hard puzzles both **tail discovery** (a correct move promoted from below 0.05 probability into the top-k) and **wrong-mode amplification** (the correct move stays outside top-k while an already-preferred wrong move is reinforced) rise. This explains the familiar pattern of RL improving pass@1 while pass@k stays flat or degrades. A 1B OLMo-2 model pretrained on 200B math-domain tokens reproduces the same predictive structure outside chess.

## Key points

**Setup**
- Pretraining corpus: 54B tokens of human Lichess games; vocabulary |V| = 81 (each move = 4 tokens: piece, source, destination, flag).
- Post-training: 156K quality-filtered puzzles in 5 difficulty bins (B1–B5); eval benchmark of 1,480 tactical puzzles; the three datasets are disjoint at board-position level to prevent contamination.
- Model ladder: {5M, 10M, 20M, 32M, 50M, 100M, 200M, 410M, 680M, 1B}, Qwen3 dense architecture.
- Pre-RL sweep: 11 pretraining compute budgets from 6.5×10¹⁶ to 6.5×10¹⁹ FLOPs (≈200M to 52B tokens) across 10 sizes.
- RL: GRPO with binary outcome reward — reward 1 only if *every* move in the solution line matches ground truth. Frontier analysis runs 1000–5000 RL steps for sizes {20M, 50M, 200M, 680M}. Cost reference: 2000 RL steps ≈ 160 H200 GPU-hours for a 50M model.

**The joint pretraining–RL scaling law (actual functional form)**

Local log-linear RL fit (Eq. 1), a first-order Taylor expansion of the sigmoid RL law:

```
R_{N,T}(C) = R^ref_{N,T} + B_{N,T} (log10 C − log10 C_ref)
```

Fitted by OLS per run; C_ref fixed at 10²⁰ FLOPs. Each 10× of RL compute adds a constant increment `B_{N,T}` to reward.

Joint law, with both coefficients predicted from pretraining properties:

```
R(C_RL, N, T) = f(L_pt(N,T)) + g(N,T) · (log10 C_RL − log10 C_ref)

f(L)    = 0.0314 + exp(4.87 − 12.85 L)
g(N,T)  = 0.216 + 0.0172 log10 T + 0.0098 log10 N
L(N,T)  = 0.414 + 95.4 N^(−0.404) + 1.62×10^5 T^(−0.677)      (Chinchilla-style)
```

- Note `f` is explicitly **nonlinear** in pretraining loss (an exponential fit beat a linear one).
- The g-coefficient on tokens (0.0172) is ~1.75× the coefficient on parameters (0.0098) — RL improvement rate is shaped mainly by pretraining *data exposure*.

**Evidence for each claim**
| Claim | Evidence |
|---|---|
| Lower pretraining loss → higher post-RL performance | Spearman \|ρ\| between R^ref and pretraining validation loss rises from **0.93 → 0.99** as log10 C_ref goes 16 → 20; measured on B3–B4 (unsaturated) puzzles. Post-SFT pre-RL performance R₀ is *less* tightly predicted by pretraining loss than post-RL performance is. |
| Slope grows with pretraining tokens | Pearson **r = +0.84** between B_{N,T} and log10 T; linear-in-T fit R² = 0.70; joint (log T, log N) fit attains lowest RMSE and highest R², so it is adopted. |
| Optimal RL share rises with total compute | Empirical frontier for the 20M model: RL compute ratio increases **5% → 32%** along the frontier. Extrapolated frontier (13 sizes 20M–2B, 260 budgets 10¹⁷–10²¹ FLOPs, 400 candidate splits each): compute-optimal RL share ≈ **20% at 50M → 28% at 680M**, rising with total compute. The law-derived frontier closely matches the empirical one. |
| Chinchilla allocation still holds for the pretraining half | Ratio T*/D_opt(C_pt) at frontier points shows **no systematic deviation** from Chinchilla token allocation — RL changes the pretraining/RL *split*, not the parameter/token split within pretraining. |
| Math transfer | 1B OLMo-2 on 200B tokens (70% Nemotron-CC-Math-v1, 30% Dolma3), 14 checkpoints from 10B–200B tokens; SFT on NuminaMath-CoT, RL on 24.9K problems (GSM8K + MATH + DeepScaler); eval on 500 held-out problems, pass@1 from 16 samples at T=0.7. Fits give ρ = −0.87 (R²=0.90) at log10 C = 17.0 tightening to ρ = −0.99 (R²=1.00) at log10 C = 18.5. |

**Mechanism (RQ1) — three policy-update categories, k = 3, ε_tail = 0.05**
- **Ground-truth amplification**: correct move already in top-k, further reinforced. Dominant on easy bins — up to ~0.8 of B1 states.
- **Tail discovery**: correct move promoted from probability < 0.05 into top-k. Small in absolute terms (≤ ~0.05 of states) but *increases* with difficulty.
- **Wrong-mode amplification**: correct move stays outside top-k while the preferred wrong move is reinforced. Reaches ~0.15–0.20 of states on the hardest bins. Measured across RL steps 50 → 750.
- Takeaway stated by the authors: mitigating wrong-mode amplification is the lever for improving RL beyond pass@1.

**Mechanism (RQ2) — reasoning-trace dynamics**
- RL expands search **breadth, not depth**: width-to-depth ratio and branching factor increase while maximum search depth stays roughly flat.
- Move quality improves for both own moves and predicted opponent replies; the model becomes more likely to surface the ground-truth move inside its CoT and to commit to the best candidate it considered.
- Continuations requiring more than 5 moves remain out of reach — RL improves candidate generation/selection faster than long-horizon search.

**Other findings**
- SFT on plain target moves improves pass@1 but *not* pass@8/pass@16 (samples lack useful diversity); SFT on synthetic reasoning traces improves all pass@k, so it is used throughout.
- pass@16 behaves differently from pass@1 under RL: it rises sharply then flattens for the 20M model, and stays flat or degrades slightly for larger models — in that regime extra pretraining beats extra RL.
- Models rarely solve B5 puzzles at all, so aggregate pass@k is reported over B1–B4 only.

## What this adds beyond pretraining-only compute-optimality

[[scaling-laws]] and [[chinchilla]] answer "given C FLOPs of *pretraining*, how do I split it between N and T?" This paper adds a second, outer split — *how much of the total budget should be pretraining at all* — and shows the answer is compute-dependent: the RL share rises with total compute (5% → 32% empirically for 20M; ~20% → 28% across the extrapolated frontier). Crucially, it finds the Chinchilla answer to the *inner* question is unchanged: at frontier points, T* does not systematically deviate from the Chinchilla-optimal token count for the pretraining compute actually spent. It also reframes pretraining loss as a *predictor of post-RL performance*, not just of pretraining-time perplexity — and the correlation gets tighter, not looser, the more RL compute is spent (|ρ| 0.93 → 0.99), which is the opposite of what "RL washes out the prior" would predict.

> [!note] Scope caveat from the authors
> The slope relationship is presented as a **local** empirical trend over the studied compute range, not a global law. Where stronger models approach saturation on easy benchmarks, the sigmoid curve plateaus and systematically compresses the estimated local slope.

## Concepts & entities
- [[scaling-laws]] — the paper extends the scaling-law program across the pretraining/post-training boundary rather than within pretraining alone.
- [[chinchilla]] — the fitted `L(N,T)` is Chinchilla-form, and the compute-optimal frontier shows no systematic deviation from Chinchilla token allocation.
- [[pre-training]] — pretraining validation loss and token count are the two properties that predict post-RL behavior.
- [[reinforcement-learning]] — RL with verifiable rewards is the post-training stage being scaled.
- [[agentic-rl]] — puzzle solving is framed as a multi-step interactive decision problem with environment replies between moves.
- [[policy-optimization-methods]] — GRPO is the optimizer used for all RL runs.
- [[ppo]] — GRPO is the group-relative descendant of PPO-style policy optimization.
- [[chain-of-thought]] — the SFT traces are explicit serialized search trees, giving direct structural access to the model's reasoning.
- [[deepseek-r1]] — the RL-with-verifiable-rewards recipe this pipeline mirrors (GRPO, binary outcome reward).
- [[rlhf]] — contrasted setting: outcome-verifiable reward here replaces a learned preference model.

## References
- [arXiv:2607.16097](https://arxiv.org/abs/2607.16097)
- Models & datasets: huggingface.co/pavelslab-nyu/pre2post-chess; code: github.com/pavelslab-nyu/pre2post-chess
- Khatri et al. (2025) — sigmoid RL scaling law that Eq. 1 is a Taylor expansion of.
- Hoffmann et al. (2022) — Chinchilla; Kaplan et al. (2020) — pretraining scaling laws.
- Yue et al. (2025) — "RL only sharpens" position; Yuan et al. (2025) / Sun et al. (2025) — opposing and mixed positions this paper adjudicates.
