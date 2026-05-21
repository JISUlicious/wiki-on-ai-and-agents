---
title: "SWE-bench: Can Language Models Resolve Real-World GitHub Issues?"
type: source
created: 2026-05-21
updated: 2026-05-21
sources:
  - swe-bench-jimenez-2023.md
arxiv_id: "2310.06770"
venue: ICLR 2024
authors:
  - Carlos E. Jimenez
  - John Yang
  - Alexander Wettig
  - Shunyu Yao
  - Kexin Pei
  - Ofir Press
  - Karthik Narasimhan
first_author: Carlos E. Jimenez
year: 2023
introduces:
  - "[[swe-bench]]"
tags:
  - 2023
status: complete
importance: high
---

# SWE-bench: Can Language Models Resolve Real-World GitHub Issues?

[arXiv:2310.06770](https://arxiv.org/abs/2310.06770) — Published as a conference paper at ICLR 2024.

## Summary

This paper introduces [[swe-bench]], the foundational benchmark for evaluating language models as software engineers on real-world coding tasks. Rather than the self-contained, function-level problems of prior coding benchmarks like HumanEval, SWE-bench frames each task as a real GitHub issue paired with the merged pull request that resolved it. The model is given the issue text plus a snapshot of the repository, and must produce a patch that, when applied, makes the project's previously-failing tests pass without breaking the existing test suite. The benchmark comprises **2,294 task instances** distilled from ~90,000 PRs across **12 popular Python repositories** (Django, sympy, scikit-learn, matplotlib, sphinx, pytest, xarray, astropy, pylint, requests, flask, seaborn).

At release, frontier models performed dismally: the best baseline, Claude 2 with BM25 retrieval, resolved only **1.96%** of issues; GPT-4 and ChatGPT-3.5 fared similarly poorly. Even with "oracle" retrieval (the files actually edited by the reference PR handed to the model), results stayed in low single digits. The authors also released **SWE-Llama** (fine-tuned 7B/13B CodeLlama variants) and a 19,000-instance training set, **SWE-bench-train**, drawn from 37 additional repositories disjoint from the eval set. A **SWE-bench Lite** subset of 300 self-contained instances was carved out to lower the cost of entry.

SWE-bench rapidly became the de facto LM-as-software-engineer evaluation. Its construction pipeline (scrape PRs → filter for issue-resolving PRs that contribute tests → execution-filter on fail-to-pass tests) seeded an entire research thread, including the contamination-resistant successor [[swe-bench-live]] and the training-environment companion [[swe-gym]]. The benchmark also catalyzed the **SWE-bench Verified** subset (human-validated tasks) that emerged later, and drove much of the 2024 wave of agentic coding work.

## Key Points

- **2,294 task instances** from 12 popular Python repositories, distilled from ~90,000 candidate PRs via a 3-stage pipeline (scrape PRs → attribute filter → execution filter).
- **Task formulation**: given (issue text, codebase snapshot) → produce a patch file. Evaluated by `patch` + running the repository's existing test suite. Success requires at least one **fail-to-pass** test to flip AND prior tests to continue passing.
- **Scale of context**: codebases average 3,010 non-test files and 438K lines; the reference "gold" patch edits 1.7 files / 3.0 functions / 32.8 lines on average. A median 51 additional tests guard against regression.
- **Continually updatable**: the pipeline is automatic and can ingest new PRs from any Python repo, enabling evaluation on post-training-cutoff issues to mitigate contamination.
- **Baseline results** with BM25 retrieval at 13k context: Claude 2 = 1.96%, ChatGPT-3.5 < 1%, GPT-4-turbo ~2%, Claude 3 Opus = 3.79%, SWE-Llama 13b = 0.70%. All models top out below 4% on the full benchmark.
- **SWE-Llama**: 7B and 13B CodeLlama-Python models fine-tuned with LoRA on attention weights, trained on a separate 19K-instance corpus from 37 disjoint repos. Competitive with Claude 2 in some settings; processes 100K+ token contexts.
- **Retrieval matters**: BM25 retrieves a superset of oracle files in only ~40% of cases at 27k context, and retrieves *none* of the oracle files in nearly half — establishing retrieval as a major bottleneck.
- **SWE-bench Lite**: 300-instance self-contained subset covering 11 of the 12 repos, intended as a cheaper on-ramp.
- **Cross-context editing**: unlike HumanEval-style cloze tasks, SWE-bench has no localized edit window; models must find the right lines among hundreds of thousands.

## Entities

- [[carlos-jimenez]] — first author, Princeton
- [[john-yang]] — co-first author, Princeton
- [[alexander-wettig]] — Princeton
- [[shunyu-yao]] — Princeton (also of [[react]] and Reflexion fame)
- [[ofir-press]] — Princeton
- [[karthik-narasimhan]] — Princeton (senior author)
- [[kexin-pei]] — University of Chicago
- [[princeton-university]] — primary affiliation
- [[princeton-language-and-intelligence]] — sub-institute hosting the work
- [[university-of-chicago]] — Pei's affiliation
- [[swe-llama]] — fine-tuned CodeLlama variants released alongside the benchmark

## Concepts

- [[swe-bench]] — the canonical benchmark this paper defines
- [[swe-bench-lite]] — 300-instance subset
- [[swe-bench-live]] — contamination-resistant successor benchmark
- [[swe-gym]] — training environment that builds on SWE-bench-style task construction
- [[llm-agent]] — SWE-bench became the proving ground for agentic coding systems
- [[code-generation]] — the broader task family SWE-bench reframes at the repo level
- [[agent-benchmark]] — SWE-bench is the prototype repo-scale agent benchmark
- [[bm25]] — sparse retrieval used for the baseline context-selection setup
- [[codellama]] — base model for SWE-Llama
- [[lora]] — parameter-efficient fine-tuning used for SWE-Llama
- [[humaneval]] — the prior coding benchmark SWE-bench contrasts itself against

## Quotes

> "We introduce SWE-bench, an evaluation framework consisting of 2,294 software engineering problems drawn from real GitHub issues and corresponding pull requests across 12 popular Python repositories." — Abstract

> "Resolving issues in SWE-bench frequently requires understanding and coordinating changes across multiple functions, classes, and even files simultaneously, calling for models to interact with execution environments, process extremely long contexts and perform complex reasoning that goes far beyond traditional code generation tasks." — Abstract

> "The best-performing model, Claude 2, is able to solve a mere 1.96% of the issues. Advances on SWE-bench represent steps towards LMs that are more practical, intelligent, and autonomous." — Abstract

> "Our collection process can be easily applied to any Python repository on GitHub and requires minimal human intervention. Therefore, we can extend SWE-bench with a continual supply of new task instances and evaluate LMs on issues created after their training date, which ensures that the solution was not included in their training corpus." — Section 2.3

> "The task of repository-scale code editing can serve as a level playing field to compare approaches ranging from retrieval and long-context models to decision-making agents, which could reason and act in code." — Section 2.3

## References

- Source file: `swe-bench-jimenez-2023.md` in `sources/`
- arXiv: [2310.06770](https://arxiv.org/abs/2310.06770) (v3, Nov 2024)
- Venue: ICLR 2024
- Leaderboard and data: [swebench.com](https://swebench.com)
