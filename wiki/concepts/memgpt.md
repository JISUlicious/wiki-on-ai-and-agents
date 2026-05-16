---
title: MemGPT
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - memgpt-packer-2023.md
status: complete
importance: high
tags:
  - 2023
---

# MemGPT

MemGPT is an OS-inspired memory-management system for [[llm-agent|LLM agents]] proposed by Packer et al. (2023, UC Berkeley). It treats the fixed context window as scarce *physical memory* and pages information in and out of it via LLM-emitted function calls, giving a finite-window model the illusion of unbounded context. It is the direct conceptual ancestor of [[letta]] and of essentially every "memory-managed agent" pattern that proliferated in 2024–2026.

## Core mechanism

- **Virtual context**: the prompt window is modeled as RAM; external stores are disk. The LLM orchestrates paging itself.
- **Three storage tiers**: **main context** (read-only system instructions + small read/write working context + FIFO queue with a recursive summary at its head); **recall storage** (warm DB of full chat history); **archival storage** (cold read/write text DB).
- **Function-call paging**: movement between tiers is driven by explicit tool calls — `archival_memory_insert/search`, `recall_memory_search`, `working_context.append/replace`. Memory ops are observable and auditable rather than implicit.
- **Queue manager + memory-pressure policy**: at a warning token count the system injects a `memory pressure` alert so the LLM proactively persists state; at the flush threshold it evicts queue items to recall storage and regenerates the recursive summary.
- **Function chaining via `request_heartbeat`**: lets the LLM sequence multi-step memory ops (paginate, search, write) within one user turn; event-driven control flow enables unprompted action.

## What it advances

Primarily **store** and **evict** in the [[memory-management]] vocabulary: MemGPT's central contribution is an explicit *tier hierarchy* with LLM-controlled movement between tiers, plus a pressure-driven eviction policy from main context to recall storage. It also operationalizes **compact** (recursive summary at the FIFO head) and **retrieve** (pageable search over warm/cold tiers) — but the novel claim is that *the LLM itself* should issue the write/page/evict operations.

## Relation to other systems

- **vs. [[memorybank]]** (concurrent, 2023): MemoryBank is single-tier with Ebbinghaus-curve forgetting; MemGPT is multi-tier with LLM-driven function-call paging. Orthogonal contributions — MemoryBank attacks *evict*, MemGPT attacks *store*.
- **vs. [[a-mem]]** (2025): A-MEM is append-non-only — neighbor notes are *rewritten* on insertion (memory evolution). MemGPT remains append-only within each tier.
- **vs. [[mem0]]** (2025): Mem0 strips the OS metaphor down to a production extract/consolidate/retrieve pipeline; MemGPT exposes the tier machinery to the model as a first-class API surface.

## References

- [[memgpt-packer-2023]]
