---
title: Context Compaction
type: concept
created: 2026-10-29
updated: 2026-10-29
sources:
  - context-language-models-shao-2026.md
  - autocompact-learning-when-to-compact-zhang-2026.md
  - cache-engineering-for-agents-fu-2026.md
  - compaction-cliff-long-running-agent-memory-zerhoudi-2026.md
  - lost-in-compaction-side-constraint-loss-wang-2026.md
  - acm-agentic-context-management-li-2026.md
  - learning-agent-compatible-context-management-yi-2026.md
  - arex-recursive-self-improving-research-baai-2026.md
status: draft
importance: high
tags:
  - 2026
  - agent
---

# Context Compaction

**Context compaction** is the step in a long-running agent where accumulated history is replaced by something shorter, usually an LLM-written summary, so the agent can keep working inside its window. Coding harnesses such as [[claude-code|Claude Code]] and Codex compact when the context nears its limit ([[autocompact-learning-when-to-compact-zhang-2026]]). It is the most common concrete operation in [[context-engineering]], and in 2026 it became a research topic in its own right because each design choice (when, what to keep, who decides) has measurable costs.

## The design questions

| Question | Default practice | Alternatives in the sources |
|---|---|---|
| **When** | Length threshold (e.g. ~90% of window) | Learned trigger: AutoCompact, [[acm-agentic-context-management-li-2026\|ACM]]; AREX's agent-invoked `update_context` fires at a mean ~25.7K of a 128K window |
| **What survives** | Task continuity: objective, state, next steps | Type-aware retention (Knowledge Triage / TypeCompact); a learned improvement-state schema (AREX) |
| **Lossy or lossless** | Raw messages discarded | ACM offloads raw messages to disk and lets the agent query them |
| **Who manages** | The harness | The agent itself (AutoCompact, ACM, CLM); a separately trained manager over a frozen agent ([[learning-agent-compatible-context-management-yi-2026\|AdaCoM]]) |
| **Unit of edit** | Text summary | Arbitrary text edits (CLM); KV-cache `drop`/`repos` operations (cache engineering) |

## What compaction loses

- **Rules and constraints.** [[compaction-cliff-long-running-agent-memory-zerhoudi-2026|The Compaction Cliff]]: Claude Code's `/compact` on Sonnet 4.6 keeps 53% of safety rules after one round and 10% after five, even when told to keep them verbatim. Type-aware TypeCompact keeps 1.00 / 0.95 / 0.80 at 50/25/10% budgets.
- **Side constraints.** [[lost-in-compaction-side-constraint-loss-wang-2026|Lost in Compaction]]: user instructions about *how* to work ("show me the draft before sending") are dropped by task-centric compactors; no compactor exceeds 36% average retention on any constraint type, and compaction usually leaves compliance near the no-constraint baseline.
- **Task progress.** Length-triggered compaction can cut through an unresolved stage. In AutoCompact's setup fixed compaction scored 1.6 and 0.9 points *below* the uncompacted base model.

## Learned compaction

- **AutoCompact** trains when/what/how to compact with judge-corrected on-policy data plus RL: +9.2 SWE-bench Verified and +5.0 SWE-PolyBench over the base. Proactive compaction helped even at a 256K window that no trajectory filled.
- **ACM** gives the agent `manage_context` and `query_memory` tools; untrained frontier models barely call them, and post-training lifts BrowseComp-Plus 0.570 → 0.727 for a 9B student.
- **AdaCoM** finds a fidelity–reliability trade-off: stronger agents want more raw context, weaker ones more aggressive compression.
- **[[context-language-models-shao-2026|Context Language Models]]** remove the fixed action menu: the model edits its own context file. 59.4% on BrowseComp-Plus at a 32K budget, 11.4% *relative* over Codex-style summarization.

## The cache cost

Any edit to the prefix invalidates the [[kv-cache]] after it, so the surviving suffix must be re-prefilled, which is why harnesses wait for a threshold and then cut hard ([[cache-engineering-for-agents-fu-2026]]). Two 2026 answers go in opposite directions: CLM edits text freely and recovers reuse with Suffix Cache Reuse in SGLang (65.0% of prefix-reuse FLOPs at matched accuracy), while [[cache-engineering-for-agents-fu-2026|cache engineering]] leaves text alone and drops or repositions KV entries directly (cost −27.7% vs standard summarization). Both reuse states computed under a context that no longer exists, so equivalence is shown empirically, not guaranteed. See [[prefix-caching]].

## Related

- [[context-engineering]] · [[agentic-context-engineering]] · [[memory-management]] · [[context-rot]] · [[long-context-llm]] · [[kv-cache-eviction]] · [[subagents]]
