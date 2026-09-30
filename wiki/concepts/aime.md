---
title: AIME
type: concept
created: 2026-09-30
updated: 2026-09-30
sources:
  - deepseek-r1-2025.md
  - native-sparse-attention-yuan-2025.md
  - quantization-hurts-reasoning-liu-2025.md
  - kimi-linear-kimi-team-2025.md
  - nemotron-3-puzzle-75b-a9b-nvidia-2026.md
  - weak-to-strong-on-policy-distillation-yu-2026.md
  - on-policy-self-distillation-without-supervision-li-2026.md
  - on-policy-delta-distillation-heo-2026.md
  - normalized-low-rank-adaptation-kang-2026.md
  - hbm-is-not-all-you-need-wei-2026.md
  - ttpo-test-time-policy-optimization-wang-2026.md
  - vibethinker-3b-verifiable-reasoning-xu-2026.md
  - what-is-missing-from-ai-post-training-ai-lim-2026.md
  - agent-skills-survey-xu-2026.md
  - how-transparent-is-diffusiongemma-engels-2026.md
  - sleep-time-compute-lin-2025.md
status: draft
importance: medium
---

# AIME

**AIME** (American Invitational Mathematics Examination) is a math competition whose problems have integer answers. It became the standard competition-math benchmark for [[reasoning]] models. Each year's contest is scored as its own benchmark, so papers write **AIME 2024 / AIME24**, **AIME 2025 / AIME25** or **AIME 2026 / AIME26**. One year has 30 problems (MathArena's AIME 2025 set has 30), so a single problem is worth 3.33 points. [[what-is-missing-from-ai-post-training-ai-lim-2026|Lim et al.]] point this out when they read pass@8 differences only qualitatively. Scores are usually averages over many samples (Avg@12, Avg@32, Avg@64), because single-sample accuracy on 30 problems is noisy.

## Why the year matters

Once a year's problems and solutions are online, they can leak into pretraining data ([[benchmark-contamination]]). **MathArena** (Balunović et al., 2025) scores models on competitions as soon as they are released, and reports "strong signs of contamination in AIME 2024". This is why the wiki's sources move on over time. [[deepseek-r1-2025|DeepSeek-R1]] and [[native-sparse-attention-yuan-2025|NSA]] report AIME 2024. Most 2025–26 distillation papers report AIME24 and AIME25 side by side. [[ttpo-test-time-policy-optimization-wang-2026|TTPO]], [[vibethinker-3b-verifiable-reasoning-xu-2026|VibeThinker-3B]] and the [[diffusiongemma]] card use AIME 2026. Scores from different years are different benchmarks and should not be compared across rows.

## Recorded scores

