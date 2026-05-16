---
title: Memory Construction
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - mem-alpha-wang-2025.md
status: draft
importance: medium
tags:
  - 2025
---

Memory construction is the problem of *building* a structured memory store from an unstructured input stream — deciding what to extract, how to categorize it, and where to put it — as opposed to *editing* an existing memory store (which is the framing of [[memory-r1]], whose actions assume the store already exists). Construction is upstream of edit: it answers "what should the memory look like in the first place?" rather than "given the current memory, what should change?" [[mem-alpha]] (Wang 2025) trains an RL agent to construct memory across core / episodic / semantic categories directly from a stream, and shows generalization from 30k-token training sequences to 400k+ at inference (~13× length extrapolation).

## Where it appears

- [[mem-alpha]] — Direct introduction; RL-trained construction of categorized memory from a stream.
- [[memory-r1]] — Contrast: edits an existing store rather than constructing one from scratch.
- [[memory-management]] — Construction is the first phase of the memory lifecycle.

## References

- [[mem-alpha-wang-2025]] — MEM-α: Learning to Construct Memory from Streams via Reinforcement Learning (Wang 2025).
