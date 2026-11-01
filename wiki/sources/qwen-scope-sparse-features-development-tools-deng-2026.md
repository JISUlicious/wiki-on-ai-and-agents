---
title: "Qwen-Scope: Turning Sparse Features into Development Tools for Large Language Models"
type: source
created: 2026-10-30
updated: 2026-10-30
sources:
  - qwen-scope-sparse-features-development-tools-deng-2026.md
arxiv_id: "2605.11887"
authors:
  - Boyi Deng
  - Xu Wang
  - Yaoning Wang
  - Yu Wan
  - Yubo Ma
  - Baosong Yang
  - Haoran Wei
  - Jialong Tang
  - Huan Lin
  - Ruize Gao
  - Tianhao Li
  - Qian Cao
  - Xuancheng Ren
  - Xiaodong Deng
  - An Yang
  - Fei Huang
  - Dayiheng Liu
  - Jingren Zhou
introduces:
  - [[qwen-scope]]
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Qwen-Scope: Turning Sparse Features into Development Tools for Large Language Models

**Qwen Team** ([[alibaba|Alibaba]]). The arXiv listing names Boyi Deng and 17 co-authors. [arXiv:2605.11887](https://arxiv.org/abs/2605.11887), v1, cs.CL, 12 May 2026. Weights are on [Hugging Face](https://huggingface.co/collections/Qwen/qwen-scope) and ModelScope. This is a technical report: one release plus seven application studies.

## Summary

[[qwen-scope|Qwen-Scope]] is an open suite of [[sparse-autoencoder|sparse autoencoders]] for [[qwen|Qwen]]3 and Qwen3.5. It contains **14 SAE groups across 7 backbones**: 5 dense models (Qwen3-1.7B/8B, Qwen3.5-2B/9B/27B) and 2 [[mixture-of-experts|MoE]] models (Qwen3-30B-A3B, Qwen3.5-35B-A3B). Each covers **every layer** of the residual stream with TopK SAEs at L0 = 50 or 100. Width is 16× the hidden size for dense models (32K–80K features); the MoE models also get 64× variants (128K). The SAEs are trained on in-house pretraining data, with the auxiliary dead-feature loss of Gao et al. (2024) and filtering of outlier-norm activations. Qwen3.5-27B is the only backbone whose SAEs are trained on the instruct model rather than the base model.

The paper's thesis is that SAEs should be **development tools, not only post-hoc analysis**. It shows four kinds of use:

1. **[[steering|Steering]]** at inference time.
2. **Benchmark analysis** without running any models.
3. **Data work:** toxicity classification and safety-data synthesis.
4. **Post-training:** [[supervised-fine-tuning|SFT]] and RL objectives built from SAE features.

## Key points

- **Steering (qualitative only).** There are two case studies and no numbers.
  - A Qwen3 model answering in English slips into Chinese. Ranking SAE features on the bad response finds a Chinese-language feature (#6159). Suppressing it restores the English answer.
  - Turning on a classical-Chinese feature (#36398) switches a modern-Chinese story into a classical literary style.
- **Benchmark redundancy without evaluation.** The authors define a *feature-redundancy* score: how quickly a benchmark's set of activated features saturates as samples are added, adjusted for the total number of features. Across 26 checkpoints and 17 benchmarks, it correlates with the performance-based redundancy curve (subset Kendall-τ AUC) at **Spearman ρ ≈ 0.85**.
- **Overlap between benchmarks.** Overlap is asymmetric: **63% of [[gsm8k|GSM8K]]'s features are covered by MATH**, while only 10% of MATH's are covered by GSM8K. Code benchmarks form a cluster. Feature overlap correlates with the similarity of model scores at Pearson **68.4%** over 28 pairs, rising to **75.5%** after partialling out [[mmlu|MMLU]], which controls for the "rising tide" of general ability.
- **Toxicity classifier made of SAE features.** A few toxic-biased features (K ≤ 10), with no trained classification head, give **held-out F1 > 0.90** in English on Qwen3-1.7B and 8B, at middle-to-late layers.
  - Features found in English transfer to Russian and French but not well to Arabic, Chinese or Amharic.
  - A cheap statistic ("top1-diff") picks a near-best layer before any evaluation.
  - Using **10% of the discovery data recovers about 99%** of macro-F1.
- **Feature-driven safety-data synthesis.** The pipeline finds safety-relevant features that the existing data never activates, then generates prompts that do activate them (an LLM judge checks relevance). It covers **99.74% of the target features** with a small budget.
  - SFT with 4k real + 4k feature-driven examples reaches safety accuracy **77.75**, against 71.75 with natural sampling and 72.00 with random safety synthesis at the same 8k budget. It approaches a 120k safety-only mix and also gets the best IFEval and TruthfulQA scores in the table.
  - Results were similar when Gemini-3-Flash generated the data.
- **SASFT (SAE-guided SFT) against code-switching.**
  - Analysis: activation of the language feature rises in the tokens *before* an unwanted switch, and ablating that feature's direction lowers the switch rate.
  - Method: SASFT adds an auxiliary loss that keeps the target language's features near their average pre-activation.
  - Result: **more than 50% less code-switching** in most settings (into Chinese, Russian or Korean), with complete elimination in some (Qwen3-1.7B, Korean). Six general benchmarks are held or slightly improved. This builds on Deng et al. (2025).
- **SAE-steered "rare negatives" in RL.** Endless repetition is too rare in rollouts for RL to learn to avoid it. So in each [[grpo|GRPO]]-style rollout group (DAPO without dynamic sampling), one of the G rollouts is generated *with repetition features turned up*, giving the policy an explicit negative example.
  - On Qwen3-1.7B, 8B and 30B-A3B, the repetition ratio drops much faster and lower than with vanilla RL.
  - General benchmarks are **mixed**: some gain, some regress.
  - Steering to create *positive* rollouts was tried first and abandoned because it hurt fluency.

## What it does not show

- **Steering has no quantitative evaluation.** Two hand-picked examples, with no success rate, side-effect measurement or comparison against prompting or steering vectors.
- **Missing baselines for the classifier and the data pipeline.** The toxicity classifier is not compared with a linear probe or a fine-tuned classifier on the same activations. The synthesis gains are on one backbone (layer-30 SAE, about 65k features), and the paper says outright that coverage is a proxy.
- **The redundancy proxy is only a correlation.** It holds across 17 benchmarks and one family of checkpoints, and features are read at the last token of one layer only. The authors note that high redundancy is not the same as low quality.
- **RL effects on capability are mixed.** The paper reports this itself; it fixes repetition but does not uniformly preserve capability.
- **No SAE quality metrics in the main text.** There is no reconstruction loss, explained variance or auto-interpretation score per release, so the SAEs cannot be compared with Gemma Scope or Llama Scope from this paper. The dead-feature claim is only qualitative ("negligible").
- **License restrictions.** The paper's social-impact statement prohibits non-scientific use that "interferes with model capabilities". That matters for the steering and unlearning work these SAEs would naturally enable.

## Concepts & entities

- [[qwen-scope]] · [[qwen]] · [[alibaba]]
- [[sparse-autoencoder]] · [[mechanistic-interpretability]] · [[superposition]]: the release extends open SAE suites to Qwen3/3.5, including MoE models.
- [[steering]] · [[steering-vectors]] · [[activation-engineering]]: the rule is h′ = h + α·d along an SAE decoder direction.
- [[transcoder]] · [[sparsified-mlps-vs-transcoders-neurons-in-disguise-shabalin-2026]]: a skeptical view of how faithful sparse dictionaries are, relevant to whether features like #6159 are what they appear to be.
- [[evil-qwen]] · [[neuron-explorer-lyramakesmusic-2026]]: community steering of Qwen internals.
- [[supervised-fine-tuning]] · [[grpo]]: post-training uses.
- [[mmlu]] · [[gsm8k]]: benchmark-overlap analysis.

## References

- [arXiv:2605.11887](https://arxiv.org/abs/2605.11887)
- [Qwen-Scope collection on Hugging Face](https://huggingface.co/collections/Qwen/qwen-scope)
