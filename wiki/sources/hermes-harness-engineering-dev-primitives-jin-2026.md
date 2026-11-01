---
title: "HERMES: Harness Engineering for Software Engineering via Modular Executable Dev-Primitives"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - hermes-harness-engineering-dev-primitives-jin-2026.md
arxiv_id: "2610.07832"
authors:
  - Haibo Jin
  - Xinjie Li
  - Peng Kuang
  - Haohan Wang
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# HERMES: Harness Engineering via Dev-Primitives

**Haibo Jin, Xinjie Li, Peng Kuang, Haohan Wang** ([[uiuc|UIUC]]; Penn State) — [arXiv:2610.07832](https://arxiv.org/abs/2610.07832), v1, cs.SE, 6 Oct 2026. Preprint. Unrelated to Nous Research's [[hermes-agent]].

## Summary

Long-horizon SWE agents keep rebuilding program state that is spread across source files, configs and tests. HERMES moves that state into the repository itself. A **Dev-Primitive** pairs one repository artifact with a **resident LLM** that has read that artifact's implementation and dependencies. The artifact can then be addressed in natural language, can message other primitives about requirements, and can edit itself.

HERMES adds two mechanisms on top. **Dynamic activation** localizes the likely components, follows dependencies to callers, configs and tests, and instantiates only that candidate set. **Bug diagnosis** maps execution failures back to the implicated components and reactivates only those, up to a revision budget (B = 3).

## Key points

- **Four benchmarks, matched backbones.** These are [[swe-bench-verified|SWE-bench Verified]], [[swe-refactor-bench-hong-2026|SWE Refactor Bench]], [[terminal-bench|Terminal-Bench]] 4.0 and DevOps-Gym. Each row compares a published harness (mini-SWE-agent, [[claude-code|Claude Code]], Codex, Grok Build) with HERMES on the same model and effort. The average gain is **+12.4 pp**.
- **Whole-repository migration benefits most.** On SWE Refactor Bench with GPT-5.6 Sol, the Codex harness scores **6.5%** and HERMES **31.0%** at medium effort (31.0 ± 1.3 over 3 repeats). At high effort the scores are 19.0% vs 36.5%. GPT-5.6 Luna goes from 0.0% to 16.0%.
- **Gains shrink as the backbone gets stronger.** On SWE-bench Verified, HERMES improves 19 of 20 backbones and ties Claude Opus 5. With GPT-5.6 Sol it reaches 97.0% vs 96.2%. On Terminal-Bench 4.0, Claude Opus 5 gains only 1.5 and 1.2 points, while GPT-5.6 Terra gains about 26.
- **HERMES usually costs more.** For example, Terminal-Bench 4.0 cost for GPT-5.6 Sol at medium effort rises from $1.9k to $3.4k. Claude Sonnet 5 is the exception: resolution rises while tokens and cost fall.
- **Small primitives are enough.** Keep GPT-5.6 Sol for activation and diagnosis and run the Dev-Primitives on Qwen3-8B. On Terminal-Bench 4.0 that scores 50.6% vs 51.8% for the all-Sol configuration, at **26.2% lower cost** ($2.51k vs $3.40k). Across benchmarks, scaling the primitives themselves adds only 1.2–4.5 points.
- Diagnosis matters more than activation. Scaling diagnosis alone takes the all-Qwen3-8B configuration from 27.6% to 46.1%. Scaling activation alone reaches 36.7%.

> [!warning] Newsletter vs paper
> The figures are reported correctly. Two pieces of context are missing. The "−26.2% cost" compares HERMES with HERMES, small primitives vs all-frontier. It is **not** HERMES vs a baseline harness, and HERMES generally costs more than the baselines. "+12.4%" is percentage points.

## What it does not show

- **Single runs.** Each HERMES configuration is run once per backbone; the only repeat is the default Refactor setting. Most baseline numbers come from leaderboards, not reruns.
- **Results depend on the backbone.** Gains are concentrated on weaker backbones and on refactoring, the setting with the most room to improve.
- **No search baseline.** There is no matched-cost comparison against best-of-N or extra revision rounds for the baseline harnesses.

## Concepts & entities

- [[agentic-harness-engineering]] · [[code-as-harness]] · [[multi-agent]] — component-resident agents as harness structure.
- [[swe-agent]] · [[agent-computer-interface]] · [[context-rot]] — the long-horizon context problem it targets.
- [[swe-bench-verified]] · [[swe-refactor-bench-hong-2026]] · [[terminal-bench]] · [[claude-code]]
- [[harness-vs-model-attribution]] — gains shrink as the backbone strengthens.

## References

- [arXiv:2610.07832](https://arxiv.org/abs/2610.07832)
