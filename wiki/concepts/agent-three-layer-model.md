---
title: Agent Three-Layer Model
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - comparison.md
  - claude-code.md
  - opencode.md
  - pi-mono.md
  - hermes-agent.md
  - openclaw.md
status: complete
importance: high
tags:
  - 2026
---

# Agent Three-Layer Model

A generative framework for reasoning about modern [[llm-agent|LLM-agent]] systems. Separates **(L1) LLM capabilities** the agent assumes, **(L2) agent-architecture abstractions** the agent must solve, and **(L3) concrete implementations** of those abstractions. Originated as a critique-driven refinement of the cross-sectional comparison filed at [[common-features-of-modern-agents]] — the 2-layer "model vs. agent" cut wasn't sharp enough to distinguish *required design problems* from *specific mechanisms*.

The model maps onto a classical software pattern (capability → architecture → mechanism) and onto the [[llm-agent|classical AI-agent]] tradition (a rational agent perceives, deliberates, acts; the LLM is one possible deliberator).

## The three layers

```
┌──────────────────────────────────────────────────────────────────┐
│ L1 — LLM CAPABILITIES                                            │
│ What the substrate provides. The agent does not implement these; │
│ it assumes them and exploits them.                               │
├──────────────────────────────────────────────────────────────────┤
│ • Instruction following (SFT)                                    │
│ • Alignment (RLHF / CAI / DPO)                                   │
│ • In-context learning  ← enables every context-assembly trick    │
│ • Reasoning (CoT-capable; RL-trained inner CoT in 2024+ models)  │
│ • Emergent capabilities at scale (tool-call format adherence,    │
│   multi-step planning, self-critique, code generation)           │
└──────────────────────────────────────────────────────────────────┘
                              │
                              │   exposed via API
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ L2 — AGENT ARCHITECTURE                                          │
│ The design problems any modern agent must solve. Six abstractions│
│ — concrete enough to be auditable, abstract enough that          │
│ different implementations satisfy them.                          │
├──────────────────────────────────────────────────────────────────┤
│ • Control flow      — percept → decide → act loop with halt cond.│
│ • Action surface    — typed interaction with the outside world   │
│ • Context assembly  — every turn's prompt is constructed         │
│ • Memory management — write/store/retrieve/compact/consolidate/  │
│                        evict/curate lifecycle                    │
│ • Persistence       — state survives the process                 │
│ • Harness           — bidirectional shaping: block harmful AND   │
│                        drive toward user goals                   │
└──────────────────────────────────────────────────────────────────┘
                              │
                              │   each L2 has 1+ L3 implementation
                              ▼
┌──────────────────────────────────────────────────────────────────┐
│ L3 — IMPLEMENTATION                                              │
│ Concrete mechanisms. Each L3 item has a primary L2 parent;       │
│ cross-cutters touch multiple.                                    │
├──────────────────────────────────────────────────────────────────┤
│ Control flow:   [[react|ReAct]] (T→A→O) · planner-executor ·     │
│                 multi-agent dispatch · tree-search over actions  │
│                                                                  │
│ Action surface: typed tool API · function calling ·              │
│                 [[mcp|MCP]] · ACP                                │
│                                                                  │
│ Context         CLAUDE.md/AGENTS.md auto-load · SKILL.md         │
│ assembly:       packages † · retrieved snippets · dynamic prompt │
│                 templates · system prompt                        │
│                                                                  │
│ Memory          hierarchical files (hot/warm/cold) ·             │
│ management:     auto-consolidate ([[sleep-time-compute]]) ·      │
│                 forgetting policies ([[memorybank]]) · graph     │
│                 index ([[hipporag-2]]) · per-type stores         │
│                 ([[mirix]]) · learned ops ([[memory-r1]])        │
│                                                                  │
│ Persistence:    JSONL session tree · SQLite event log ·          │
│                 snapshot git                                     │
│                                                                  │
│ Harness:        sandbox · rule-based permission engine ·         │
│                 refusal classifier · hooks ‡ · human-approval    │
│                 prompt · plan-mode · constitutional principles · │
│                 eval harness                                     │
└──────────────────────────────────────────────────────────────────┘

† Skills also touch Action surface (declare tools) and Harness
  (allowed-paths metadata). Cross-cutter — primary parent is Context.
‡ Hooks also touch Control flow (they intercept the loop).
  Cross-cutter — primary parent is Harness.
```

## L2 abstractions in detail

### Control flow

The agent must run a multi-step loop with a halt condition. The classical AI-agent abstraction (Russell & Norvig): **percept → decide → act**, repeatedly. The LLM is one possible *decider*.

