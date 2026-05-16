---
title: Ebbinghaus Forgetting
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - memorybank-zhong-2023.md
status: draft
importance: medium
tags:
  - 2023
---

# Ebbinghaus Forgetting

The **Ebbinghaus forgetting curve** comes from Hermann Ebbinghaus's 1885 self-experiments on memorization: retention decays roughly exponentially with elapsed time unless the memory is reinforced by recall. The classical form gives memory strength `S = e^(-t / s)`, where `t` is time since last access and `s` is a per-item strength parameter that grows on each successful retrieval.

[[memorybank]] ([[memorybank-zhong-2023|Zhong et al. 2023]]) was the first LLM-agent memory system to operationalize this formalism as a principled eviction-and-strengthening policy. Each stored memory carries its own decaying strength; retrieval reinforces it, neglect causes it to fade out of the working set. The result is a biologically motivated alternative to crude FIFO or LRU eviction.

## Where it appears

- [[memorybank]] — canonical application of the Ebbinghaus curve to LLM memory.
- [[memory-management]] — supplies the *evict* / *strengthen* lifecycle policy.
- [[tiered-memory]] — forgetting strength can govern hot→warm→cold demotion.

## References

- [[memorybank-zhong-2023]]
