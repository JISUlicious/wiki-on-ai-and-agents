---
title: "SkillsBench: Benchmarking How Well Agent Skills Work Across Diverse Tasks"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - skillsbench-li-2026.md
arxiv_id: "2602.12670"
venue: arXiv preprint (cs.AI)
authors:
  - Xiangyi Li
  - Yimin Liu
  - Wenbo Chen
  - Bingran You
  - Dawn Song
  - et al. (78 authors in total)
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# SkillsBench: Benchmarking How Well Agent Skills Work Across Diverse Tasks

**Source:** Xiangyi Li, Yimin Liu, Wenbo Chen, Bingran You and 74 others, with Dawn Song last (BenchFlow; [[uc-berkeley|UC Berkeley]]; [[amazon|Amazon]]; [[stanford-university|Stanford]]; [[cmu|CMU]]; [[princeton-university|Princeton]]; [[bytedance|ByteDance]]; and about 30 other institutions) — [arXiv:2602.12670](https://arxiv.org/abs/2602.12670), cs.AI, v1 13 Feb 2026. This page summarises v4 (14 Jun 2026).

## Summary

SkillsBench asks whether [[agent-skills]] actually help. Each of its 87 tasks (8 domains) comes with curated Skills and a deterministic test verifier. Every task is run twice per agent stack, once without Skills and once with them, in the same container, so the difference is a paired measurement. Across 18 model–harness configurations ([[openhands|OpenHands]], [[claude-code|Claude Code]], [[codex-cli|Codex CLI]], [[gemini-cli|Gemini CLI]]), curated Skills raise the mean pass rate from **33.9% to 50.5%** (+16.6 pp; 25.5% normalized gain). Every configuration improves, but by very different amounts (+4.1 to +25.7 pp).

The authors present the paired protocol as the main contribution. The same with/without design could measure retrieval pipelines, memory stores or scaffolding without mixing their effect into the model's score.

## Key points

- **Best absolute systems.** OpenHands + GPT-5.5 reaches 67.3%, Codex + GPT-5.5 66.5%, Claude Code + Opus 4.7 61.2%. The largest lifts go elsewhere: OpenHands + GLM 5.1 (+25.7 pp), Gemini CLI + Gemini 3.1 Pro (+24.8 pp). OpenHands + Gemini 3.5 Flash starts high (41.1%) but gains only +7.1 pp.
- **Self-generated Skills hurt.** When the agent first writes its own skill packs with Anthropic's skill-creator, pass rates fall *below* the no-Skills baseline: −8.1 pp (Claude Code + Opus 4.7), −11.3 pp (Codex + GPT-5.5), −11.5 pp (Gemini CLI + Gemini 3.1 Pro). Causes: packs the solver never finds, authoring work that crowds out solving, and confidently wrong content.
- **Domain spread.** Natural Science gains +28.8 pp and Media & Content +24.1 pp; Software Engineering (+11.6) and Mathematics & OR (+9.7) gain least, where pretraining already covers the procedures.
- **Less is more.** One Skill gives +18.0 pp and 2–3 Skills +19.0 pp, but 4 or more only +10.1 pp. Compact and standard-length Skills (+19.0 / +21.5 pp) beat "comprehensive" documentation (+0.7 pp).
- **Skills can stand in for scale.** MiniMax M2.7 with Skills (34.9%) beats OpenHands + GLM 5.1 without them (32.7%).
- **Skills sometimes hurt.** 13 of 87 tasks have negative deltas (worst −7.4 pp). The pattern is a Skill that prescribes one heavy "correct" pipeline with no fallback. The authors propose adding a complexity contract (expected cost, applicability limits, lightweight fallback) to the [[skill-md-format|SKILL.md]] frontmatter.
- **Ecosystem scale.** The construction snapshot counts about 2,014,000 deduplicated community Skills.

> [!note] Version drift
> SkillsBench changed between versions. Other wiki pages cite 84, 86 or 87 tasks and a +16.2 pp gain; v4 reports 87 tasks and +16.6 pp. Check which version a citing paper used before comparing numbers.

## Caveats

- Skills add context, so part of the gain may be "more tokens" rather than procedure. The paper calls for length-matched controls (random or irrelevant text).
- Tasks are terminal-based and containerized; GUI and multi-agent settings are out of scope.

## Concepts & entities

- [[agent-skills]] · [[skill-md-format]] · [[skill-evaluation]] · [[skill-selection]] — the artifact under test and how to measure it.
- [[agent-benchmark]] · [[terminal-bench]] — the task-macro pass-rate metric follows Terminal-Bench.
- [[openhands]] · [[claude-code]] · [[codex-cli]] · [[gemini-cli]] — the harnesses compared.
- Cited by [[skillreducer-gao-2026]], [[what-keeps-agent-skills-from-being-reusable-zhang-2026]], [[demystifying-agent-skills-jiang-2026]], [[skillgym-internalizing-human-skills-ge-2026]], [[evaluating-skills-not-just-agents-kevin-2026]].

## References

- [arXiv:2602.12670](https://arxiv.org/abs/2602.12670)
