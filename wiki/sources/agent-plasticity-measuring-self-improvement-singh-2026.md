---
title: "Agent Plasticity: Measuring Self-Improvement Through Experience"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - agent-plasticity-measuring-self-improvement-singh-2026.md
arxiv_id: "2610.08902"
authors:
  - Harman Singh
  - Anton Bakhtin
  - Rulin Shao
  - Gabriel Synnaeve
  - Ilia Kulikov
  - Rob Fergus
  - Sanjeev Arora
  - Kurt Keutzer
  - Jason Weston
  - Anuj Mahajan
  - Anirudh Goyal
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Agent Plasticity: Measuring Self-Improvement Through Experience

**Harman Singh, Anton Bakhtin, Rulin Shao, Gabriel Synnaeve, Ilia Kulikov, Rob Fergus, Sanjeev Arora, Kurt Keutzer, Jason Weston, Anuj Mahajan, Anirudh Goyal** ([[uc-berkeley|UC Berkeley]] · [[meta|Meta]] Superintelligence Labs · [[university-of-washington|UW]] · [[princeton-university|Princeton]]) — [arXiv:2610.08902](https://arxiv.org/abs/2610.08902), v1, cs.AI, 6 Oct 2026. Preprint.

## Summary

Benchmarks measure what an agent can do at a fixed moment. This paper measures **how efficiently it gets better**. Model weights stay frozen. Between checkpoints, a reflection pass rewrites a persistent inventory: CLAUDE.md/AGENTS.md, [[skill-md-format|SKILL.md]] skills, Python tools, strategies and memory. Each acting episode starts with fresh context but inherits that inventory. Held-out in-distribution (ID) and out-of-distribution (OOD) games are never shown to reflection. **Agent plasticity P = ΔS / C** is the held-out score gain per unit of learning cost. **P_sat** is the gain up to the fitted saturation point (90% of improvement), reported in pp per $1,000. The test environments are Chess (Hard and Easy, vs depth-limited Stockfish), 5×5 Go, 6×6 Hex, and NetHack (through native [[claude-code|Claude Code]] and Codex harnesses).

## Key points

- **Improvement is model- and task-dependent.** In Hard chess, Claude Fable 5 rises from 37.5% to **73.3%** held-out ID and Opus 5 from 25.0% to 66.4% (checkpoints 16–20). GPT-5.6 Sol starts near 0% and reaches 36.9%. Several models stay flat or decline. Opus 4.8 loses ground in Go but improves in Hex.
- **Endpoint ≠ efficiency.** On the combined chess + Go + Hex curve, **Fable 5 reaches the highest fitted endpoint**, while **GPT-5.6 Sol has the highest P_sat (298 pp/$1k)**, ahead of Opus 5 (233), Fable 5 (57) and Opus 4.8 (14). GPT-5.6 Luna gets 0; GPT-5.5 had not saturated.
- **NetHack:** only **[[claude-opus-5-5|Claude Opus 5.5]]** shows a reliable rise. Mean score grows from ~2k to ~6k, with a fitted gain of 66.25 normalized points at a learning cost of **$1,072.73**, giving P_sat = 61.75. GPT-5.6 Sol, the most plastic model on the board games, *declines* in NetHack (≈570 → 420).
- **OOD transfer is partial.** Gains against stronger held-out opponents are smaller or inconsistent, with exceptions such as GPT-5.6 Sol in Hex (OOD 28.1 → 78.1).
- **Where the loop breaks.** Weak improvers rarely reuse artifacts: GPT-5.6 Luna takes ~98% of relevant decisions without its artifacts, Gemini 3.1 Pro ~87%, and the strong group 2–6%. Strong improvers reuse at 94–98% yet still diverge in score, so artifact **quality and application**, not just recall, is the residual bottleneck. One Fable 5 lineage wrote its own chess engine.

> [!warning] Newsletter vs paper
> The blurb is substantively correct. Fable 5's "best final" is the fitted endpoint on the *combined* three-game curve, and GPT-5.6 Sol's "best per $" is P_sat on the same curve. "+66 for ~$1,073" is the NetHack fitted gain and cost for Opus 5.5. The authors stress that plasticity is a protocol-dependent measure that depends on provider pricing and the saturation fit, **not a universal model ranking**.

## What it does not show

- **Experience is not separated from compute.** The paper says the comparisons "do not isolate the contribution of environment experience from compute required for artifact development".
- **Per-dollar efficiency inherits list prices.** A pricing change re-ranks models.
- **Games with crisp feedback.** NetHack is the only long-horizon, partially observed case, and five of six models show no reliable rise there.

## Concepts & entities

- [[self-improving-agent]] · [[experiential-learning]] · [[skill-library]] · [[procedural-memory]] — frozen-weight improvement through persistent artifacts.
- [[agent-evaluation]] · [[agent-benchmark]] — measuring learning rate rather than endpoint capability.
- [[tool-creation]] — self-written engines and tools as artifacts.
- [[claude-opus-5-5]] · [[claude-opus-4-8]] · [[claude-code]] · [[meta]].
- Siblings: [[are-self-evolving-agents-continual-learners-chen-2026]] (plasticity vs stability under drift), [[on-the-fragility-of-self-improving-agents-ye-2026]], [[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]].

## References

- [arXiv:2610.08902](https://arxiv.org/abs/2610.08902)
