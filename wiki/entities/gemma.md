---
title: Gemma
type: entity
created: 2026-09-30
updated: 2026-10-10
sources:
  - how-transparent-is-diffusiongemma-engels-2026.md
  - uniprefill-block-wise-dynamic-sparsification-fan-2026.md
  - pop-prefill-only-pruning-he-2026.md
  - recirculation-mozer-2026.md
  - language-models-can-control-their-own-attention-ho-2026.md
  - basert-native-metal-inference-apple-silicon-rathnayaka-2026.md
  - system-one-ecosystem-sweep-2026-09-28.md
  - wikiskill-compiling-agent-experience-tang-2026.md
  - exploiting-llm-quantization-egashira-2024.md
  - q-resafe-chen-2025.md
family: gemma
parameters: 270M to 31B across generations
release_date: 2024-02-21
status: draft
importance: medium
tags:
  - model
---

# Gemma

[[google|Google]]'s family of open-weights models, which Google describes as built from the same research as Gemini and developed by Google DeepMind ([[deepmind]]) and other Google teams. Gemma is the wiki's most common **open-weight test subject from Google**. Papers on prefill, sparse attention, looping, quantization safety and agent skills evaluate it next to [[qwen]] and [[llama]]. It is also the base of [[diffusiongemma|DiffusionGemma]].

## By generation, as the wiki uses it

| Generation | Variants seen in the wiki | Where |
|---|---|---|
| Gemma (2024) | 2B, 7B / 7B-Instruct | [[exploiting-llm-quantization-egashira-2024\|quantization attacks]], [[q-resafe-chen-2025\|Q-resafe]], [[quantization-safety]] |
| Gemma 2 | layer sweeps only | [[recirculation-mozer-2026\|Recirculation]] |
| Gemma 3 | 270M IT, 1B, 4B, 12B, 27B | [[nimble]] baseline, [[uniprefill-block-wise-dynamic-sparsification-fan-2026\|UniPrefill]], [[pop-prefill-only-pruning-he-2026\|POP]], Recirculation |
| Gemma 4 | E2B, E4B, 12B, 26B A4B, 31B | [[how-transparent-is-diffusiongemma-engels-2026\|DiffusionGemma transparency]], [[language-models-can-control-their-own-attention-ho-2026\|self-controlled attention]], [[basert-native-metal-inference-apple-silicon-rathnayaka-2026\|BaseRT]], [[wikiskill-compiling-agent-experience-tang-2026\|WikiSkill]] |

## Recurring themes

- **Hybrid attention limits prefill tricks.** Per UniPrefill, Gemma-3-12B has 48 layers in a 5:1 sliding-window : global pattern, so only 8 layers are quadratic. Sparse attention gains least there (1.49× TTFT at 128K against 2.26× on LLaMA-3.1-8B), and it runs at a net loss at 4K. [[methods-to-speed-up-prompt-processing]] recommends token pruning instead for this kind of model. See [[sparse-attention]].
- **Gemma 4 26B A4B as the MoE workhorse.** It is DiffusionGemma's matched autoregressive sibling: its opaque serial depth is 28.6× naively and 1.1× once the inter-step bottleneck is counted. It is the only model [[weight-streaming|TurboFieldfare]] streams. Lichen runs it frozen on llama.cpp for [[jev|Jev]]-style decisions ([[system-one-ecosystem-sweep-2026-09-28]]). See [[mixture-of-experts]].
- **Gemma 4 31B as the open-weight agent baseline.** It serves as a backbone or comparand in WikiSkill (33.9 → 73.1 on LiveMath with skills transferred from a 4B model), [[meta-n-recursive-self-improvement-emergent-depth-kim-2026|Meta^n]], [[evomal-self-poisoning-self-evolving-coding-agents-wu-2026|EvoMal]] and [[evo-bench-can-language-models-improve-agent-harness-huang-2026|Evo-Bench]].
- **Unstable where architecture is unusual.** Recirculation finds Gemma 4 E2B/E4B sweeps unstable, which it attributes to cross-layer KV sharing and per-layer embeddings. Training-free looping gives no robust gain on Gemma 3.

## Lineage

- **Gemma (February 2024)**: 2B and 7B models, pre-trained and instruction-tuned.
- **Gemma 2 (2024)**: 2B, 9B and 27B. It adds interleaved local-global attention and grouped-query attention, and trains the 2B and 9B by [[knowledge-distillation]].
- **Gemma 3 (2025)**: 1B to 27B, at least 128K context and vision input. A higher local-to-global ratio with short local spans shrinks the KV cache. A 270M model followed in August 2025, meant for task-specific fine-tuning.
- **Gemma 4 (2026)**: E2B, E4B, 12B, 26B A4B and 31B, built from Gemini 3 research. [[diffusiongemma|DiffusionGemma]] (2026-06-10) adapts Gemma 4 26B A4B into a diffusion LM.

## Related

- [[diffusiongemma]] · [[google]] · [[deepmind]] · [[qwen]] · [[llama]] · [[sparse-attention]] · [[mixture-of-experts]]

## References

- [Gemma: Introducing new state-of-the-art open models](https://blog.google/technology/developers/gemma-open-models/) — 2024-02-21; 2B and 7B; built from Gemini research by Google DeepMind and other Google teams
- [Gemma 2, arXiv:2408.00118](https://arxiv.org/abs/2408.00118) — 2B/9B/27B, interleaved local-global attention, GQA, distillation
- [Gemma 3 Technical Report, arXiv:2503.19786](https://arxiv.org/abs/2503.19786) — 1B–27B, ≥128K context, higher local:global ratio
- [Introducing Gemma 3 270M](https://developers.googleblog.com/en/introducing-gemma-3-270m/) — 2025-08-14
- [Gemma 4](https://deepmind.google/models/gemma/gemma-4/) — variants; "from Gemini 3 research and technology"
- [[how-transparent-is-diffusiongemma-engels-2026]] · [[uniprefill-block-wise-dynamic-sparsification-fan-2026]] · [[pop-prefill-only-pruning-he-2026]] · [[recirculation-mozer-2026]] · [[language-models-can-control-their-own-attention-ho-2026]] · [[basert-native-metal-inference-apple-silicon-rathnayaka-2026]] · [[system-one-ecosystem-sweep-2026-09-28]] · [[wikiskill-compiling-agent-experience-tang-2026]]

## Lineage sources

- [[s1-decision-model-raghavan-2026]] — s1: Gemma 4 26B-A4B decision fine-tune
