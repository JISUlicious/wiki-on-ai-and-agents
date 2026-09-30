---
title: GPQA
type: concept
created: 2026-09-30
updated: 2026-09-30
sources:
  - deepseek-r1-2025.md
  - kimi-k3-moonshot-2026.md
  - qwen3.8-flash-next-model-card-2026.md
  - kimi-linear-kimi-team-2025.md
  - weak-to-strong-on-policy-distillation-yu-2026.md
  - nemotron-3-puzzle-75b-a9b-nvidia-2026.md
  - how-transparent-is-diffusiongemma-engels-2026.md
  - sakana-fugu-sakana-ai-2026.md
  - speculative-prefill-liu-2025.md
  - what-is-missing-from-ai-post-training-ai-lim-2026.md
  - when-is-routing-meaningful-huot-2026.md
  - openhands-wang-2024.md
status: draft
importance: medium
---

# GPQA

**GPQA** ("A Graduate-Level Google-Proof Q&A Benchmark", Rein et al., 2023) is a multiple-choice benchmark of questions in biology, physics and chemistry, written by domain experts. The questions are meant to be "Google-proof": in the paper, PhD-level experts reach 65% (74% after discounting mistakes they later recognised), while skilled non-experts reach only 34% even after more than 30 minutes with unrestricted web access. The strongest GPT-4-based baseline in the paper scored 39%. The authors built it for **scalable oversight** research: questions that even people who check an AI's answers cannot easily look up.

The paper defines three nested sets:

- **Extended**: 546 questions.
- **Main**: 448 questions, with low-quality items removed. This is the "GPQA Main" in [[what-is-missing-from-ai-post-training-ai-lim-2026|PostTrainBench]].
- **Diamond**: 198 questions that both expert validators answered correctly and most non-experts got wrong. **GPQA Diamond** is the version almost every model card in this wiki reports.

[[mmlu]] notes that GPQA is one of the harder benchmarks used to separate frontier models once MMLU saturated.

## Recorded scores

| System | Variant / setting | Score | Source |
|---|---|---|---|
| DeepSeek-R1 | Diamond | 71.5 | [[deepseek-r1-2025\|DeepSeek-R1]] |
| DiffusionGemma vs Gemma 4 26B A4B | Diamond | 73.2 vs 82.3 | [[diffusiongemma]] |
| Kimi K3 (max effort) | Diamond | 93.5 | [[kimi-k3-moonshot-2026\|Kimi K3]] |
| Claude Fable 5 / GPT-5.6 Sol / Opus 4.8 / GPT-5.5 / GLM-5.2 | Diamond, from Kimi K3's table | 92.6 / 94.1 / 91.0 / 93.5 / 91.2 | [[kimi-k3-moonshot-2026\|Kimi K3]] |
| Qwen3.8-Flash-Next / Qwen3.8-27B / DeepSeek-V4-Flash / Claude-Opus-4.6 Max | Diamond | 91.7 / 89.2 / 90.8 / 91.3 | [[qwen3.8-flash-next-model-card-2026\|Qwen3.8 card]] |
| Nemotron 3 Puzzle vs Super (BF16) | "GPQA (no tools)" | 78.6 vs 80.5 | [[nemotron-3-puzzle-75b-a9b-nvidia-2026\|Nemotron Puzzle]] |
| Kimi Linear / MLA / GDN-H (1.4T-token SFT) | Diamond, Avg@8 | 62.1 / 57.1 / 58.6 | [[kimi-linear-kimi-team-2025\|Kimi Linear]] |
| Qwen3-8B base → OPD → W2S-OPD | Diamond, out of domain | 38.9 → 54.4 → 56.5 | [[weak-to-strong-on-policy-distillation-yu-2026\|W2S-OPD]] |

## Other uses in the wiki

- **Monitorability and transparency.** [[how-transparent-is-diffusiongemma-engels-2026|Engels et al.]] use GPQA, and an encoded variant (GPQA Encoded), in their chain-of-thought monitorability suite. They find DiffusionGemma and Gemma 4 about equally monitorable.
- **Routing.** In [[sakana-fugu-sakana-ai-2026|Sakana Fugu]], the learned orchestrator sends GPQA-Diamond questions mainly to Gemini-3.1-Pro without being told to. The wiki reads this as evidence of provider specialisation ([[learned-orchestration]], [[sakana-fugu-review]]). [[when-is-routing-meaningful-huot-2026|Huot et al.]] include GPQA among EmbedLLM's ten benchmarks.
- **Short-context checks.** [[speculative-prefill-liu-2025|SpecPrefill]] reports 8-shot GPQA only in figures. [[openhands-wang-2024|OpenHands]] lists it among its 15 integrated benchmarks.

## Reading the numbers

- Frontier Diamond scores in 2026 sit at 91–94. With 198 questions, one question is worth about 0.5 points, so gaps of 1–2 points between frontier models are only a few questions.
- **Opus is reported twice in different tables:** Opus 4.8 at 91.0 in Kimi K3's table and Opus 4.6 (Max) at 91.3 in the Qwen3.8 card. These are different model versions, and each table was compiled by a competing vendor.
- Nemotron's "GPQA (no tools)" does not say which subset it uses, so it is not directly comparable to the Diamond rows.

> [!note] DeepSeek-V4-Flash appears at two values
> 90.8 on the Qwen3.8 model card (V4-Flash-0731) and 89.9 in DeepSeek's own [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026|V4.1-Flash report]] (§5.3.2). Most likely two checkpoints; neither source says.

## Related

[[benchmark]] · [[mmlu]] · [[benchmark-contamination]] · [[reasoning]] · [[aime]] · [[diffusiongemma]]

## References

- Rein et al., "GPQA: A Graduate-Level Google-Proof Q&A Benchmark" (2023), [arXiv:2311.12022](https://arxiv.org/abs/2311.12022). Abstract: 448 questions; 65% / 74% expert accuracy; 34% for non-experts with web access; 39% for GPT-4.
- Subset sizes (Extended 546, Main 448, Diamond 198) and the Diamond inclusion rule: https://arxiv.org/html/2311.12022
- The wiki sources listed in frontmatter.
