---
title: "CRAFT: Customizing LLMs by Creating and Retrieving from Specialized Toolsets"
type: source
created: 2026-05-29
updated: 2026-05-29
sources:
  - craft-yuan-2023.md
arxiv_id: "2309.17428"
venue: ICLR 2024
authors:
  - Lifan Yuan
  - Yangyi Chen
  - "[[xingyao-wang]]"
  - Yi Fung
  - "[[hao-peng]]"
  - "[[heng-ji]]"
first_author: Lifan Yuan
year: 2023
introduces:
  - "[[tool-creation]]"
tags:
  - 2023
status: complete
importance: medium
---

# CRAFT: Customizing LLMs by Creating and Retrieving from Specialized Toolsets

[arXiv:2309.17428](https://arxiv.org/abs/2309.17428) · ICLR 2024 · Lifan Yuan, Yangyi Chen, Xingyao Wang, Yi R. Fung, Hao Peng, Heng Ji (UIUC).

## Summary

CRAFT makes the **create-then-retrieve skill-library loop** explicit: rather than augmenting an LLM with general-purpose APIs at inference time, it builds a task-curated toolset **offline** through a four-stage pipeline — **generate → abstract → validate → deduplicate** — and then **retrieves** the relevant code-snippet tools at inference. Tools are created by prompting GPT-4 to solve sampled training problems in Python; correct solutions are abstracted into reusable functions (specific variable names generalized, hard-coded inputs lifted to arguments, a general function name and docstring attached), validated by re-solving the original problems, and deduplicated by grouping on function name and argument count. The result is a large, diverse, correct toolset (>100 tools, "theoretically unlimited") that customizes an off-the-shelf LLM to a new domain or modality **without any fine-tuning** — a training-free, plug-and-play approach.

CRAFT is best read as the **retrieval half of the skill-library pattern made rigorous**, complementing the creation half emphasized by [[llm-as-tool-makers-cai-2023|LATM]] (which builds a single tool from a handful of examples and applies it to all test cases) and CREATOR (which builds an unverified, non-reusable tool per query). At inference, CRAFT prompts the LLM to "describe what it needs" — emitting a candidate function name and docstring for the target problem — then performs **multi-view matching**: it retrieves separately by problem similarity, function-name similarity, and docstring similarity (all via SimCSE embeddings), aggregates the three candidate lists by frequency, and keeps the tools selected by majority vote. Retrieved snippets are inserted into the prompt and invoked from generated code. On vision-language QA, tabular processing, and mathematical reasoning, CRAFT beats strong baselines — e.g. **+10.4 / +18.0 / +15.2 absolute soft-accuracy** over ViperGPT on GQA / OK-VQA / A-OKVQA, and an average ~43% relative F1 gain on VQA. Authorship overlaps heavily with [[codeact-wang-2024|CodeAct]] and [[openhands-wang-2024|OpenHands]] — this is the same UIUC Heng Ji group, and CRAFT is an early statement of their "code as the agent's medium" line of work.

## Key Points

- **Offline toolset construction pipeline (generate → abstract → validate → deduplicate).** Iteratively samples diverse problems from a source dataset using a min-max strategy (`argTopKmin` over max SimCSE similarity to the current set), prompts GPT-4 for executable Python solutions, and discards any that don't produce the correct answer. The toolset is built once, offline, at an estimated ~$2,500 total construction cost.
- **Abstraction for reusability.** GPT-4 rewrites each correct solution into a general-purpose tool: specific variable names are generalized (e.g. `cat → animal`, `desk → object`), hard-coded textual inputs are lifted into function arguments (e.g. `df["date"] → df[column_name]`), and a suitable function name plus docstring are generated. Ablating abstraction causes a clear performance drop, confirming its importance.
- **Validation and deduplication.** Validation re-runs each abstract tool against its originating problem and discards tools that fail. Deduplication groups tools by (function name, argument count) and prompts GPT-4 to keep the single most broadly applicable tool per group, reducing redundancy and naming collisions.
- **Inference-time multi-view retrieval.** The LLM first generates a target function name and docstring. CRAFT retrieves top-k tools three ways — by problem (`q`), name (`f`), and docstring (`d`) similarity (SimCSE) — then aggregates by occurrence frequency and majority-votes the final tools, filtering singletons. If no tool recurs, the model falls back to plain code generation. Ablations show **function name** is the single most important retrieval view (>6.6 absolute SAcc drop when removed).
- **Scales with toolset size and backbone capability.** Soft accuracy rises monotonically as the toolset grows (0 → 261 → 337 → 525 tools), and CRAFT also improves a GPT-4 backbone, though gains are smaller than for GPT-3.5-Turbo (a GPT-4-built toolset offers itself fewer novel insights).
- **Tools are atomic and low-complexity.** Average cyclomatic complexity 1.34–2.64 (well under the "≤10" software-quality threshold); Louvain community detection finds many distinct tool classes (195/23/234 classes across 525/181/282 VQA/tabular/math tools), indicating well-structured, reliable, diverse tools.
- **Training-free, plug-and-play customization.** No fine-tuning of the backbone; the released toolsets are reusable across downstream tasks. Implemented on GPT-3.5-Turbo-0613 (optimized for tool learning) because weaker code models like CodeLlama performed near-random in the setting.
- **Shares authors with [[codeact-wang-2024|CodeAct]] and [[openhands-wang-2024|OpenHands]].** Lifan Yuan, Xingyao Wang, Hao Peng, and Heng Ji are common authors — the UIUC Heng Ji group's broader program of treating code as the unifying substrate for tool use and agent action.

## Entities

- [[uiuc]] — University of Illinois Urbana-Champaign; all six authors' affiliation (first author during a UIUC internship).
- [[heng-ji]] — senior author; UIUC.
- [[xingyao-wang]] — co-author; also first author of [[codeact-wang-2024|CodeAct]] / [[openhands-wang-2024|OpenHands]].
- [[hao-peng]] — co-author; UIUC.

## Concepts

- [[tool-creation]] — the paradigm this paper advances; CRAFT contributes the offline-construction-plus-retrieval formulation.
- [[skill-library]] — CRAFT's task-curated toolset is a concrete skill library; this paper makes the retrieval half of that pattern rigorous.
- [[agent-skills]] — created tools are reusable agent skills (atomic, named, documented code snippets).
- [[retrieval-augmented-generation]] — CRAFT augments generation by retrieving relevant tools (not documents) from a constructed store at inference.
- [[tool-use]] — CRAFT is a tool-use method; it departs from fixed general-purpose APIs by curating task-specific ones.
- [[llm-as-tool-makers-cai-2023|LATM]] — closest prior tool-creation work; CRAFT generalizes from a single example-built tool to a large, validated, retrievable toolset.

## Notable Quotes

> "In this work, we present CRAFT, a general tool creation and retrieval framework for LLMs. It creates toolsets specifically curated for the tasks and equips LLMs with a component that retrieves tools from these sets to enhance their capability to solve complex tasks."

> "For each task, we collect specific code solutions by prompting GPT-4 to solve the training examples. Following a validation step ensuring the correctness, these solutions are abstracted into code snippets to enhance reusability, and deduplicated for higher quality. At inference time, the language model retrieves snippets from the toolsets and then executes them or generates the output conditioning on the retrieved snippets."

> "Our method is designed to be flexible and offers a plug-and-play approach to adapt off-the-shelf LLMs to unseen domains and modalities, without any finetuning."

> "For better retrieval outcomes, we prompt the LLM to 'describe what it needs'. During inference, the evaluated LLM is asked to generate the function names and the docstrings based on the target problem ... we conduct multi-view matching, searching tools via [problem, name, docstring] respectively in the toolset."

## References

- Source PDF: `sources/craft-yuan-2023.md`
- arXiv: <https://arxiv.org/abs/2309.17428>
- Project / code / toolsets: <https://github.com/lifan-yuan/CRAFT>
- Venue: Published as a conference paper at ICLR 2024.
