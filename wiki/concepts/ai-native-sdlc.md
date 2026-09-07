---
title: AI-Native SDLC
type: concept
created: 2026-09-07
updated: 2026-09-07
sources:
  - agentic-coding-trends-report-anthropic-2026.md
  - swe-refactor-bench-hong-2026.md
  - when-agents-coordinate-destefanis-2026.md
  - same-task-different-work-weinberger-2026.md
  - why-does-claude-md-keep-growing-chakrabarti-2026.md
status: complete
importance: high
---

# AI-Native SDLC

The claim that agentic coding does not merely speed up the software development lifecycle but **restructures the role of the engineer within it** — from writing implementations to specifying, orchestrating and validating agents that write them.

The stages survive. [[agentic-coding-trends-report-anthropic-2026|Anthropic's framing]] is explicit that *"traditional SDLC stages remain"* and what collapses is cycle time. So this is a claim about **who does which stage and how long it takes**, not about a new lifecycle diagram.

## The two halves of the claim, and their very different evidential status

**Half one — the role shift — is well evidenced and largely uncontested.** Engineers report using AI in roughly **60%** of their work. The delegation boundary is consistent and behavioural: they hand off what is **easily verifiable, well-defined, or repetitive**, and keep anything requiring organizational context or judgment. Gains show up as **output volume** rather than time-per-task — more features, more experiments, more previously-deprioritized "papercuts" fixed.

**Half two — autonomous long-horizon delivery — is where the narrative outruns the measurements.** Every controlled long-horizon result in this wiki lands well below what the industry framing implies.

| What is claimed | What is measured |
|---|---|
| Agents work "for days at a time, building entire applications" | [[swe-refactor-bench-hong-2026]]: whole-repository migrations, **28 of 520 runs (5.4%)** pass all three stages; **13 of 20 tasks** get no accepted solution at all |
| Sustained coherence over long horizons | [[merchantbench-long-term-coherence-shi-2026]]: **27.3%** of human performance over 365 simulated days; a rule-based baseline beats 6 of 16 LLM configurations purely by never stopping |
| Multi-agent orchestration replaces single-agent | [[when-agents-coordinate-destefanis-2026]]: naming a coordinator produces **no communication hub and no reliable success gain**; the one significant contrast **fails to reproduce under a sealed rerun** |
| AI reviews AI, so oversight scales | [[rehearse-confidence-cliff-autoresearch-ji-2026]]: judge selective accuracy **82.8% → 56.9%** as a trajectory deepens, *while coverage rises* — degrading silently, committing more |

## The number that should anchor any planning

Developers report being able to **"fully delegate" only 0–20% of tasks** — a figure Anthropic publishes in its own bullish report and names *"the collaboration paradox."*

Take that seriously and the practical shape of an AI-native SDLC is not *autonomy* but **throughput under supervision**: many parallel supervised streams rather than a few unsupervised ones. Which relocates the bottleneck onto human review capacity — precisely what the [[rehearse-confidence-cliff-autoresearch-ji-2026|degrading-judge]] result says cannot be safely automated away.

## What actually determines whether this works

The wiki's measurements point at three factors that dominate model capability, and none of them appear in industry SDLC narratives:

**The harness, not the model.** Scaffold-only variation moves outcomes **11–15 points** on SWE-bench Verified with the model held fixed, and up to ~48 on some leaderboards ([[harness-vs-model-attribution]]). Cost per success swings **5–30×** across harnesses at equal success rate ([[same-task-different-work-weinberger-2026]]).

**Prompt wording, independent of content.** Asking for "multiple approaches" inflates reasoning tokens **2.4–7.4×** across six models **with no success gain**. Bounded-efficiency wording — scope, acceptance criteria, stop condition — is neutral-or-better everywhere. Most agent spend is decided before the model reasons at all.

**Instruction-file entropy.** Agentic instruction files grow **+226%** over their lifetime because adding is cheap and deleting requires provenance nobody recorded ([[why-does-claude-md-keep-growing-chakrabarti-2026]], [[catastrophic-remembering]]). An AI-native SDLC accumulates a new maintenance burden the traditional one didn't have, and **better models ratchet harder**.

## Reading the vendor literature

> [!note] Provenance discipline
> The industry sources here are vendor reports whose case studies are customers, and which carry **no methodology sections** — no sample sizes, no instruments, no time windows. They are valuable as statements of direction and as descriptions of current practice, and they are not evidence for their forecasts. Where this wiki has a controlled measurement on the same question, prefer the measurement and note the gap.

## Related

- [[agentic-coding-trends-report-anthropic-2026]] — the anchor source, and the origin of the 60% / 0–20% figures.
- [[agentic-harness-engineering]] · [[code-as-harness]] · [[harness-vs-model-attribution]] — the layer that actually moves outcomes.
- [[catastrophic-remembering]] — the new maintenance cost.
- [[swe-bench]] · [[swe-refactor-bench-hong-2026]] · [[terminal-bench]] — how "can agents build systems" gets measured.
- [[llm-as-a-judge]] · [[agent-as-a-judge]] — the oversight-scaling assumption.
- [[claude-code]] · [[agent-skills]] · [[multi-agent]] · [[llm-agent]]

## References

- [[agentic-coding-trends-report-anthropic-2026]] · [[swe-refactor-bench-hong-2026]] · [[when-agents-coordinate-destefanis-2026]] · [[same-task-different-work-weinberger-2026]] · [[why-does-claude-md-keep-growing-chakrabarti-2026]] · [[merchantbench-long-term-coherence-shi-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]]
