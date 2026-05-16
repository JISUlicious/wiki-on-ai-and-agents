---
title: "MemGPT: Towards LLMs as Operating Systems"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - memgpt-packer-2023.md
arxiv_id: "2310.08560"
venue: COLM 2024
authors:
  - "[[charles-packer]]"
  - Sarah Wooders
  - Kevin Lin
  - Vivian Fang
  - Shishir G. Patil
  - "[[ion-stoica]]"
  - Joseph E. Gonzalez
first_author: "[[charles-packer]]"
year: 2023
introduces:
  - "[[memgpt]]"
  - "[[virtual-context]]"
  - "[[tiered-memory]]"
tags:
  - 2023
status: complete
importance: high
---

# MemGPT: Towards LLMs as Operating Systems

- **Source**: `sources/memgpt-packer-2023.md`
- **arXiv**: [arXiv:2310.08560](https://arxiv.org/abs/2310.08560)
- **Venue**: COLM 2024
- **Authors**: [[charles-packer]], Sarah Wooders, Kevin Lin, Vivian Fang, Shishir G. Patil, [[ion-stoica]], Joseph E. Gonzalez
- **Affiliation**: [[uc-berkeley]]

## Summary

MemGPT applies the **operating-system virtual-memory metaphor** to LLMs: rather than try to grow the context window, treat the fixed window as scarce *physical memory* and page data in and out of it on demand. The paper proposes **virtual context management**, in which the LLM itself orchestrates movement of information between a small in-context region (main memory / RAM) and external stores (disk), giving the illusion of an effectively unbounded context while still running on a finite-window model like GPT-4.

The architecture defines three storage tiers. **Main context** (the prompt) is split into a read-only system instruction block, a small read/write *working context* block (persona, key user facts, top-priority state), and a **FIFO queue** of recent messages whose head holds a recursive summary of evicted history. **Recall storage** is a database of the full message history — warm storage searchable by function call. **Archival storage** is a read/write text database for long-term documents and facts — cold storage with explicit insert/search operations. Movement between tiers is driven entirely by the LLM emitting **function calls** (`archival_memory_insert`, `archival_memory_search`, `recall_memory_search`, `working_context.append`, `working_context.replace`, ...). The model thus acts as its own *OS scheduler* over memory, deciding what to evict, what to retrieve, and when to consolidate.

A **queue manager** enforces context-window discipline: when prompt tokens cross a "warning token count" (e.g. 70% of the window), it inserts a `memory pressure` system message so the LLM can proactively persist important state via function calls. At the "flush token count" it evicts a chunk of the queue, regenerates the recursive summary, and writes evicted messages to recall storage. Control flow uses **function chaining** via a `request_heartbeat=true` flag so the LLM can sequence multiple memory operations (paginate, retrieve, write) before yielding back to the user.

The system is evaluated in two long-context domains: **multi-session conversational agents** (extended MSC dataset; deep memory retrieval and conversation-opener tasks) and **document analysis** (long-document QA, key-value retrieval, and a new nested-KV multi-hop task). MemGPT substantially beats fixed-context GPT-3.5/GPT-4/GPT-4-Turbo baselines that rely on lossy recursive summarization — e.g. DMR accuracy jumps from 32.1% (GPT-4) to 92.5% (GPT-4 + MemGPT). MemGPT is the **direct conceptual ancestor of [[letta]] and of essentially every "memory-managed agent" pattern that proliferated in 2024–2025**.

## Key Points

- **Virtual-context paradigm**: treat the context window as a constrained memory resource and provide the illusion of unbounded context via OS-style paging between tiers.
- **Three-tier memory hierarchy**: main context (system prompt + working context + FIFO queue) is hot; recall storage (chat history DB) is warm; archival storage (arbitrary-length text DB) is cold.
- **LLM-issued function calls drive paging**: write, store, search, page, and evict are all *explicit* operations the model emits — not implicit retrieval. This makes memory management observable, auditable, and self-directed.
- **Main context layout**: read-only system instructions describe the memory hierarchy and the function schema; a small writable working-context block holds persona + key user facts; the FIFO queue carries recent messages with a recursive summary at the head.
- **Memory-pressure warning policy**: queue manager inserts system warnings as the context approaches its limit, prompting the LLM to persist important state before eviction; at the flush threshold it evicts messages to recall storage and updates the recursive summary.
- **Function chaining via `request_heartbeat`** lets the LLM sequence multi-step memory ops (paginate, search, write) within a single user turn.
- **Event-driven control flow**: user messages, system alerts, scheduled timers, and tool callbacks all act as events that trigger inference — enabling unprompted, agentic behavior.
- **Pagination of retrieval results** prevents large search hits from blowing the window.
- **Empirical wins on long-context tasks**: outperforms GPT-4 / GPT-4-Turbo baselines on MSC deep memory retrieval, conversation openers, and document QA despite using the same finite window.

## Entities Mentioned

- [[charles-packer]] (first author)
- [[ion-stoica]]
- [[uc-berkeley]]
- [[gpt-4]] (primary backbone for experiments; also GPT-4 Turbo and GPT-3.5 Turbo)

## Concepts Discussed

- [[memgpt]]
- [[virtual-context]]
- [[tiered-memory]]
- [[memory-management]]
- [[llm-agent]]
- [[function-calling]]

## Notable Quotes

> "We propose virtual context management, a technique drawing inspiration from hierarchical memory systems in traditional operating systems which provide the illusion of an extended virtual memory via paging between physical memory and disk."

> "In MemGPT, we treat context windows as a constrained memory resource, and design a memory hierarchy for LLMs analogous to memory tiers used in traditional OSes."

## References

_Original source: `sources/memgpt-packer-2023.md`_