[[react|ReAct]] is the dominant L3 implementation in 2026 — `Thought → Act → Observation` is structurally the classical loop with the model emitting both Thought and Act. But planner-executor splits ([[metagpt-hong-2023|MetaGPT]]), multi-agent dispatch ([[autogen-wu-2023|AutoGen]], [[hermes-agent]] orchestrator), and tree-search over actions ([[tree-of-thoughts]], LATS) are also valid implementations of the same L2 abstraction.

[[pi-mono]]'s refusal of sub-agents is not a missing L2 — it's an explicit L3 choice (single-agent ReAct).

### Action surface

The agent needs typed interaction with the outside world: file I/O, shell, web fetch, code execution, etc. L3 mechanisms: function calling (OpenAI / Anthropic native), [[mcp|MCP]] (Model Context Protocol), ACP (Agent Client Protocol).

Choice of action-surface protocol is partly a coupling decision: MCP-first agents ([[claude-code]], [[opencode]], [[hermes-agent]]) commit to one ecosystem; MCP-refusing agents ([[pi-mono]], [[openclaw]] in core) keep the agent's tool layer self-contained.

### Context assembly

Every turn, the agent constructs a prompt. The decision of *what to include* is the agent's, not the user's. L3 inputs to context assembly:

- **Auto-loaded files** — [[claude-code]]'s nested `CLAUDE.md`, AGENTS.md walks, `USER.md`, `MEMORY.md`
- **Skills** — `SKILL.md` packages selected by relevance to the current task
- **Retrieved snippets** — lexical (grep/ripgrep/FTS5/[[bm25]]), [[dense-retrieval|dense]] embedding, hierarchical, or tool-based on-demand
- **Dynamic prompt templates** — slot-fills based on task type, user preferences, model
- **System prompt / persona** — the always-prepended preamble

This category exists because [[in-context-learning]] is an L1 capability. Without ICL, none of these mechanisms would change behavior — they'd just be noise. The whole agent layer is a wager that *putting the right tokens in the prompt* will get the right behavior out.

### Memory management

State across turns and sessions. The wiki's [[memory-management]] page defines the seven-operation lifecycle — `write` / `store` / `retrieve` / `compact` / `consolidate` / `evict` / `curate`. L3 implementations span:

- Hierarchical files (hot/warm/cold) — see [[memgpt]]
- Auto-consolidation in idle time — see [[sleep-time-compute]]
- Forgetting policies — see [[memorybank]]
- Graph-indexed retrieval — see [[hipporag-2]]
- Per-type memory stores — see [[mirix]]
- RL-learned ops — see [[memory-r1]], [[mem-alpha]]

Critically, **retrieval is one operation, not the umbrella**. Modern memory papers advance other operations more than they advance retrieval (which is why [[mem0]] and [[a-mem]] beat naive [[retrieval-augmented-generation|RAG]] baselines on long-horizon agent tasks despite using the same retrieval primitives).

### Persistence

State survives the process. Separate from memory because memory *curates* what's worth keeping; persistence *stores* whatever the curation step produces. L3: JSONL session trees ([[pi-mono]], [[claude-code]]), SQLite + event log ([[opencode]], [[hermes-agent]]), snapshot git for filesystem rollback ([[opencode]]).

### Harness

Bidirectional shaping: the agent must be **constrained** away from harmful or unintended behavior **and** **driven** toward the user's actual goal. The term "harness" is established in the AI-evals literature (METR, AISI) and captures both directions of shaping with one concept — like a horse harness, which bridles *and* steers.

This is distinct from the wiki's [[steering]] and [[guardrailing]] concepts, which sit at the *model* layer (steering = internal activation manipulation; guardrailing = training-time / inference-time external boundaries). Harness is the *agent-layer* counterpart that uses model-layer capabilities (RLHF-aligned refusals, instruction-tuned obedience) as primitives.

L3 mechanisms — note that several do double duty (block AND drive):

| Mechanism | Block | Drive |
|---|---|---|
| Sandbox / permission rules | ✓ (deny dangerous actions) | |
| Refusal classifier | ✓ (filter harmful outputs) | |
| Hooks (pre/post-tool, prompt-submit) | ✓ (gate) | ✓ (rewrite prompts) |
| Human-approval prompt | ✓ (require user OK) | ✓ (correct plan before execution) |
| Plan-mode / explicit planning | | ✓ (commit to user-shaped plan) |
| System prompt / persona | ✓ (refusal style) | ✓ (set tone, goals) |
| [[constitutional-ai\|Constitutional]] principles in prompt | ✓ (rule violation) | ✓ (positive principles) |
| Eval harness | ✓ (test for unsafe behavior) | ✓ (test for task completion) |

The bidirectional view reveals why naive "guardrail = filter" framings undershoot: most production harness mechanisms simultaneously block *and* direct.

