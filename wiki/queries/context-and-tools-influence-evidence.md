---
title: "Context and Tools: Influence Evidence"
type: query
created: 2026-11-01
updated: 2026-11-01
sources:
  - rag-lewis-2020.md
  - nakano-webgpt-2021.md
  - effective-context-engineering-anthropic-2025.md
  - advanced-tool-use-anthropic-2025.md
  - claude-code-latent-space-2025.md
status: complete
importance: high
---

# Context and Tools: Influence Evidence

Continues [[ai-agent-influence-graph]] with the same grading. Page numbers are PDF pages of local copies. Absence means not found in what was checked, not that no influence happened.

## Dates

| Item | Date | Source |
|---|---|---|
| [[bm25]] | 1995 cited form (Robertson et al., via Toolformer) | [[bm25]] |
| [[drqa-chen-2017|DrQA]] | 2017-03 | [arXiv:1704.00051](https://arxiv.org/abs/1704.00051) |
| [[realm]] | 2020-02 | [arXiv:2002.08909](https://arxiv.org/abs/2002.08909) |
| [[dpr]] | 2020-04 | [arXiv:2004.04906](https://arxiv.org/abs/2004.04906) |
| [[rag]] | 2020-05 | [arXiv:2005.11401](https://arxiv.org/abs/2005.11401) |
| [[openai-codex|Codex]] | 2021-07 | [arXiv:2107.03374](https://arxiv.org/abs/2107.03374) |
| [[github-copilot]] | 2021-06 | [GitHub](https://github.blog/2021-06-29-introducing-github-copilot-ai-pair-programmer/) |
| [[retro]] | 2021-12 | [arXiv:2112.04426](https://arxiv.org/abs/2112.04426) |
| [[mrkl]] | 2022-05 | [arXiv:2205.00445](https://arxiv.org/abs/2205.00445) |
| [[talm]] | 2022-05 | [arXiv:2205.12255](https://arxiv.org/abs/2205.12255) |
| [[atlas]] | 2022-08 | [arXiv:2208.03299](https://arxiv.org/abs/2208.03299) |
| [[pal-gao-2023|PAL]] | 2022-11 | [arXiv:2211.10435](https://arxiv.org/abs/2211.10435) |
| [[llamaindex]] | 2022-11 (repo) | [GitHub](https://github.com/run-llama/llama_index) |
| [[hyde]] | 2022-12 | [arXiv:2212.10496](https://arxiv.org/abs/2212.10496) |
| [[code-interpreter]] | 2023-03 | [OpenAI](https://openai.com/blog/chatgpt-plugins) |
| [[aider]] | 2023-05 (repo) | [GitHub](https://github.com/Aider-AI/aider) |
| [[gorilla]] | 2023-05 | [arXiv:2305.15334](https://arxiv.org/abs/2305.15334) |
| [[toolllm]] | 2023-07 | [arXiv:2307.16789](https://arxiv.org/abs/2307.16789) |
| [[memgpt]], [[letta]] | 2023-10 | [arXiv:2310.08560](https://arxiv.org/abs/2310.08560) |
| [[self-rag]] | 2023-10 | [arXiv:2310.11511](https://arxiv.org/abs/2310.11511) |
| Parallel function calls | 2023-11 | [OpenAI](https://openai.com/index/new-models-and-developer-products-announced-at-devday/) |
| [[corrective-rag]] | 2024-01 | [arXiv:2401.15884](https://arxiv.org/abs/2401.15884) |
| [[graphrag]] | 2024-04 | [arXiv:2404.16130](https://arxiv.org/abs/2404.16130) |
| Anthropic tool use GA | 2024-05 | [Anthropic](https://www.anthropic.com/news/tool-use-ga) |
| [[structured-outputs]] | 2024-08 | [OpenAI](https://openai.com/index/introducing-structured-outputs-in-the-api/) |
| [[late-chunking]] | 2024-09 | [arXiv:2409.04701](https://arxiv.org/abs/2409.04701) |
| [[contextual-retrieval]] | 2024-09 | [Anthropic](https://www.anthropic.com/engineering/contextual-retrieval) |
| Writing tools guide | 2025-09 | [Anthropic](https://www.anthropic.com/engineering/writing-tools-for-agents) |
| [[effective-context-engineering-anthropic-2025|Anthropic context engineering]] | 2025-09 | [[effective-context-engineering-anthropic-2025]] |
| [[programmatic-tool-calling]], [[tool-search]] | 2025-11 | [Anthropic](https://www.anthropic.com/engineering/advanced-tool-use) |
| [[llm-wiki]] | 2026-04 | [Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) |

## Coding-agent lineage

| Relation | Grade | Evidence |
|---|---|---|
| Claude Code origin | First-hand, no influence named | [Latent Space](https://www.latent.space/p/claude-code) 00:02:04: an experiment through the public API |
| Memory literature and products → CLAUDE.md | Weak, developer mention | 00:09:36: wanted to be inspired, then shipped a simple file; none is named |
| Cursor rules ~ CLAUDE.md | Weak, interviewer's comparison | 00:09:26; not the team's claim |
| Cursor, Devin, Aider → Claude Code | Not confirmed | 00:10:39: used and liked, compared only |
| MCP → Claude Code | Confirmed use | [Changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md): first MCP mention is 0.2.31 (npm 2025-03-05). Earlier support may have gone unlisted. Remote transports came 2025-06-18 per [release notes](https://docs.anthropic.com/en/release-notes/claude-code) |
| Skills → Claude Code | Confirmed use | [[claude-code]] timeline: 2.0.20, 2025-10-16; [Anthropic](https://claude.com/blog/skills) |
| ReAct, SWE-agent, Aider → Claude Code | Not confirmed | No such statement found |
| Any named product → Codex CLI | Not confirmed | The launch section and README name none |
| Between coding-agent products | Not confirmed | No first-party statement found. Shared features are not evidence of direction |
| Codex → Copilot | Confirmed use | Codex p. 1; GitHub post |

## RAG lineage

| A → B | Grade | Where |
|---|---|---|
| DPR → RAG | Confirmed use | Lewis p. 2: its retriever |
| DrQA → DPR | Weak, framing | Karpukhin p. 1 |
| DrQA → REALM | Weak, prior work | Guu p. 2 |
| RAG, REALM → RETRO | Weak, prior work | Borgeaud p. 1 |
| REALM, RAG, RETRO → Atlas | Weak, prior work | Izacard p. 1 |
| DPR → HyDE | Weak, background | Gao p. 1 |
| InstructGPT → HyDE | Confirmed use | Abstract names it as example model |
| RAG, REALM → Self-RAG | Weak, definition | Asai pp. 1–2 |
| RAG, Self-RAG → CRAG | Confirmed use | Yan p. 2: implemented into both |
| RAG → GraphRAG | Weak, definition | Edge p. 1 |
| GraphRAG → LangChain, LlamaIndex | Confirmed adoption | Edge p. 2 |
| BM25 → Contextual Retrieval | Confirmed use | Anthropic post |
| Contextual Retrieval ~ Late Chunking | Parallel work | Günther p. 3 |
| RAG → LLM Wiki | Contrast | Karpathy gist |

## Tool-use lineage

| A → B | Grade | Where |
|---|---|---|
| TALM → Toolformer | Weak, closest related work | Schick p. 10 |
| Toolformer → Gorilla | Weak, related work | Patil p. 3 |
| Gorilla → ToolLLM | Benchmark comparison | Qin p. 3 |
| Gorilla → BFCL | Same team | BFCL page |
| PAL ~ Code Interpreter | Not confirmed | Same idea; no link checked |
| MRKL → LangChain | Weak, third party | [Issue #2284](https://github.com/langchain-ai/langchain/issues/2284), a user's report |
| Plugins → function calling → tool use → MCP | Not confirmed | Date order only |
| Function calling → parallel calls, structured outputs | Confirmed extension | OpenAI posts |
| MCP → Advanced tool use | Weak, stated motive | Post discusses large tool sets and code execution with MCP |
| Codex/Copilot → tool calling | Not confirmed | Code generation is a different function; treat as a precondition only |

## Where retrieval and tools meet

| Evidence | Grade | Where |
|---|---|---|
| WebGPT sets itself against document retrieval | Confirmed contrast | Nakano p. 3: REALM and RAG improve retrieval for a given query; WebGPT uses browsing actions instead |
| ReAct searches through a Wikipedia API | Confirmed use | Yao p. 1 |
| Toolformer's search tool is a BM25 retriever | Confirmed use | Schick p. 4 |
| Gorilla pairs API calling with retrieval | Confirmed use | Patil p. 1 |
| MemGPT chains function calls for multi-step retrieval | Confirmed use | Packer p. 3 |
| Anthropic Tool Search offers BM25 search over tools | Confirmed use | [Advanced tool use](https://www.anthropic.com/engineering/advanced-tool-use): retrieval applied to tool definitions |
| Retrieval as a tool call | Named concept | [[agentic-search]], [[grep-all-you-need-sen-2026]] |
| Claude Code dropped RAG for agentic search | Confirmed, developer statement | [[cherny-agentic-search-statements]] |

## Codebase RAG to agentic search: grading

| Claim | Grade | Evidence |
|---|---|---|
| Claude Code tried RAG, then chose agentic search | **Confirmed, developer statement** | [[cherny-agentic-search-statements]]: HN 2025-02-24, Latent Space 00:45:30, X 2026-02-01 |
| Agentic search performed better for Claude Code | **Weak** | Stated, but "mostly vibes" plus unpublished benchmarks |
| Indexing brings staleness and security costs | Stated reasons | Latent Space 00:46:20; X reply |
| Semantic search beats grep alone in Cursor | Vendor measurement, Nov 2025 | [[cursor-semantic-search-blog-2025]] |
| Cursor moved away from stored embeddings | **Documentation change only** | [[cursor-docs-search-history]]: between 2026-07-19 and 2026-09-21; no reason found |
| Claude Code → Cursor's change | **Not confirmed** | No statement found; date order only |
| An industry-wide shift from RAG to agentic search | **Not established** | Two products with different evidence. Cursor advocated embeddings until at least March 2026. Aider's [[repo-map]] is a third path |
| Grep beats vector search in agents generally | **Not established** | [[grep-all-you-need-sen-2026]] tested inline grep above vector in 10 of 10 harness-model pairs, but on 116 chat-memory questions, not code. Programmatic delivery favoured vector in 5 of 10, and harness differences outweighed retriever differences |

"Agentic search" is also not the opposite of RAG: retrieval still feeds generation.

## Where both merge into context engineering

The firmest evidence is Anthropic's own wording, so this is one vendor's framing, not a field-wide finding.

- **Tools bring context:** [[effective-context-engineering-anthropic-2025]] says tools let agents pull in new context as they work.
- **Retrieval moves into tools:** the same post describes a shift from embedding-based pre-inference retrieval to "just in time" loading through tools.
- **Hybrid case:** it describes Claude Code as CLAUDE.md up front, with glob and grep at run time.
- **Tool design as context cost:** [[writing-tools-for-agents-anthropic-2025]] and [[advanced-tool-use-anthropic-2025]] judge tools by context used.
- **Second builder:** [[context-engineering-lessons-manus-ji-2025]] manages tool definitions and KV-cache as context problems.

## Not ingested

Remaining: Cursor's first release date and any statement on why its docs changed, ToolBench as its own page, LangChain RAG docs, and Letta's own materials. The MemGPT, compaction, prompt-chaining and Agent Skills pages already exist.
