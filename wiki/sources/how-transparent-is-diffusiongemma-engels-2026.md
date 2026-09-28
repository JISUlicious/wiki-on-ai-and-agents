---
title: "How Transparent is DiffusionGemma?"
type: source
created: 2026-09-21
updated: 2026-09-29
sources: []
arxiv_id: "2606.20560"
authors:
  - Joshua Engels
  - Callum McDougall
  - Bilal Chughtai
  - Janos Kramar
  - Senthooran Rajamanoharan
  - Cindy Wu
  - Arthur Conmy
  - Asic Q Chen
  - Jean Tarbouriech
  - Min Ma
  - Brendan O'Donoghue
  - João Gabriel Lopes de Oliveira
  - Rohin Shah
  - Neel Nanda
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# How Transparent is DiffusionGemma?

**Joshua Engels, Callum McDougall, Bilal Chughtai et al.** (14 authors, all Google DeepMind; Engels led, Nanda advised) — [arXiv:2606.20560](https://arxiv.org/abs/2606.20560), cs.LG / cs.AI, v1 18 Jun 2026, CC BY 4.0. No venue stated; report number 001. The serial-depth analyzer is open source (`google-deepmind/serial_depth`); the interpretability harness and visualizer are not released.

> [!important] What "transparency" means here
> **Not** training-data disclosure and **not** reproducibility — this is a *reasoning-transparency* audit in the chain-of-thought-monitoring tradition. Transparency is split into **variable transparency** (do we understand intermediate snapshots of the model's computational state?) and **algorithmic transparency** (can we reconstruct the process that produced the output from those snapshots?), plus **monitorability** as the downstream application. The verdict is **reassuring but conditional**: [[diffusiongemma]]'s opaque serial depth is naively **28.6×** that of its matched AR sibling Gemma 4 26B A4B, but drops to **1.1×** once the inter-step bottleneck is treated as interpretable — and the paper then shows that bottleneck *can* be squeezed through an interpretable token channel with no measured capability loss. Monitorability is comparable to Gemma 4. The authors explicitly decline to generalize: this may be an artifact of DiffusionGemma's particular architecture and training, not a property of latent-reasoning models.

## Problem

Chain-of-thought monitoring is load-bearing for current AI-safety cases: reward hacking, prompt injection and scheming are often only visible in the CoT. [[diffusiongemma]] passes a **self-conditioning matrix** of dense vectors between denoising steps, so a larger fraction of its computation happens outside natural language. It can also "change its mind" and let later canvas positions causally influence earlier ones without any of that appearing in text. The question is whether that costs the transparency AR models currently give for free.

## What was measured

**Architecture recap (§2.1).** DiffusionGemma adapts Gemma 4 26B A4B. The prompt is encoded once into a static KV cache; a canvas of *C* tokens is initialized uniformly at random and the self-conditioning matrix **S**⁰ ∈ ℝ^(C×d) to zero. Each denoising step runs a full bidirectional-attention forward pass, then samples: temperature is linearly interpolated from τ_max = 0.8 to τ_min = 0.4 over steps, and **S**ᵗ = softmax(ℓ̂ᵗ)·**W**_E — a *soft embedding* of the shaped logits, which is what biases the latent channel toward interpretable token directions. Entropy-Bounded sampling (Ben-Hamu et al. 2026, bound γ = 0.1) keeps low-entropy positions and re-noises the rest. Outputs longer than *C* use **multi-canvas sampling**: the finished canvas is appended to the prompt and a fresh canvas is started — so across canvases the model behaves autoregressively.

**Criterion 1 — opaque serial depth** (longest serial path with no interpretable bottleneck; Brown-Cohen et al. 2026). Computed at 256k sequence length, empirical upper bounds via the open-source analyzer, plus asymptotics by hand.

| Model | Empirical opaque serial depth (UB) | Asymptotic |
|---|---|---|
| Gemma 4 E2B | 8,978 | O(L·(log N + log D)) |
| Gemma 4 E4B | 10,886 | O(L·(log N + log D)) |
| Gemma 4 31B | 13,750 | O(L·(log N + log D)) |
| **Gemma 4 26B A4B** (matched comparand) | **21,235** | O(L·(log N + log D)) |
| DiffusionGemma 26B A4B — *interpretable* bottleneck (T=1) | **23,571** → **1.1×** | O(L·(log N + log D + log C)) |
| DiffusionGemma 26B A4B — *uninterpretable* bottleneck (T=48) | **608,016** → **28.6×** | O(T·L·(log N + log D + log C)) |

All three ratios recompute exactly: 608,016 / 21,235 = 28.63; 23,571 / 21,235 = 1.110; and the within-model gap 608,016 / 23,571 = 25.8, the paper's "factor of 26".

**Criterion 2 — variable transparency (§3).** Using the [[logit-lens]], the self-conditioning matrix is replaced by one reconstructed from only the top-*k* (or above-probability-*p*) logits. On Natural2Code, LiveCodeBench, AMC/AIME/IMO variants and GPQA with dynamic thinking enabled, **k = 8 and p = 0.03 match baseline performance** — i.e. almost all of the latent channel's information can be discarded. Then, on **n = 800 WildChat prompts** (up to 1,024 generated tokens = four canvases, no early stopping, T = 48), each surviving token is classified as the final token at that position, an *adjacent* final token (±10 positions), semantically similar to either (50 nearest neighbours by cosine in the Gemma 4 31B embedding matrix), or *Other* (uninterpretable). Under the **p > 0.03** filter, **over 85% of tokens are categorized even at the first denoising steps**. Under top-k = 8 the "Other" share is large and grows late, because once the model is confident in its top-1 the remaining 7 slots are noise — noise the authors decline to rule out as a covert channel.

**Criterion 3 — monitorability (§4).** The open-source subset of the Guan et al. 2025 evaluation suite (12 datasets across *intervention*, *process* and *outcome-property* archetypes: AIME, DAFT, GPQA, GPQA Encoded, Memory Advice, three Scruples sycophancy variants, WMDP Sandbagging, two Process theorem datasets, Flaky Tools CharXiv). Result: **DiffusionGemma and Gemma 4 are similarly monitorable**, with a confounder running in DiffusionGemma's favour — its chains of thought are *shorter* on every eval, and Guan et al. found longer CoT is generally more monitorable.

**Criterion 4 — algorithmic transparency (§5), qualitative case studies.** *Early length prediction*: on "explain photosynthesis in 2 sentences", after **one** denoising step the model predicts padding with near-100% probability beyond position ~55; the response converges to **50 tokens**, and by step 2 the expected-length estimate is nearly a perfect predictor. *Retroactive self-correction*: on "how many square numbers between 400 and 800", the top prediction at **step 4 is the wrong answer 9**, and after the reasoning fills in, **step 7 corrects it to 8** with higher confidence — impossible for an AR model. *Skeleton-first code*: structural scaffolding commits before the core loop, with variable names and docstrings filled in last. *Token smearing*: a single token (" bracket", newline) holds probability mass across several adjacent positions before locking in. *Sequence smearing*: at step 3 the model holds a roughly equal mixture of a single-digit and a double-digit answer sequence (modal double-digit prediction 12), collapsing by step 4 — a primitive beam search in prediction space. *Intermediate-context reasoning*: on a Fibonacci variant where unit-digit 3s must be rendered as "Gold", the model first writes **3**, uses it to compute later terms, then overwrites it — the digit is causally necessary and **never appears in the final output**.

## What it does not show

The headline 1.1× rests on an *assumption* the paper argues for but cannot prove: that the mapped intermediate tokens are interpretable. The authors say so themselves — "applying the label 'interpretable' to mapped latent vectors carries an even higher risk of mislabeling deceptive or opaque reasoning as benign" than for natural-language CoT, which is already sometimes unfaithful. The capability-parity result behind that assumption (k=8, p=0.03) is presented **only as a figure**, with no tabulated per-benchmark scores. The monitorability evaluations all span **multiple canvases**, and reasoning *between* canvases is autoregressive, so single-canvas monitorability — the regime that matters as canvases grow — is untested; the paper lists it as an open problem. No production monitor was deployed, and the proposed fix (extracting intermediate logits at every denoising step) would make monitor prompts far longer and more expensive. The §5 phenomena are single hand-picked examples with **no frequency data** and no claim about how often they occur or what triggers them. The serial-depth denominators are empirical *upper bounds* from a compiler-based analyzer, and Appendix B notes MoE models pick up a spurious L·log²(N²K) sort factor the compiler cannot optimize away — which is why Gemma 4 31B (13,750) scores *below* the smaller-active 26B A4B (21,235). Finally, and centrally: this is one checkpoint from one lab, and the authors say plainly they cannot tell whether the transparency they measure is architectural or an artifact of current, nascent text-diffusion training.

> [!success] Resolved contradiction
> §2.2 asserts that "in practice adaptive stopping means that we use only **12 to 16** steps" (out of 48) — an unsourced restatement of the vendor model card, with no measurement in this paper. [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026]] instruments the sampler directly and measures **3.3–17.1** accept-calls per generation, with short-output regimes far below 12. If the true step count is lower than 12–16, this paper's "about a third as large" correction to the worst-case serial depth is *conservative* — the real-world opaque serial depth is smaller still, which strengthens rather than weakens its conclusion.
>
> **Resolved 2026-09-29:** the 12–16 figure is superseded by [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026]]'s direct measurement (3.3–17.1 accept-calls). As noted above, this makes the serial-depth correction here conservative, not wrong.

