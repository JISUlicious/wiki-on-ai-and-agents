---
title: Andrej Karpathy
type: entity
created: 2026-09-30
updated: 2026-09-30
sources:
  - parcae-scaling-laws-looped-2026.md
  - rehearse-confidence-cliff-autoresearch-ji-2026.md
  - last-ai-built-by-humans-recursive-self-improvement-duan-2026.md
  - what-is-missing-from-ai-post-training-ai-lim-2026.md
  - wikiskill-compiling-agent-experience-tang-2026.md
affiliations:
  - org: [[university-of-toronto]]
    start: 2005
    end: 2009
    role: BSc
  - org: [[university-of-british-columbia]]
    start: 2009
    end: 2011
    role: MSc
  - org: [[stanford-university]]
    start: 2011
    end: 2015
    role: PhD
  - org: [[openai]]
    start: 2015
    end: 2017
    role: Founding member, research scientist
  - org: [[openai]]
    start: 2023
    end: 2024
status: draft
importance: medium
tags:
  - person
---

# Andrej Karpathy

AI researcher and educator. His own site describes him as a founding member of [[openai|OpenAI]] and later Director of AI at Tesla (2017–2022), where he led the Autopilot computer-vision team. He did his PhD at [[stanford-university|Stanford]] under Fei-Fei Li. In this wiki he appears in three ways: as the author of [[nanochat]], as the originator of the "autoresearch" loop, and as the source of the **LLM Wiki** pattern.

## In this wiki

- **nanochat, the baseline harness.** [[nanochat]] is his minimal end-to-end LLM training harness. [[parcae-scaling-laws-looped-2026|Parcae]] uses it as the parameter-matched Transformer baseline for its [[looped-transformer]] results.
- **Autoresearch, the reference loop.** An agent edits an ML repo, pays for a training run and keeps the edit only if the metric improves. [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] calls Karpathy's nanochat loop the reference instance, and [[what-is-missing-from-ai-post-training-ai-lim-2026]] treats it as the shared reference loop. The Karpathy repo `karpathy/autoresearch` describes itself as agents running research on single-GPU nanochat training. See [[self-improving-agent]].
- **A rung on the self-improvement ladder.** [[last-ai-built-by-humans-recursive-self-improvement-duan-2026|Duan et al.]] place "Karpathy's autoresearch" at **L2, improvement-strategy autonomy**: the agent chooses *how* to improve while the objective and evaluator stay fixed.
- **Benchmark for orchestration.** [[sakana-fugu-review]] reports Fugu-Ultra beating single frontier agents on AutoResearch (123 autonomous experiments, ~14h on one H100) under an identical scaffold.
- **LLM Wiki.** [[wikiskill-compiling-agent-experience-tang-2026|WikiSkill]] names "Karpathy (2026)'s perspective on LLM Wiki" as its inspiration, and that page records that this vault is the same perspective run as a practice. Karpathy's `llm-wiki.md` gist (April 2026) lays out three layers — immutable raw sources, an LLM-maintained markdown wiki, and a schema file such as `CLAUDE.md` — plus **ingest**, **query** and **lint** operations. This vault's `sources/`, `wiki/` and `CLAUDE.md` follow that layout.

## Related

- [[nanochat]] · [[openai]] · [[stanford-university]] · [[self-improving-agent]] · [[parcae]] · [[gpt-2]]

## References

- [karpathy.ai](https://karpathy.ai/) — bio: OpenAI founding member (2015–2017) and return (2023–2024), Tesla Director of AI (2017–2022), Stanford PhD (2011–2015, Fei-Fei Li), UBC MSc (2009–2011), UofT BSc (2005–2009)
- [karpathy/autoresearch](https://github.com/karpathy/autoresearch) — "AI agents running research on single-GPU nanochat training automatically"
- [llm-wiki.md gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — the LLM Wiki pattern (created 2026-04-04)
- [[parcae-scaling-laws-looped-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] · [[last-ai-built-by-humans-recursive-self-improvement-duan-2026]] · [[what-is-missing-from-ai-post-training-ai-lim-2026]] · [[wikiskill-compiling-agent-experience-tang-2026]]

## Related 2026 sources

- [[wfm-wiki-foundation-model-agentic-reasoning-dong-2026]] — Tencent Youtu's graph retriever for LLM-wiki-style memory; not budget-matched, and some cells self-described as unmeasured
- [[agora-git-shared-memory-collective-autoresearch-zhang-2026]] — Agora (NVIDIA): a Git DAG as shared memory for 13 autoresearch agents; 98% of the gain came in the first 6 hours