## Cross-cutters

Some L3 mechanisms genuinely span multiple L2 abstractions. Two prominent cases:

- **Skills** primarily live under context assembly (they inject task-specific prompts into context), but they also declare tools (action surface) and constrain allowed paths (harness). Three-way cross-cutter.
- **Hooks** primarily live under harness (they enforce gates), but they intercept the loop (control flow) and can rewrite prompts (context assembly). Three-way cross-cutter.
- **MCP** primarily lives under action surface but bundles authentication (harness) and structured-prompt resource injection (context assembly).

Convention: assign each L3 item to its *primary* L2 parent; annotate cross-cutters explicitly. The reader gets a clean default with the option to drill in.

## How to use this model

### Evaluate a new agent

Walk the six L2 abstractions and ask "how does this agent solve this?" Three patterns to watch for:

1. **Unanswered abstraction** — a true gap. An agent with no persistence is a session-only toy; an agent with no harness is a research demo, not a product.
2. **Explicit refusal** — answered by *not doing it*. [[pi-mono]] refuses sub-agents; that's an L3 choice ("single-agent ReAct only"), not a missing L2.
3. **Stack of implementations** — one L2 satisfied by multiple L3s. Hermes's [[hermes-agent|four-layer memory]] is multiple L3 mechanisms under one L2.

### Predict propagation

Capability changes at L1 propagate up:

- Longer reasoning context → expect L3 control-flow shifts (more sub-agent dispatch, more planning)
- Better tool-call adherence → expect L3 action-surface shifts (richer tool APIs, less guardrail effort)
- Longer base context → expect L3 memory shifts (less aggressive eviction, more in-context state)
- Better refusal calibration → expect L3 harness shifts (less classifier-stacking, more reliance on model-native refusal)

### Spot mis-leveled features

- "RLHF is an agent feature" — no, L1.
- "ReAct is the agent architecture" — no, L3 implementation of the L2 control-flow abstraction.
- "Our agent has dense retrieval" — that's a single L3 mechanism under memory-management. Doesn't say what other L2 abstractions you've solved.
- "Steering reduces hallucinations in our agent" — at the model layer ([[activation-engineering|ActAdd]]/[[contrastive-activation-addition|CAA]]/etc.) or via the agent harness (constitutional prompts)? Different things; don't conflate.

### Distinguish enabling conditions from design choices

L1 = enabling (you can't change it without changing the model). L2 = required design problem (you can't skip it without removing capability). L3 = chosen mechanism (you could have implemented it differently). The map of which is which is the value the model adds.

## Tensions and limits

The model is auditable but prescriptive. Some honest warts:

- **Reasoning straddles L1 (capability) and L3 (elicitation).** The model can reason; the agent decides how to elicit reasoning (CoT prompts, scratchpads, sub-agent planners). The straddle is acknowledged rather than fixed.
- **Skills are unambiguously cross-cutting.** They aren't a clean instance of any one L2. The layered model already strains here; alternative decompositions would have to handle this too.
- **[[mcp|MCP]] is becoming its own protocol layer.** Currently classified under action surface but trends toward being a fourth L2 abstraction ("Interop") once universal adoption settles.
- **Sub-agent recursion isn't shown.** A sub-agent is itself an L1/L2/L3 system. The layered description doesn't render the composition — the model is single-agent-shaped, with multi-agent dispatch shown as an L3 choice. A true compositional rendering would be a graph, not a stack.
- **Infrastructure is excluded.** I/O surface, streaming, markdown rendering, terminal/web/desktop shells — all real, all universal, all *not* interesting architectural choices. Lives below L1 as enabling scaffolding.

## Origin

Refined across [a critique chain in 2026-05-16 conversations](#) — the 8-feature flat list at [[common-features-of-modern-agents]] was first split into a 2-layer (model / agent) model, then sharpened to 3 layers after identifying that "model-vs-agent" still collapses architecture and implementation. The harness naming came from recognizing that "guardrail" undershoots — the agent must *also* drive toward goals, not just block harm; "harness" (already used in evals literature) captures both directions.

## References

- [[common-features-of-modern-agents]] — the 8-feature flat predecessor
- [[cross-agent-comparison-2026]] — the cross-sectional data this model abstracts from
- [[memory-management]] — the lifecycle that fills the L2 memory-management abstraction
- [[steering]], [[guardrailing]] — model-layer paired safety concepts (distinct from agent-layer harness)
- [[from-transformer-to-modern-agents]] — the 9 evolutionary tracks orthogonal to this cross-section
- [[llm-agent]] — the classical agent definition this model maps onto
- [[react]] — the dominant L3 control-flow implementation
