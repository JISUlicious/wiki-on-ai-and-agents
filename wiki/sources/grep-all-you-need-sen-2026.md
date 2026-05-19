---
title: "Is Grep All You Need? How Agent Harnesses Reshape Agentic Search"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - grep-all-you-need-sen-2026.md
arxiv_id: "2605.15184"
authors:
  - Sahil Sen
  - Akhil Kasturi
  - Elias Lumer
  - Anmol Gulati
  - Vamse Kumar Subbiah
first_author: Sahil Sen
year: 2026
introduces:
  - "[[agentic-search]]"
  - "[[chronos]]"
tags:
  - 2026
status: complete
importance: high
---

# Is Grep All You Need? How Agent Harnesses Reshape Agentic Search

- **arXiv**: [2605.15184](https://arxiv.org/abs/2605.15184) (v1, May 14 2026)
- **Authors**: Sahil Sen, Akhil Kasturi, Elias Lumer, Anmol Gulati, Vamse Kumar Subbiah (all [[pwc|PricewaterhouseCoopers, U.S.]])
- **Source PDF**: `sources/grep-all-you-need-sen-2026.pdf`

## Summary

An empirical study of how **retrieval strategy** (grep vs. vector), **agent harness** (custom vs. provider-native CLI), and **tool-result delivery mode** (inline-stdout vs. file-based) jointly determine end-to-end agentic-search accuracy. The headline finding inverts the conventional RAG wisdom: with inline tool delivery, **lexical search (grep) beats dense vector retrieval for every harness-model pair tested**, contradicting the assumption that semantic similarity is strictly more powerful than exact-match. The deeper finding is that **harness choice shifts accuracy by as much as retriever choice does** — the same Claude Opus 4.6 backbone hits **93.1% on Chronos but 76.7% on Claude Code** with grep-only retrieval. Result delivery mode (inline vs. file-based) can also invert the lexical advantage without changing the corpus.

Position in the wiki: this paper sits at the intersection of [[memory-management]] (long-horizon QA), [[retrieval-augmented-generation]] (the retrieval substrate), [[function-calling]] / [[tool-use]] (how agents call retrieval), and [[agent-three-layer-model]] (the harness L2 abstraction is the variable being studied). It is one of the first systematic measurements that treats **retriever + harness + delivery mode as a single jointly-evaluated system** rather than three independent design choices.

## Setup

- **Benchmark**: 116-question subset of [[longmemeval]] (Wu et al., ICLR 2025) — long-term-memory QA over multi-session chat corpora, 6 information-retrieval categories.
- **Harnesses**:
  - **Chronos** (custom; LangChain-based, authors' prior work, [[chronos|arXiv:2603.16862]]) — fine-grained control over tool-calling loop, context construction, and result formatting.
  - **Provider-native CLI agents**: [[claude-code|Claude Code]] (Anthropic), Codex CLI (OpenAI), Gemini CLI (Google) — shell-based interfaces where the model has direct access to `grep`, `cat`, etc.
- **Retrievers**: grep (lexical pattern match) and vector search (top-15 by cosine similarity).
- **Delivery modes**:
  - **Inline (standard)** — tool stdout appended to the agent's context.
  - **Programmatic (file-based)** — results written to a file the agent must explicitly read.
- **Backbones tested**: Claude Opus 4.6, Claude Haiku 4.5, GPT-5.4, Gemini 3.1 Pro, Gemini 3.1 Flash-Lite.
- **Grader**: GPT-4o.

## Key results

### Experiment 1 — Retrieval × harness × delivery

| Configuration                                | Headline |
|---|---|
| Inline grep > inline vector                  | Holds for **every** harness-model pair tested (10/10) |
| Largest grep advantage (inline)              | Chronos + Gemini 3.1 Flash-Lite: **86.2% vs. 62.9%** |
| Narrowest grep advantage (inline)            | Claude Code + Claude Opus 4.6: 76.7% vs. 75.0% |
| Top inline grep score                        | Chronos + Claude Opus 4.6 = **93.1%**; tied by Codex + GPT-5.4 |
| Harness effect on identical backbone         | Claude Opus 4.6: 93.1% (Chronos) vs. 76.7% (Claude Code) — **harness gap > retriever gap** |
| Programmatic delivery flips the picture      | Programmatic vector > programmatic grep on **5 of 10** harness-model pairs |
| Sharpest regression                          | Codex + GPT-5.4: **93.1% inline grep → 55.2% programmatic grep** |

### Experiment 2 — Context scaling under corpus noise

Mixes increasing amounts of unrelated chat history into each haystack to measure how each retriever degrades as the noise-to-signal ratio grows. Specific finding: grep and vector degrade at different rates; the relative ordering depends on harness as well.

## Implications

1. **Lexical search is undersold in agentic-search literature.** With inline tool calling, grep / regex / BM25-style retrieval is consistently stronger than dense retrieval on long-horizon chat QA. The paper attributes this to (a) the agent's iterative refinement loop being more compatible with exact match (issue queries → inspect snippets → re-query), and (b) vector top-k often returning syntactically-near-but-semantically-irrelevant passages that look plausible to the LLM grader.

2. **Tool-result delivery mode is a load-bearing design choice.** Inline-stdout vs. file-based isn't an aesthetic preference — it can erase, preserve, or reverse the retriever ranking. This is consistent with the broader observation that the [[agent-three-layer-model|L2 harness layer]] mediates all of an agent's perception.

3. **Provider-native CLI harnesses differ structurally from custom SDK harnesses.** Claude Code / Codex / Gemini CLI ship with shell-mediated tool calling and pre-installed `grep`/`cat`/etc.; the same retrieval *strategy* runs differently inside their loops. The paper argues these should be evaluated as a distinct class of agent harnesses rather than collapsed under "agent with tools."

4. **Score variance from harness ≈ score variance from retriever.** Both axes matter roughly equally in this benchmark. RAG ablation studies that fix the harness obscure half the action.

## Entities mentioned

- [[pwc]] (PricewaterhouseCoopers) — the authors' institution.
- [[anthropic]] — Claude Code, Claude Opus 4.6, Claude Haiku 4.5.
- [[openai]] — Codex CLI, GPT-5.4.
- [[google]] — Gemini CLI, Gemini 3.1 Pro / Flash-Lite.

## Concepts discussed

- [[agentic-search]] — the umbrella concept this paper concretizes.
- [[retrieval-augmented-generation]] — the substrate; this paper studies its in-loop variant.
- [[dense-retrieval]] / [[bm25]] / [[dpr]] — retrieval baselines.
- [[tool-use]] / [[function-calling]] — how agents invoke retrieval.
- [[react]] — the underlying loop pattern.
- [[chronos]] — the authors' custom LangChain-based harness; prior work.
- [[longmemeval]] — the benchmark used.
- [[claude-code]] / [[memgpt]] / [[gorilla]] — cited prior agentic systems.
- [[agent-three-layer-model]] — the harness L2 abstraction this paper measures the impact of.

## Notable quotes

> "Across Chronos and the provider CLIs, grep consistently yields higher accuracy than vector retrieval in our comparisons, with inline grep exceeding inline vector for every harness-model pair we evaluate."

> "The same Claude Opus 4.6 backbone reaches 93.1% under Chronos but 76.7% under Claude Code, so changing the harness shifts the ceiling by roughly as much as swapping retrievers within a fixed harness."

> "These results highlight retrieval mechanics, harness orchestration, and delivery path as a single jointly evaluated system rather than independent design choices."

## References

- [[chronos]] — Sen et al. 2026, [arXiv:2603.16862](https://arxiv.org/abs/2603.16862) — the custom harness used as one of the four agents in this study.
- [[longmemeval]] — Wu et al. ICLR 2025 — the benchmark.
- [[memgpt-packer-2023]] — cited as an early long-horizon memory agent.
- [[react-yao-2022]] — the underlying reason-act loop.
- [[dpr-karpukhin-2020]] — DPR cited as canonical dense-retrieval baseline.
- Canonical link: https://arxiv.org/abs/2605.15184