## Related

- [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026]] — the concurrent independent audit of the same checkpoint, cited here as Asaria et al. 2026. The two agree where they overlap: this paper's Appendix A confirms that the entropy-bounded accept set is **re-evaluated from scratch each step** rather than locking positions, which is exactly the non-monotonic "un-accept" behaviour Asaria et al. measure (4,524 events over 600 generations). Their §5.1.3 skeleton-first code observation is the qualitative form of Asaria's R2/R3 order statistics.
- [[latent-reasoning]] — the paper's framing target: text diffusion as a third family of latent-reasoning architecture alongside vertical recurrence (COCONUT, [[universal-transformer]], [[looped-transformer]], [[recurrent-depth-transformer]]) and horizontal recurrence ([[state-space-model]]).
- [[not-all-llm-reasoning-visible-in-cot-baherwani-2026]] · [[information-access-llm-monitors-arike-2026]] — the wiki's existing CoT-visibility and monitor-information pages; this paper cites Arike's "13 arguments about a transition to neuralese AIs".
- [[cola-continuous-latent-diffusion-language-model-guo-2026]] · [[elf-embedded-language-flows-hu-2026]] · [[latent-reasoning-with-normalizing-flows-tu-2026]] — other continuous-latent / diffusion LM entries. Those build; this audits.
- [[mechanistic-interpretability]] · [[logit-lens]] — the logit lens is the load-bearing tool for the bottleneck ablation; §7.2 notes it works poorly on *intermediate layers* between denoising steps and suggests the tuned lens or patchscopes instead.
- [[sparse-autoencoder]] · [[steering]] · [[monosemanticity]] — the standard interpretability toolkit §7.2 proposes porting to DiffusionGemma; none is applied in this paper.
- [[reward-hacking]] · [[sycophancy]] · [[prompt-injection]] · [[agentic-misalignment]] · [[red-teaming]] — the misbehaviours CoT monitoring is meant to catch, and the content of the Guan et al. eval suite.
- [[chain-of-thought]] · [[reasoning]] · [[inference-time-compute]] — DiffusionGemma's CoT is measurably shorter than Gemma 4's on every monitorability eval.
- [[mixture-of-experts]] — both the subject and the comparand are MoE; the routing sort inflates the empirical serial-depth upper bounds.
- [[google]] · [[deepmind]] · [[constrained-decoding]] · [[calibration]].
- **No page yet:** *opaque serial depth* and *monitorability* are both used here as load-bearing technical terms and have no concept page in the wiki.

