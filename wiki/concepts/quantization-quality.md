---
title: "Quantization Quality: Perplexity and Capability Preservation"
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - accuracy-is-not-all-you-need-dutta-2024.md
  - scaling-laws-for-precision-kumar-2024.md
  - low-bit-favors-undertrained-ouyang-2024.md
  - quantization-hurts-reasoning-liu-2025.md
  - long-context-quantization-mekala-2025.md
  - multilingual-quantization-marchisio-2024.md
builds-on:
  - [[quantization]]
status: complete
importance: high
---

# Quantization Quality: Perplexity and Capability Preservation

How much capability does [[quantization]] actually cost? The short answer is that **the headline numbers are far more reassuring than the underlying reality**, and the gap between them is the most important thing on this page.

## Loss by bit width

> [!warning] Perplexity is not comparable across papers
> WikiText-2 perplexity depends on context length. Llama-2-7B FP16 is reported as **5.47 @2048**, **5.12 @4096**, and Llama-2-70B as **3.32 @2048** vs **3.120 @4096**. SmoothQuant even uses seqlen 512 for one table and 2048 for another *in the same paper*. Any table mixing these is wrong.

**8-bit is genuinely near-lossless — but only weight-only or FP8.** Kurtic et al.'s ~500,000 evaluations find FP8 "effectively lossless across all model scales" (99.31% / 99.72% / 100.12% recovery at 8B / 70B / 405B). [[llm-int8-dettmers-2022|LLM.int8()]] matches FP32 C4 perplexity exactly on OPT-13B (12.45 → 12.45).

The caveat is severe: **naive INT8 *activation* quantization is catastrophic**, not near-lossless. Per-tensor W8A8 on OPT-175B gives WikiText perplexity **93,080** vs FP16's 10.99. [[smoothquant-xiao-2022|SmoothQuant]] repairs it to 11.17. "8-bit is lossless" is a claim about weights, not activations.

**4-bit weight-only is the practical default.** AWQ INT4-g128 costs +1.8% to +2.7% perplexity on Llama-2; GPTQ costs OPT-175B **+0.36%** (8.34 → 8.37).

**The knee is at 3–2 bits — but it is a property of the *method*, not the bit width.** LLaMA3-8B WikiText-2:

| Method | 8-bit | 4-bit | 3-bit | 2-bit |
|---|---|---|---|---|
| RTN | 6.2 | 8.5 | 27.9 | 1.9e3 |
| GPTQ | 6.1 | 6.5 | 8.2 | 2.1e2 |
| AWQ | 6.1 | 6.6 | 8.2 | 1.7e6 |
| QuIP | — | 6.5 | **7.5** | **85.1** |

With vector quantization the cliff largely disappears: QuIP# and AQLM reach **3.83–3.91** on Llama-2-70B at ~2 bits (FP16 3.12), and QuIP# claims 3-bit beats *a theoretical lossless 4-bit model*. Meanwhile 2-bit RTN/AWQ collapses to chance — LLaMA2-7B LAMBADA goes **73.32 → 0.00**.

> [!note] The "4-bit is optimal" claim is dated
> Dettmers & Zettlemoyer's k-bit inference scaling laws (35,000+ experiments, 19M–176B) established 4-bit as the universal accuracy-per-bit optimum, and it held "across all models studied." That was 2022. QuIP#/AQLM/QTIP have since made 2–3 bit competitive per bit. Date-stamp the claim rather than repeating it as timeless.

## Perplexity understates the damage

This is the section that matters most.

### Flips: accuracy stays flat while behavior diverges

[[accuracy-is-not-all-you-need-dutta-2024|Dutta et al.]] (Microsoft Research India) measure **%flips** — answers that changed correct→incorrect *or* incorrect→correct. Their finding: *"Accuracy is preserved within 1% for the majority of the quantization methods, tasks and models… [yet] all quantization schemes except GPTQ W8A16 have significant %flips"* — **up to 13.6%**.

MMLU, % of correct / incorrect baseline answers that flipped:

| Model | BnB 8-bit | SmoothQuant 8-bit | GPTQ 4-bit | BnB 4-bit |
|---|---|---|---|---|
| Llama2-7b chat | 4.7 / 7.9 | 16.9 / 24.5 | 9.3 / 15.3 | 12.8 / 19.9 |
| Llama2-70b chat | 3.0 / 7.2 | 3.1 / 8.2 | 3.7 / 9.5 | 5.0 / 13.2 |
| Yi-34b chat | 1.6 / 7.8 | **36.4 / 55.2** | 5.8 / 18.4 | 6.0 / 19.4 |

