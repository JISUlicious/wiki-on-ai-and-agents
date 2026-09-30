---
title: "SoL-Pi: Recursively Scaling Auto-Research Loops for Efficient Agent Harness"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - sol-pi-scaling-auto-research-loops-efficient-agent-harness-liu-2026.md
arxiv_id: "2609.20519"
authors:
  - Haozhe Liu
  - Tian Ye
  - Sensen Gao
  - Qihang Cao
  - Yitong Li
  - Mingchen Zhuge
  - Duomin Wang
  - Ruihua Zhang
  - Ping Luo
  - Jiawang Bian
  - Lei Zhu
  - Ligeng Zhu
  - Enze Xie
  - Song Han
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# SoL-Pi: Recursively Scaling Auto-Research Loops for Efficient Agent Harness

**Haozhe Liu\*, Tian Ye\*, … Enze Xie, Song Han** ([[nvidia|NVIDIA]]; NTU; [[mit|MIT]]) — [arXiv:2609.20519](https://arxiv.org/abs/2609.20519), PDF dated 17 Aug 2026. Preprint / blog post; no venue.

## Summary

Most harness-evolution work optimizes task score. SoL-Pi optimizes **token efficiency** at a fixed capability floor, and extends [[pi-mono|Pi]] as the base harness. A research agent reads execution traces from Pi, proposes harness changes, and runs each proposal as an isolated, disposable autoresearch-style lineage ([[andrej-karpathy|Karpathy]]'s propose–run–keep loop) with a [[ralph-loop|Ralph-loop]] implement–review cycle. The search is a broad-to-deep funnel: 152 proposed directions across six families, run on 535 executable search environments (495 repository issue–PR tasks and 40 synthetic verifier tasks), totalling more than 3,000 runs and more than 60,000 agent–environment interactions. Acceptance has two gates, both declared in advance and kept outside the optimizer's control. Every capability metric must stay within tolerance, and at least one efficiency metric must improve.

Four mechanisms survive and together form SoL-Pi. **Action Fusion** attaches a follow-up command to file-mutation tools. **Online Context Compact** compacts at plan-step boundaries only when projected savings beat the cache-rewrite cost. **ObservationPack** replaces large tool outputs, after two sends, with a stable handle and a 1 KB excerpt. **Evidence-Preserving Reducer** has a cheaper model (GPT-5.6 Luna) compress build and test logs into a receipt that is checked deterministically, and falls back to the raw log when the check fails. EdgeBench is held out: 11 of its 51 public tasks for one-way acceptance and 40 for final evaluation.

## Key points

- **EdgeBench, GPT-5.6 Sol (search backend).** Pi scores 44.8 using 2.15B tokens at a cost of $1,339. The full stack ("Efficiency") scores **42.0 on 1.10B tokens (−49.0%) at $894 (−33.2%)**, which is 93.7% of Pi's score. Other harnesses on the same model: Codex 34.7, OpenCode 29.6, Oh-My-Opencode 38.5.
- **Transfer to Opus 5 (held-out backend), unchanged.** Pi 44.8 on 2.37B tokens. The full stack scores 42.2 on 1.31B (−44.7%), with cost −33.5%. Claude Code on the same model: 43.7 at $2,535.
- **"Performance" point.** This is the single mechanism with the best EdgeBench score, chosen *per backend* from Table 4: ObservationPack on Sol (47.2, +5.3%) and Action Fusion on Opus (50.5, +12.8%). Because EdgeBench picks the configuration, this row is selected on the test set.
- **Terminal-Bench 4 (63 CPU-only tasks).** Codex and Pi each solve **18**; SoL-Pi solves **15**, at $211 vs Pi's $286. Cost per solved task is 11.6% lower, but absolute capability drops.
- **IMO 2026, Lean-verified, 6 problems.** Codex 5/6, Pi 3/6, SoL-Pi 3/6. SoL-Pi has the lowest cost per solved problem ($20.90).
- **Agent swarm on kernel optimization** (one two-hour run each; a Codex coordinator with 20 workers): the SoL-Pi workers reach 1,127 cycles for $60.11, the Pi workers 1,366 cycles for $82.12, and a single Codex agent 1,333 cycles for $39.20.
- Mechanisms trigger less often and less intensely on Opus 5, the backend the search never saw. The authors call this "preliminary evidence of transfer".

> [!note] Abstract vs body
> The abstract's "estimated hourly savings of $8.75–$13.50 relative to native Codex and Claude Code" appears only in the abstract. The body gives no derivation for any per-hour figure. "Comparable performance" also covers a 6.3% relative score drop on Sol and 5.7% on Opus.

## What it does not show

- **No matched-compute search baseline.** This is an efficiency paper, and the natural control would be Pi with a smaller budget or fewer turns. It is not run. Every comparison is a harness-vs-harness row from a **single run** without seeds, and the Terminal-Bench 4 and IMO samples are small (63 tasks and 6 problems).
- The authors do address the held-out objection of [[rethinking-evaluation-harness-evolution-wang-2026]], which they cite: EdgeBench stays isolated from the search, and the full stack is fixed before it is evaluated.

## Concepts & entities

- [[agentic-harness-engineering]] — automated harness search, here with an efficiency objective rather than a score objective.
- [[self-improving-agent]] — framed as RSI at the harness layer. The authors call scaling environments and ideas "pretraining the harness".
- [[pi-mono]] — the base harness and the main comparator. [[claude-code]] · [[opencode]] are the native and third-party rows.
- [[prompt-compression]] · [[prefix-caching]] · [[kv-cache]] — ObservationPack and Online Context Compact trade prefix reuse against repeated input.
- [[the-harness-effect-token-economics-writer-2026]] — the harness as a lever on token cost, not only on accuracy.
- [[harness-vs-model-attribution]] — a fixed harness across Sol and Opus, with efficiency gains that hold across the two.
- [[multi-agent]] — the swarm experiment.

## References

- [arXiv:2609.20519](https://arxiv.org/abs/2609.20519)
- Related harness search: Meta-Harness ([[meta-harness-lee-2026]]), AHE ([[agentic-harness-engineering-lin-2026]]), AutoHarness ([[autoharness-lou-2026]]).
