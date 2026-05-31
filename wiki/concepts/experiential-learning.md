---
title: Experiential Learning
type: concept
created: 2026-05-29
updated: 2026-05-29
sources:
  - expel-zhao-2023.md
status: draft
importance: medium
tags:
  - 2023
---

# Experiential Learning

**Experiential learning** (in the LLM-agent sense) is the pattern of distilling reusable *natural-language* insights and rules from a collection of past trajectories, then injecting those insights into future prompts. Unlike [[library-learning]] or [[tool-creation]], the artifacts produced are **not executable code** — they are textual heuristics ("when X, prefer Y") accumulated across many task attempts. It is therefore the NL counterpoint to code-based [[skill-library|skill libraries]]: experience is compressed into language rather than functions.

- **ExpeL** ([[expel-zhao-2023|Zhao et al. 2023]]) gathers experiences across a training set of tasks, extracts cross-task insights via comparison and abstraction, and recalls relevant insights + successful trajectories at inference — without updating model weights.
- Builds directly on per-episode self-reflection ([[reflexion]]) but generalizes it from a single task's scratchpad to a persistent, cross-task insight pool.
- The insight pool functions as a form of distilled long-term [[memory-management|memory]] keyed by relevance.
- Strength is interpretability and zero gradient updates; weakness is that NL rules are softer and less composable than code abstractions.

## Related

- [[reflexion]] — single-episode reflection that experiential learning generalizes.
- [[memory-management]] — how distilled insights are stored and retrieved.
- [[skill-library]] — the executable-code counterpart.

## References

- [[expel-zhao-2023]]