| System | Year | Score | Source |
|---|---|---|---|
| DeepSeek-R1-Zero (RL from V3-Base) | 2024 | 15.6 → 77.9 pass@1 (86.7 with self-consistency) | [[deepseek-r1-2025\|DeepSeek-R1]] |
| DeepSeek-R1 | 2024 | 79.8 pass@1 | [[deepseek-r1-2025\|DeepSeek-R1]] |
| NSA-R vs full attention (SFT, 16K limit) | 2024 | 0.146 vs 0.092 | [[native-sparse-attention-yuan-2025\|NSA]] |
| R1-Distill-Qwen-32B BF16 → W4A4 | AIME-120 (unspecified years) | 74.49 → 72.17 | [[quantization-hurts-reasoning-liu-2025\|Liu et al.]] |
| Qwen3-8B: base → OPD → W2S-OPD | 2024 / 2025 | 26.1 → 62.1 → 68.9 / 21.1 → 54.8 → 60.1 | [[weak-to-strong-on-policy-distillation-yu-2026\|W2S-OPD]] |
| Qwen3-4B non-thinking: base → u-OPSD | 2024 / 2025 | 25.83 → 37.50 / 17.78 → 27.78 | [[on-policy-self-distillation-without-supervision-li-2026\|u-OPSD]] |
| Gemma4-E4B-it: → OPD² | 2024 | 51.7 → 69.2 | [[on-policy-delta-distillation-heo-2026\|OPD²]] |
| Qwen3-32B / Qwen3-8B, BF16 | 2024 / 2025 | 85.3 / 86.7; 75.5 / 67.8 | [[hbm-is-not-all-you-need-wei-2026\|HMA-Serve]] |
| Kimi Linear vs MLA (SFT, Avg@64) | 2025 | 21.3 vs 20.6 | [[kimi-linear-kimi-team-2025\|Kimi Linear]] |
| Nemotron 3 Puzzle vs Super (BF16, no tools) | 2025 | 89.7 vs 92.2 | [[nemotron-3-puzzle-75b-a9b-nvidia-2026\|Nemotron Puzzle]] |
| Agentic Proposing, 30B solver | 2025 | 91.6 | [[agent-skills-survey-xu-2026\|Agent skills survey]] |
| DiffusionGemma vs Gemma 4 26B A4B | 2026 | 69.1 vs 88.3 | [[diffusiongemma]] |
| Qwen3-1.7B / 4B / 8B: base → TTPO | 2026 | 37.8 → 48.9 / 65.8 → 70.8 / 67.5 → 73.9 | [[ttpo-test-time-policy-optimization-wang-2026\|TTPO]] |
| VibeThinker-3B (with CLR) | 2026 | 94.3 (97.1) | [[vibethinker-3b-verifiable-reasoning-xu-2026\|VibeThinker-3B]] |

## Same model, different numbers

- **Qwen3-8B on AIME25**: 21.1 in W2S-OPD, 23.33 in u-OPSD's non-thinking table, and 67.8 in HMA-Serve's BF16 row. u-OPSD's number is explicitly non-thinking mode. The W2S-OPD and HMA-Serve pages do not state the mode, sample count or budget, so the threefold gap is unexplained on those pages. TTPO's thinking and non-thinking tables show a gap of that size is possible from mode alone.
- **Qwen3-1.7B TTPO on AIME26**: 48.9 in the main test-time-training table and ablations, and 46.5 in the λ sweep and privileged-information ablations, which train on OpenThoughts.
- **DeepSeek-R1 family on AIME 2024**: 77.9 is R1-Zero at the end of RL, 86.7 is R1-Zero with self-consistency, and 79.8 is the final R1.
- **Quantization direction**: [[hbm-is-not-all-you-need-wei-2026|HMA-Serve]]'s paper says full BFP8 costs AIME accuracy, but on Qwen3-8B AIME25 it gains 7.5 points (75.3 vs 67.8). Its page flags this contradiction.

## Other uses

AIME is the hard end of [[quantization-quality]] studies (−3.9% at W4A4KV4, versus about 0 on GSM8K). [[sleep-time-compute-lin-2025|Sleep-time compute]] builds a "Stateful AIME" variant. [[how-transparent-is-diffusiongemma-engels-2026|Engels et al.]] use AMC/AIME/IMO variants in DiffusionGemma's transparency and monitorability tests. The DiffusionGemma gap of about 19 points is the wiki's clearest case of parallel decoding hurting chained reasoning ([[diffusion-language-model]]).

## Related

[[benchmark]] · [[benchmark-contamination]] · [[reasoning]] · [[on-policy-distillation]] · [[test-time-compute]] · [[gpqa]] · [[deepseek-r1]]

## References

- Balunović et al., "MathArena: Evaluating LLMs on Uncontaminated Math Competitions" (2025), [arXiv:2505.23281](https://arxiv.org/abs/2505.23281). Evaluates on newly released competitions and reports "strong signs of contamination in AIME 2024".
- MathArena AIME 2025 dataset (30 problems with integer gold answers): https://huggingface.co/datasets/MathArena/aime_2025
- The wiki sources listed in frontmatter.
