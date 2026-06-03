---
title: "Coding Agents are Effective Long-Context Processors"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - coding-agents-effective-long-context-processors-cao-2026.md
arxiv_id: "2603.20432"
authors:
  - Weili Cao
  - Xunjian Yin
  - Bhuwan Dhingra
  - Shuyan Zhou
first_author: Weili Cao
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Coding Agents are Effective Long-Context Processors

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This paper argues that long-context processing can be *externalized* from an LLM's latent, uninterpretable attention into explicit, executable interactions. LLMs scaled to massive context windows still degrade significantly as context length grows because access is mediated by attention rather than by interpretable operations. Instead, the authors let off-the-shelf frontier coding agents organize text in a file system and manipulate it with their native tools (executable code, terminal commands), treating a large corpus as a directory structure to navigate rather than a window to attend over.

Used as a general interface across long-context reasoning, retrieval-augmented generation, and open-domain QA — over corpora up to three trillion tokens — these coding agents outperform published state of the art by 17.3% on average. The authors attribute the gains to two factors: **native tool proficiency** (using executable code/terminal commands instead of passive semantic queries) and **file system familiarity** (navigating massive corpora as directories). The takeaway: delegating long-context processing to coding agents is an effective alternative to both semantic search and brute-force context-window scaling.

## Key points

- Reframes long-context processing as **explicit, executable interaction** over a file system rather than latent attention over a long window.
- LLMs degrade as context grows because attention is latent/uninterpretable; agents sidestep this by manipulating text as files.
- Evaluated on long-context reasoning, RAG, and open-domain QA with corpora up to **3 trillion tokens**.
- Off-the-shelf frontier coding agents beat published SOTA by **17.3% on average**.
- Two attributed causes: **native tool proficiency** (executable code/terminal vs. semantic queries) and **file system familiarity** (corpus-as-directory navigation).
- Positions coding agents as an alternative to semantic search and context-window scaling for long-context tasks.

## Concepts & entities

[[long-context-llm]] · [[code-as-harness]] · [[llm-agent]] · [[retrieval-augmented-generation]] · [[attention]] · [[tool-use]] · [[open-domain-question-answering]] · [[context-window]]

## References

- arXiv: [arXiv:2603.20432](https://arxiv.org/abs/2603.20432)
- Local source: `sources/coding-agents-effective-long-context-processors-cao-2026.md`