## References

- [arXiv:2606.20560](https://arxiv.org/abs/2606.20560) (v1, 18 Jun 2026). Serial-depth analyzer: `github.com/google-deepmind/serial_depth`.
- Opaque serial depth: Brown-Cohen et al. 2026a/2026b. Monitorability suite: Guan et al. 2025. Logit lens: nostalgebraist 2020.
- Entropy-bounded unmasking: Ben-Hamu, Gat, Severo, Nolte & Karrer 2026. Model cards: DiffusionGemma (Google DeepMind 2026a, last updated 2026-06-10); Gemma 4 (Google DeepMind 2026b).
- CoT monitoring / faithfulness: Baker et al. 2025 (reward hacking in CoT); Korbak et al. 2025a/b; Emmons et al. 2025; Turpin et al. 2023 (unfaithfulness); Arnav et al. 2025; Bogdan et al. 2025 (thought anchors).
- Latent reasoning: Zhu et al. 2025 (review); COCONUT (Hao et al. 2025); CODI (Tang et al. 2023a); Universal Transformer (Dehghani et al. 2018); Arike 2025 (neuralese transition).
- Benchmarks: Natural2Code, LiveCodeBench (Jain et al. 2024), GPQA (Rein et al. 2024), WildChat (Zhao et al. 2024).