The **mechanism** is elegant: define *top margin* as the probability gap between the best and second-best option. Quantization perturbs logits; answers flip when the margin is small. Correct answers have higher margins, so **more wrong answers flip than right ones** — the two directions nearly cancel in aggregate accuracy while the model's actual behavior has changed substantially. KL-divergence correlates with %flips at Spearman **0.981**.

On MT-Bench the cancellation stops working: Llama-2-70B drops **7.431 → 6.801** at GPTQ 4-bit (~5%), with degradation *"higher for the harder turn2 problem"* (up to 10%).

### Automatic metrics understate human-visible damage by 10–55×

The single most striking number in this literature, from [[multilingual-quantization-marchisio-2024|Marchisio et al.]] (Cohere For AI):

| Language | Automatic-benchmark drop | Human-evaluator drop |
|---|---|---|
| French | **−0.3%** | **−16.6%** |
| Japanese | **−1.7%** | **−16.0%** |

Also: LLM-as-a-Judge on challenging prompts **−25.9%**; MGSM math **−13.1%**. Discriminative tasks (XWinograd, XCOPA, XStoryCloze) were "minimally impacted" — precisely the task type that dominates leaderboards.

### Perplexity can be flat while downstream swings 14 points

Vicuna-7B + SparseGPT across 50 calibration sets: perplexity **12.72 ± 0.18** while BoolQ accuracy ranged **57.0% → 71.6%**. Near-constant perplexity, 14.6pp downstream swing, purely from calibration-data choice.

## Where damage concentrates

**Long context.** The cleanest capability-specific result ([[long-context-quantization-mekala-2025|Mekala et al.]], EMNLP 2025) — RULER accuracy delta vs BF16:

| Method | 8K | 64K | 128K |
|---|---|---|---|
| FP8 | −1.9 | −1.3 | −0.7 |
| AWQ-int4 | −0.6 | −6.4 | −5.8 |
| GPTQ-int4 | −0.2 | −4.5 | **−12 to −13** |
| BNB-nf4 | −0.2 | −0.9 | **−23** |

Same models, same quantization: **~0% damage at 8K, up to −23 points at 128K**. Non-English degrades "up to five times more than in English."

**Reasoning and math.** [[quantization-hurts-reasoning-liu-2025|Liu et al.]] (COLM 2025) find *"harder tasks suffer up to 4× greater degradation than simpler ones."* At W4A4KV4 on a 32B reasoning model: AIME −3.9%, MATH-500 −1.2%, **GSM8K −0.0%**. Small models collapse where large ones shrug: W3G128 costs Qwen-1.5B **−16.6%** but Qwen-32B only −3.2%. Li et al. add that at W3, the dominant failure is **incorrect logic (~50%)**, not calculation error, and that multi-step reasoning and self-calibration tolerate quantization *worse* than instruction-following and in-context learning.

> [!warning] Contradiction — is code disproportionately damaged?
> **Claim A (yes)**: Shi et al. report *"up to a 92% drop in HumanEval pass rate"*; Giagnorio et al. measure CodeLlama-7B Python pass@1 falling 29.8 → 16.4 at 2-bit.
> **Claim B (no)**: Kurtic et al. find code among the *most robust* — Llama-3.1-70B HumanEval pass@1: BF16 79.7, FP8 80.0, **INT4 80.5**. Liu et al. put math ~4× worse than code.
> **Partial reconciliation**: the 92% figure is Llama-2-13B off an already-low ~18% base, on TensorRT-LLM, and the culprit is **W8A8 activation quantization, not low bit width**. Claim A is well-supported at ≤3-bit or under INT8 activations; at 4-bit weight-only, Claim B looks right. No paper cleanly settles it on matched models and harnesses.

## Model size: bigger-quantized beats smaller-full-precision

