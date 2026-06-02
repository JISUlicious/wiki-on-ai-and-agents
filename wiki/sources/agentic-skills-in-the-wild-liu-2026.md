---
title: "How Well Do Agentic Skills Work in the Wild: Benchmarking LLM Skill Usage in Realistic Settings"
type: source
created: 2026-06-02
updated: 2026-06-02
sources:
  - agentic-skills-in-the-wild-liu-2026.md
arxiv_id: "2604.04323"
authors:
  - Yujian Liu
  - Jiabao Ji
  - Li An
  - Tommi Jaakkola
  - Yang Zhang
  - Shiyu Chang
first_author: Yujian Liu
year: 2026
introduces:
  - "[[skill-evaluation]]"
tags:
  - 2026
status: complete
importance: high
---

# How Well Do Agentic Skills Work in the Wild: Benchmarking LLM Skill Usage in Realistic Settings

**Source file**: [[agentic-skills-in-the-wild-liu-2026]]
**arXiv**: [arXiv:2604.04323](https://arxiv.org/abs/2604.04323) (v1, 6 Apr 2026, cs.CL — Preprint, under review)
**Authors**: Yujian Liu*, Jiabao Ji* ([[uc-berkeley|UC Santa Barbara]]), Li An, Shiyu Chang† (UC Santa Barbara); Tommi Jaakkola† ([[mit|MIT CSAIL]]); Yang Zhang† (MIT-IBM Watson AI Lab). *Equal contribution; †Equal advising.
**Code**: [UCSB-NLP-Chang/Skill-Usage](https://github.com/UCSB-NLP-Chang/Skill-Usage)
**Year**: 2026

> [!note] Affiliation note
> The lead institution is **UC Santa Barbara** (Liu, Ji, An, Chang). The wiki currently has no UCSB entity page; [[uc-berkeley]] is used as a placeholder link for the "UC" affiliation and should be replaced once a `uc-santa-barbara` page exists. [[mit|MIT CSAIL]] (Jaakkola) and the MIT-IBM Watson AI Lab (Zhang) are the co-advising institutions.

## Summary

This is **the first comprehensive benchmark of agent _skill usage_ under realistic conditions** — the question of whether [[agent-skills|skills]] still help once an agent must discover, select, and adapt them itself rather than being handed a curated, task-tailored set. Prior skill benchmarks (notably SkillsBench) evaluate under "overly idealized" conditions: skills are hand-crafted to overfit each task — often spelling out a step-by-step solution guide — and are placed directly in the agent's context, bypassing the practical challenge of finding the right skill in a large, noisy collection. Liu et al. close this [[skill-evaluation|skill-evaluation]] gap by assembling a collection of **34,198 real-world skills** scraped from open-source repositories (via skillhub.club and skills.sh), filtered for permissive licenses (MIT/Apache 2.0), quality, and deduplication, then forcing agents to retrieve from that pool instead of from a hand-picked handful.

The headline finding is that **the benefits of skills are fragile**: performance gains degrade consistently as the evaluation setting becomes more realistic, with pass rates approaching the no-skill baseline in the hardest scenario (retrieval from the full 34k pool with curated skills removed). Two bottlenecks drive the collapse — agents struggle to decide which retrieved skills are worth loading (a *selection* failure that leaves helpful skills unused), and retrieved skill content is noisy or lacks the precise information a task needs (an *adaptation* failure). The good news: **query-specific skill refinement substantially recovers the lost performance** when the initially retrieved skills are of reasonable relevance and quality — the agent explores and adapts retrieved skills to the target task at inference time. The authors further show the retrieval + refinement recipe generalizes beyond skill-designed benchmarks: on **Terminal-Bench 2.0** (a general-purpose agent benchmark with no human-curated skills), it lifts **Claude Opus 4.6 from a 57.7% to a 65.5% pass rate**. Results are consistent across multiple models (Claude Opus 4.6, Kimi K2.5, Qwen3.5-397B-A17B), framing skills as a real but currently brittle capability-extension mechanism.

## Key Points

### A realistic skill corpus, not a curated handful
- **34,198 real-world skills** assembled from open-source GitHub repositories, sourced via the skillhub.club and skills.sh aggregation platforms, including each skill's [[skill-md-format|SKILL.md]] and helper files.
- Filtered by permissive license (MIT, Apache 2.0), de-empty-named/described, and deduplicated by content. Spans web development, data engineering, DevOps, scientific computing, and more.
- This is the central methodological move: it replaces the idealized "curated skills placed directly in context" assumption with a large, noisy, realistic pool the agent must search.

### Three challenges the realistic setting reintroduces
1. **Skill selection** — even when relevant skills are available, the agent must recognize and decide to load them, especially amid many irrelevant ones.
2. **Skill retrieval** — users rarely pre-select skills; the agent must search a large repository on its own.
3. **Skill adaptation** — when no skill was authored for the task, the agent must extract useful information from partially-relevant, general-purpose content.

### Retrieval study: agentic hybrid search wins
- Built a skill search engine indexing each skill by (i) metadata (name + description) and (ii) full SKILL.md content, using **Qwen3-Embedding-4B** for [[dense-retrieval|dense embeddings]] and [[bm25|BM25]] for sparse keyword matching.
- Compared direct (single-query) retrieval vs. **[[agentic-search|agentic search]]** where the agent iteratively formulates queries, inspects candidates, and refines its strategy.
- Agentic search beats direct search by **+18.7 Recall@3** points with the same semantic tool. Semantic >> keyword. Adding the full-content index gives a modest consistent bump (Recall@5: 63.5% → 65.5%; Recall@10: 66.7% → 68.3%).
- Default for downstream experiments: **agentic hybrid search with full skill content** (Recall@3 = 57.3%).

### The degradation finding (the core result)
- Six **progressively realistic settings**, ordered idealized → realistic: *Curated + forced load* (upper bound) → *Curated* (agent decides whether to load) → *Curated + distractors* → *Retrieved (w/ curated in pool)* → *Retrieved (w/o curated)* → *No skills* (baseline).
- Pass rates fall monotonically across these settings for every model. Example (Claude Opus 4.6 trajectory in Fig. 1): curated 51.2% → curated+distractors 43.5% → retrieved-w 40.1% → retrieved-w/o 38.4% — converging toward the no-skill baseline.
- Trend holds across Claude Opus 4.6, Kimi K2.5, and Qwen3.5-397B-A17B, each run in its native harness ([[claude-code|Claude Code]] for Claude; Terminus-2 for the open-weight models).

### Query-specific refinement recovers the gains
- Two refinement strategies studied: **query-specific** (agent explores and adapts retrieved skills to the target task at test time) vs. **query-agnostic** (skills improved offline without knowledge of the downstream task).
- **Query-specific refinement substantially recovers lost performance** when retrieved skills are of reasonable quality; gains are limited when no relevant skill exists in the collection at all (you cannot refine your way out of a missing skill).

### Generalization to Terminal-Bench 2.0
- On **Terminal-Bench 2.0** (a general-purpose agent benchmark, no human-curated skills), applying retrieval + refinement raised **Claude Opus 4.6's pass rate from 57.7% to 65.5%** — evidence the approach is not specific to skill-designed benchmarks.

## Entities

- [[mit|MIT CSAIL]] — co-advisor Tommi Jaakkola's institution; with the MIT-IBM Watson AI Lab (Yang Zhang), the academic-industry collaborator alongside UC Santa Barbara.
- [[anthropic|Anthropic]] — originator of the standardized [[agent-skills|agent-skills]] / [[skill-md-format|SKILL.md]] format the benchmark studies; Claude Opus 4.6 is the primary evaluated model.
- [[claude-code|Claude Code]] — the agent harness used for Claude in retrieval and evaluation experiments.

## Concepts

- [[skill-evaluation]] — **introduced/anchored here**: rigorous benchmarking of whether and when skills help agents, under realistic discovery/selection/adaptation conditions rather than idealized hand-fed setups.
- [[agent-skills]] — the reusable, domain-specific knowledge artifacts whose real-world utility is under test.
- [[skill-md-format]] — the structured SKILL.md + helper-file format of the indexed skills.
- [[skill-optimization]] — the refinement angle: query-specific vs. query-agnostic strategies for improving retrieved skills (no page yet; candidate concept).
- [[agent-benchmark]] — this work is a new benchmark methodology; relates to SkillsBench and Terminal-Bench 2.0.
- [[agentic-search]] — the retrieval mechanism shown to dominate: the agent iteratively queries, inspects, and selects skills.
- [[skill-library]] — the 34k-skill collection as a realistic instance of a large, noisy skill library.
- [[dense-retrieval]], [[bm25]] — the embedding-based and sparse-keyword retrieval primitives combined in the hybrid search engine.
- [[self-improving-agent]] — query-specific refinement connects to test-time adaptation / agent self-improvement.

## Quotes

> "Our findings reveal that the benefits of skills are fragile: performance gains degrade consistently as settings become more realistic, with pass rates approaching no-skill baselines in the most challenging scenarios."

> "We show that query-specific refinement substantially recovers lost performance when the initial skills are of reasonable relevance and quality."

> "We further demonstrate the generality of retrieval and refinement on Terminal-Bench 2.0, where they improve the pass rate of Claude Opus 4.6 from 57.7% to 65.5%."

> "Even the closest matching skills may not be well-tailored for the task." — on why adaptation, not just retrieval, is the bottleneck.

> Two bottlenecks limiting skill utility: "❶ Agents struggle to determine which skills are worth loading, leaving potentially helpful skills unused; and ❷ The content of retrieved skills is often noisy or lacks the precise information needed for the task."

## References

- [arXiv:2604.04323](https://arxiv.org/abs/2604.04323) — Liu, Ji, An, Jaakkola, Zhang, Chang. *How Well Do Agentic Skills Work in the Wild: Benchmarking LLM Skill Usage in Realistic Settings* (2026).
- Code/data: [github.com/UCSB-NLP-Chang/Skill-Usage](https://github.com/UCSB-NLP-Chang/Skill-Usage)
- Skill sources: [skillhub.club](https://www.skillhub.club/), [skills.sh](https://skills.sh/)
- Related benchmarks discussed: **SkillsBench** (Li et al., 2026) — the idealized-setting predecessor; **Terminal-Bench 2.0** (Merrill et al., 2026) — the general-purpose agent benchmark used to show generalization.
- Companion wiki source: [[agent-skills-survey-xu-2026]] — the academic survey of the agent-skills paradigm.
