---
title: AI Safety
type: concept
created: 2026-09-29
updated: 2026-09-30
sources:
  - darwin-godel-machine-zhang-2025.md
  - hyperagents-zhang-2026.md
status: draft
importance: medium
---

# AI Safety

**AI safety** is the umbrella field concerned with keeping AI systems from causing unintended or harmful outcomes, whether through misspecified objectives, unsafe behaviour during learning, misuse, or loss of human oversight. An early ML-focused agenda, *Concrete Problems in AI Safety* ([[dario-amodei|Amodei]], [[chris-olah|Olah]], [[paul-christiano|Christiano]] et al., 2016), framed "accidents" as five research problems: avoiding side effects, avoiding [[reward-hacking]], scalable supervision, safe exploration, and distributional shift. This page is a hub. The wiki holds the substance on the narrower pages linked below.

## Map of the wiki's safety pages

| Area | Pages |
|---|---|
| Objective misspecification | [[reward-hacking]] · [[sycophancy]] · [[reward-modeling]] |
| Training-time alignment | [[rlhf]] · [[constitutional-ai]] · [[rlaif]] · [[dpo]] · [[institutional-alignment]] |
| Agentic failure | [[agentic-misalignment]] · [[prompt-injection]] · [[indirect-prompt-injection]] |
| Evaluation and probing | [[red-teaming]] · [[agent-evaluation]] |
| Runtime control | [[guardrailing]] · [[steering]] · [[representation-engineering]] |
| Understanding models | [[mechanistic-interpretability]] · [[sparse-autoencoder]] |
| Deployment and compression | [[quantization-safety]] · [[quantization-conditioned-backdoor]] |
| Misuse | [[ai-enabled-cyber-operations]] · [[influence-operations]] · [[illicit-distillation]] |

## Self-improving systems

Both pages that link here are about [[self-improving-agent|self-improving agents]]. In the [[darwin-godel-machine]] ([[darwin-godel-machine-zhang-2025]]), all execution is sandboxed with time limits and the self-modification domain is deliberately narrow. The archive gives a traceable lineage of every modification for human review. The paper flags that a self-improvement loop optimized against benchmarks that miss safety properties "could amplify misalignment over successive generations", and suggests [[constitutional-ai]]-style invariants as a future direction. [[hyperagents-zhang-2026|HyperAgents]] makes even the meta-modification procedure editable, and it discusses sandboxing, human oversight and what safety means for self-accelerating systems.

## Related

- [[center-for-ai-safety]] · [[dan-hendrycks]] · [[anthropic]] · [[jan-leike]] · [[paul-christiano]]
- [[self-improving-agent]] · [[darwin-godel-machine]] · [[godel-machine]]

## References

- Amodei et al., "Concrete Problems in AI Safety" — [arXiv:1606.06565](https://arxiv.org/abs/1606.06565)
- [[darwin-godel-machine-zhang-2025]] · [[hyperagents-zhang-2026]] · [[agentic-misalignment-lynch-2025]] · [[red-teaming-ganguli-2022]]

## Related 2026 sources

- [[divide-consult-conquer-capability-laundering-russinovich-2026]] — Microsoft: an unaligned orchestrator composes harmful capability from individually permitted frontier-model answers
