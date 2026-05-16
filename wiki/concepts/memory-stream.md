---
title: Memory Stream
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - generative-agents-park-2023.md
status: draft
importance: medium
tags:
  - 2023
---

# Memory Stream

The **memory stream** is an append-only, natural-language log of an agent's experience, introduced by [[generative-agents]] ([[generative-agents-park-2023|Park et al. 2023]]). Every observation, dialogue turn, action outcome, or [[reflection-mechanism|reflection]] is recorded as a timestamped NL string and never deleted. At decision time, the agent retrieves the top-k most relevant entries by a composite score combining `recency × importance × relevance`, where recency is an exponential decay over time, importance is an LLM-assigned 1-10 score at write time, and relevance is embedding similarity to the current query.

## Where it appears

- [[generative-agents]] — original Smallville simulation; the canonical implementation.
- [[memgpt]] — `recall storage` plays a comparable role in the [[tiered-memory|tiered]] [[virtual-context|virtual context]] design.
- [[memory-management]] — the stream is the archetypal *write-everything, score-at-read* memory backend; later work (e.g. [[memorybank]], [[mirix]]) layers structure or [[ebbinghaus-forgetting|forgetting]] on top.

## References

- [[generative-agents-park-2023]]