At equal memory, quantizing a larger model wins ([[quantization]] practitioners' rule of thumb, now measured):

| Comparison | Leaderboard v1 | v2 |
|---|---|---|
| 4-bit Llama-2-13B (6.5 GB) vs FP16 7B (14 GB) | **+4.66%** | +1.16% |
| 4-bit Llama-3.1-405B (202 GB) vs FP16 70B (140 GB) | **+3.15%** | +4.07% |

**The exceptions matter**: FP16 7B beats quantized 13B on **TruthfulQA**, and 4-bit 405B is *worse* than FP16 70B on **IFEval**. The rule holds on average and breaks on hallucination and instruction-following.

> [!warning] Contradiction — are larger models more robust to quantization?
> It depends entirely on **what** you quantize, and the literature routinely conflates the two cases:
> - **Weights and KV cache** — weight-tensor kurtosis *decreases* with model size (fewer outliers), so **larger = more robust**. OPT + GPTQ 4-bit is cleanly monotone: 125M +12.6% → 13B +1.8% → 175B **+0.36%**.
> - **Activations** — activation kurtosis *increases* sharply with size (>1000 vs ~10 for weights), so **larger = less robust**. This is the [[llm-int8-dettmers-2022|emergent-outlier phase shift at ~6.7B]].
>
> And even the weight-only trend isn't clean for Llama-2, where **13B beats both 7B and 70B** in relative perplexity at 4, 3, and 2 bits. The folk claim overshoots the data.

An under-cited subtlety: Dettmers et al. show the outlier phase shift tracks **perplexity, not parameter count** — *"model perplexity rather than mere model size determines the phase shift."*

## Scaling laws: overtrained models quantize worse

[[scaling-laws-for-precision-kumar-2024|Kumar et al.]] (ICLR 2025) fit post-training-quantization degradation as

$$\delta_{PTQ}(N,D,P) = C_T\left(\frac{D^{\gamma_D}}{N^{\gamma_N}}\right)e^{-P/\gamma_{post}}$$

with all coefficients positive — degradation **grows with training tokens D**, shrinks with parameters N, and grows exponentially as precision falls. Since fitted γ_D ≈ γ_N, it is approximately a power law in **D/N**. Their conclusion: *"there exists an amount of pretraining data beyond which additional data is actively harmful to performance at inference-time."* Compute-optimal *pretraining* precision comes out at **7–8 bits**.

This is corroborated from two independent directions:

- [[low-bit-favors-undertrained-ouyang-2024|Ouyang et al.]] (Pythia, 1,500+ checkpoints): *"models with larger sizes or fewer training tokens experience less quantization-induced degradation."* ΔqLoss ≈ b·D^0.53.
- **Direct measurement across model generations**: q2_K costs **+10.8%** perplexity on Llama-2-7B (2T tokens) but **+56.5%** on Llama-3-8B (15T tokens). Qwen3-8B at 3-bit AWQ goes 10.4 → **23.8** perplexity where LLaMA3-8B goes 9.2 → 11.6.

**The practical implication is uncomfortable**: as models are trained on ever more tokens, they are becoming *harder* to quantize. Tokens-per-parameter predicts fragility better than parameter count.

> [!note] Scope caveat
> Kumar et al. fit on models up to **1.7B params / 26B tokens**, then extrapolate. The authors themselves caution *"not to take too seriously the numerical values we fit, but the trends and functional forms."*

## Evaluation caveats

1. **Report KL-divergence and %flips**, not just accuracy (§ flips above).
2. **Calibration choice rivals the effect being measured** — GPTQ zero-shot spread of 0.9–1.6pp across 50 calibration sets, up to 9pp per task; 7.7pp swing on code pass@1 from calibration data alone.
3. **Prompt-template variance can exceed the quantization delta** — MMLU template ranges of 2–10pp vs a largest observed quantization delta of 3.2pp.
4. **Harness and recipe are first-order confounds.** Kurtic et al. explicitly refute prior published conclusions (that AWQ beats GPTQ at 4-bit; that W8A8-INT is inferior), attributing them to *"suboptimal hyperparameter choices"* and *"insufficient hyperparameter tuning."* Marchisio et al. recover **>6pp on math** by changing scaling granularity alone.
5. **Averages hide per-task collapse**, and near-chance baselines make "% recovery" meaningless. Harder benchmarks reveal more: Llama-3.1-8B W4A16 recovers 98.7% on Leaderboard-v1 but **96.1% on v2**.
6. **Never conflate W / A / KV.** Any "N-bit" headline that doesn't say *which* is uninterpretable — the single largest effect in this literature (10.99 → 93,080) is an activation artifact, not a bit-width law.
7. **Cheap fidelity proxies fail exactly where you need them.** Per-token KLD correlates with benchmark score overall (ρ ≈ −0.72 to −0.86) but **collapses in the near-baseline "silent zone"** (ρ = +0.00) — the regime where you're actually choosing between good recipes.

> [!note] Disclosure
> The largest and most reassuring study here (Kurtic et al., ~500k evaluations) comes from **Neural Magic / Red Hat**, vendors of the quantization tooling being evaluated; its favourable conclusion is also the commercially convenient one. The scale and openness are genuinely strong — the incentive is worth stating.

## Related

- [[quantization]] — methods. · [[quantization-performance]] — what you buy. · [[quantization-safety]] — the alignment dimension.

## References

- [[accuracy-is-not-all-you-need-dutta-2024]] · [[multilingual-quantization-marchisio-2024]] · [[long-context-quantization-mekala-2025]] · [[quantization-hurts-reasoning-liu-2025]] · [[scaling-laws-for-precision-kumar-2024]] · [[low-bit-favors-undertrained-ouyang-2024]] · [[give-me-bf16-or-give-me-death-kurtic-2024]]
