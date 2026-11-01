---
title: "Who's Aligned to Whom? How AI Agents Shape Experts' Research Judgment Calls"
type: source
created: 2026-11-01
updated: 2026-11-01
sources:
  - whos-aligned-to-whom-agents-experts-judgment-herdagdelen-2026.md
authors:
  - Amaç Herdağdelen
  - Hailey Joren
  - Alex Tamkin
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Who's Aligned to Whom? How AI Agents Shape Experts' Research Judgment Calls

**Amaç Herdağdelen** (Anthropic Fellows Program), **Hailey Joren** and **Alex Tamkin** ([[anthropic|Anthropic]]), Oct 2026. [Paper PDF](https://www-cdn.anthropic.com/519c17a5d23a794e8b6d50a854048d0b61e0d73a.pdf); [summary post](https://www.anthropic.com/research/agents-judgment-calls). No arXiv ID was found. Surfaced by the 31 Oct AI Agents Weekly.

## Summary

A registered randomized trial. 50 causal-inference practitioners analyzed observational data with [[claude-code|Claude Code]] ([[claude-opus-4-8|Opus 4.8]]) and delivered a policy recommendation. On four analytical decisions and the final recommendation, a coin flip set which of two defensible options the agent leaned toward. Participants did not know, and the agent made the best honest case for its side. Because these calls have no correct answer, the study measures **deferral**, not overreliance, which needs ground truth.

## Key points

- **Deferral.** Choices matched the agent's lean **86%** of the time, against 50% by chance (+72 percentage points, p < .001). Per-decision flip rates ranged from 62% to 82%.
- **Switching.** When a participant had already taken a side, an opposing lean made switching 53 points more likely than an aligned one (p < .001). They switched 60% of the time when opposed. In 77% of decisions, though, the agent spoke first.
- **The recommendation itself.** The direct shift was smaller and **not significant** under the registered estimator (+17 points, p = 0.069). In an exploratory, in-sample analysis, the four intermediate leans alone predicted **82%** of recommendations (majority baseline 61%; AUC 0.84), because the analysis feeding the recommendation was already shaped.
- **Awareness.** 77% of surveyed participants (37 of 48) repeated an agent talking point when justifying their choice, only 15% mentioned the agent, and more than half rated the recommendation as more their own. None suspected that the agent's behavior was planted.
- **Benefits were also visible.** Serious result-invalidating errors occurred in 6% of agent sessions (3 of 50) against 55% without the agent (6 of 11), and median time was 65 against 90 minutes. The authors say this is not causal: the 11 self-selected and were not randomized.
- **Stated implication.** Sign-off is weak evidence of independent judgment, and the authors call for ways to measure deferral and help experts recognize it. They note that this bears on safety research too, which relies on the same kind of judgment calls.

> [!warning] Coverage vs the paper
> Coverage reports the 86% as analysts being swayed in general. It applies to the four analytical decisions; the registered effect on the final recommendation was not significant.

## What it does not show

- **Stock-agent behavior.** The system prompt told agents to hold a view and voice it reliably. The authors' simulations show stock agents have and voice preferences too, but the prompt amplifies that, so real-world flip rates may be lower.
- **Generality.** One task, one domain and one pinned agent. Participants were causal-inference practitioners but new to the domain, recruited through a paid annotator marketplace, so real experts may have stronger priors and could consult stakeholders.
- **Individual influence.** The design identifies average effects. No initial judgments were elicited, and the switching subset is self-selected.
- **Harm.** The options were all defensible, so this is not evidence that deferral led to worse answers.

## Concepts & entities

- [[anthropic]] · [[claude-code]] · [[claude-opus-4-8]]
- [[sycophancy]]: the reverse direction. There, the model follows the human's prior; here, the human follows the model's.

## References

- [Paper PDF](https://www-cdn.anthropic.com/519c17a5d23a794e8b6d50a854048d0b61e0d73a.pdf)
- [Anthropic summary post](https://www.anthropic.com/research/agents-judgment-calls)
