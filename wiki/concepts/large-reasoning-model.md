---
title: Large Reasoning Model (LRM)
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - brain-inspired-graph-multi-agent-systems-hao-2026.md
  - agentic-ai-next-intelligence-explosion-evans-2026.md
  - when-to-retrieve-during-reasoning-guo-2026.md
  - metis-memory-foundation-model-zhang-2026.md
  - primer-in-post-training-reasoning-data-li-2026.md
status: draft
importance: medium
---

# Large Reasoning Model (LRM)

A **large reasoning model** is an LLM that has been post-trained, usually with RL on verifiable rewards, to produce a long [[chain-of-thought]] before it answers. Examples include [[o1|OpenAI o1]], [[deepseek-r1|DeepSeek-R1]] and QwQ-32B. [[reasoning]] is the *capability*: multi-step inference, whether prompted, trained or latent. An LRM is a *class of model* that has that capability built into its weights and default decoding. The wiki's summary of Metis describes LRMs as having absorbed chain-of-thought into the forward pass. A standard LLM, by contrast, reasons step by step only when prompted to.

## In this wiki

- **The recipe.** [[deepseek-r1]] ([[deepseek-r1-2025]]) is the wiki's reference LRM: [[grpo|GRPO]] with rule-based rewards, where self-verification and backtracking emerge. [[primer-in-post-training-reasoning-data-li-2026]] surveys more than 150 studies on the post-training data that drives LRM progress, and warns that long traces are not necessarily faithful.
- **Collapse beyond a complexity threshold.** [[brain-inspired-graph-multi-agent-systems-hao-2026|BIGMAS]] starts from the finding that both LLMs and LRMs suffer accuracy collapse past a threshold of problem complexity. It reports that its [[global-workspace-theory|global-workspace]] [[multi-agent]] graph improves both kinds of model, a gain it treats as orthogonal to model-level reasoning.
- **A "society of thought".** [[agentic-ai-next-intelligence-explosion-evans-2026|Evans et al.]] argue that LRMs such as DeepSeek-R1 and QwQ-32B improve not just by thinking longer but by simulating internal multi-perspective debate, and that RL for accuracy alone elicits this.
- **Retrieval during reasoning.** [[when-to-retrieve-during-reasoning-guo-2026|ReaLM-Retrieve]] argues that standard [[retrieval-augmented-generation|RAG]] fits LRMs poorly. RAG supplies evidence before reasoning starts, whereas LRMs need it injected at the right step.
- **An analogy for memory.** [[metis-memory-foundation-model-zhang-2026|Metis]] proposes that memory should be internalized into the model the same way reasoning was internalized into LRMs.

Inference-side controls for LRMs are covered in [[test-time-compute]], [[reasoning-effort]] and [[adaptive-thinking]].

## Related

- [[reasoning]] · [[chain-of-thought]] · [[latent-reasoning]] · [[deepseek-r1]] · [[grpo]] · [[reinforcement-learning]]
- [[test-time-compute]] · [[inference-time-compute]] · [[reasoning-effort]] · [[adaptive-thinking]]

## References

- [[brain-inspired-graph-multi-agent-systems-hao-2026]] · [[agentic-ai-next-intelligence-explosion-evans-2026]] · [[when-to-retrieve-during-reasoning-guo-2026]]
- [[metis-memory-foundation-model-zhang-2026]] · [[primer-in-post-training-reasoning-data-li-2026]] · [[deepseek-r1-2025]]
