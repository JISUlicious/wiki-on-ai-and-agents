---
title: "Fundamental Components of an LLM Agent Harness"
type: query
created: 2026-05-27
updated: 2026-05-27
sources:
  - code-as-agent-harness-ning-2026.md
  - agentic-harness-engineering-lin-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Fundamental Components of an LLM Agent Harness

Drawing on the two canonical 2026 frameworks ([[code-as-agent-harness-ning-2026]] survey + [[agentic-harness-engineering-lin-2026]]) plus the [[agent-three-layer-model]] L2 decomposition. The components cluster three ways — by **operational artifact** (what files / services exist), by **mechanism category** (what they do), and by **cross-cutting concern** (what they touch).

## The 6 operational components (Lin 2026's parts list)

[[agentic-harness-engineering-lin-2026]] frames the harness as **six editable artifacts** an Evolution Agent can mutate. The most concrete and shippable view:

| Component | What it is | Wiki examples |
|---|---|---|
| **System prompt** | The model's always-prepended preamble — persona, refusal style, top-level instructions | [[claude-code]] embedded preamble; [[opencode]] template-based prompts |
| **Tools** | Function definitions the model can invoke + their implementations | bash, file editor, web fetch, MCP servers; [[function-calling]], [[mcp]] |
| **Middleware** | Pre/post processors on tool calls and model calls — logging, retries, redaction, rate limits | [[claude-code]] permission engine; [[opencode]] `chat.params` mutator |
| **Skills** | Composable capability modules with their own context + tool declarations + allowed-paths | [[agent-skills]] / [[skill-md-format]] (Anthropic 2025) |
| **Sub-agents** | Forkable specialized agents the main loop can spawn | [[claude-code]] sub-agent system; [[hermes-agent]] orchestrator; [[chatdev]] roles |
| **Long-term memory** | Durable state across sessions: skills library, conversation log, learned facts | [[memory-management]] umbrella; concrete: [[memgpt]], [[mirix]], [[mem0]], [[chronos]] |

These are not abstract abstractions — they are **files and services on disk** the harness developer (or Evolution Agent) edits.

## The 5 mechanism categories (Ning 2026 survey §3)

The [[code-as-agent-harness-ning-2026|Ning survey]] reorganizes the same surface area by **what the mechanism does**, not what file holds it:

| Category | What it controls | Sub-types |
|---|---|---|
| **Planning** (§3.1) | How the agent decomposes work and commits to a sequence | Linear ([[react]], Plan-And-Act, PLAN.md), structure-grounded (over repo/graph), search-based (tree-of-code, MCTS), orchestration-based ([[metagpt]], [[claude-code]] long-running harness) |
| **Memory & Context Engineering** (§3.2) | What's in the prompt this turn; what persists across turns | Working / semantic / experiential / long-term / multi-agent / compaction. See [[memory-management]] |
| **Tool Use** (§3.3) | The agent's action and observation layer | Function-oriented (typed APIs), environment-interaction ([[agent-computer-interface]]), verification-driven (tests as tools, [[agentcoder]]), workflow-orchestration ([[mcp]], [[opencode]]) |
| **Plan-Execute-Verify (PEV) loop** (§3.4) | The cybernetic control loop | Plans-as-contracts → sandboxed execution → deterministic-sensor verification → human-in-the-loop for risky tiers. Cites [[reflexion]], Self-Debugging, LDB, QualityFlow |
| **Adaptive Optimization (AHE)** (§3.5) | How the harness improves itself | Deep telemetry + Evolution Agent + governed mutation. The frontier discipline ([[agentic-harness-engineering]], [[adas]], Meta-Harness, GEPA, Live-SWE-Agent) |

These are **operational behaviors**, not files. Mapping is many-to-many: tools live in §3.3 but planning artifacts (PLAN.md) also use the tool layer; memory feeds context-assembly which feeds the LLM call which feeds the PEV loop.

## The 6 + 6 cross-cutting concerns (3-layer model L2 view)

The [[agent-three-layer-model]] cuts the harness perpendicular to Ning's mechanism cut, by **architectural problem the system must solve**:

| L2 abstraction | What every agent must solve here |
|---|---|
| **Control flow** | Multi-step loop with a halt condition (percept → decide → act) |
| **Action surface** | Typed interaction with the outside world (file I/O, shell, web, code exec) |
| **Context assembly** | What goes into the prompt this turn — auto-loaded files, retrieved snippets, dynamic templates |
| **Memory management** | write / store / retrieve / compact / consolidate / evict / curate lifecycle |
| **Persistence** | State survives the process (JSONL session, SQLite event log, snapshot git) |
| **Harness** (narrow sense) | Bidirectional shaping: **constrain** away from harmful behavior AND **drive** toward user goals |

The narrow "Harness" L2 lives next to the others. Its concrete L3 components exist *only* to shape behavior:

- **Sandbox** — Docker / seatbelt / OS adapter; resource limits
- **Permission engine** — rule-based allowlist; per-tool / per-session policies
- **Refusal classifier** — post-output filter (model-side)
- **Hooks** — inline interceptors at lifecycle points; the cross-cutter par excellence (see [[hooks-in-modern-agents]])
- **Human-approval prompt** — confirmation dialogs for risky actions
- **Plan-mode** — explicit user-shaped plan before execution
- **Constitutional principles** — written rules in the system prompt ([[constitutional-ai]])
- **Eval harness** — pre-deployment safety + capability testing

## Unified inventory — *the* fundamental components

Synthesizing the three views, **eight components are universal across all the production harnesses documented in this wiki** ([[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]]):

1. **Loop / control flow** — the [[react]]-style percept→decide→act, with a halt condition. Always there.
2. **Tool registry + invocation layer** — typed schemas (often via [[mcp]] or [[function-calling]]); the tool-call parser; execution dispatch.
3. **Sandbox / execution environment** — the bounded surface on which tool actions run. Tiered by permission level.
4. **System prompt + context-assembly pipeline** — what's prepended every turn; auto-loaded `CLAUDE.md` / `AGENTS.md` / [[agent-skills]]; retrieved snippets; dynamic slot-fills.
5. **Memory subsystem** — short-term (in-context summary, compaction) + long-term (sessions, skills, knowledge). See [[memory-management]].
6. **Permission / safety layer** — rule engine + refusal classifier + human-approval prompts + [[constitutional-ai]] principles + plan-mode gates.
7. **Hooks** — inline interceptors at every lifecycle point. Universal in 4 of 5 documented 2026 agents; [[pi-mono]] is the minimalist exception with 2 hooks. See [[hooks-in-modern-agents]] for the full ~30-hook taxonomy.
8. **Persistence** — JSONL session trees, SQLite event logs, snapshot git. Required for replay, sharing, sub-agent forking.

Two additional components are **near-universal**:

9. **Sub-agent orchestration** — forking specialized agents from the main loop. Present in [[claude-code]], [[hermes-agent]]; refused in core by [[pi-mono]] (extension required).
10. **Telemetry / observability** — structured traces, replay support, evaluation hooks. Becoming standard in 2026 ([[agentic-harness-engineering-lin-2026]] makes this the foundation of harness evolution).

## Cross-cutters — components that touch multiple categories

Three concrete artifacts genuinely span multiple categories (and trip up clean taxonomies):

| Artifact | Categories touched | Why it's cross-cutting |
|---|---|---|
| **Skills (`SKILL.md`)** | Context-assembly + Action-surface + Harness | Inject context (when relevant), declare new tools, constrain allowed paths |
| **Hooks** | Harness + Control-flow + Context-assembly | Gate transitions, rewrite prompts, abort the loop |
| **MCP** | Action-surface + Harness + Context-assembly | Standardize tools (action), authenticate per server (harness), expose resources (context) |

Linked pages: [[agent-skills]], [[hooks-in-modern-agents]], [[mcp]].

Convention from [[agent-three-layer-model]]: assign each cross-cutter to its **primary** category, annotate the cross-cuts. Don't pretend they're clean.

## How the 5 documented 2026 agents instantiate these

| Component | claude-code | opencode | pi-mono | hermes-agent | openclaw |
|---|---|---|---|---|---|
| System prompt | embedded | template | embedded | dynamic | template |
| Tool set | 15+ built-in | plugin-based | **7 only** | ~12 + sub-agents | extensible |
| Sandbox | OS adapter | none in core | none | 7 backends | Docker tiers |
| Memory | session + skills | event log | JSONL only | SQLite + curator | per-channel |
| Hooks | 21 events | 18 events | **2 events** | 6 lifecycle | 24 events |
| Sub-agents | first-class | optional | refused | orchestrator-style | optional |
| Telemetry | structured logs | event log | minimal | curator-driven | per-channel |
| Persistence | JSONL tree | SQLite | JSONL | SQLite | per-channel |

Page links: [[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]].

[[pi-mono]] is the **minimalist control**: it explicitly refuses 4 of the 10 components (sandbox, sub-agents, rich hooks, MCP-in-core). [[claude-code]] and [[openclaw]] are at the maximalist end. The 2026 Ning survey argues the maximalist end is where autonomy gains live.

## The conceptual takeaway

Two equally-valid mental models of *what a harness is*:

1. **A set of operational artifacts** (Lin 2026): six editable files / services the harness developer or Evolution Agent edits.
2. **A set of mechanism categories** (Ning 2026): five interacting behaviors the harness enacts.

Both are correct. The first is shippable; the second is teachable. The L3 components listed above (sandbox, hooks, refusal classifier, etc.) are concrete instantiations of *both* — they live in operational files *and* enact mechanism behaviors.

The harness has bottlenecked autonomy more than the model has for the last two years. Ning's survey is the first to argue this point with 500+ citations of evidence; Lin's paper is the first to operationalize it — treat the harness *as* the object of optimization, with telemetry-driven mutation.

## Related pages

- [[agent-three-layer-model]] — the L2 cut (control flow, action, context, memory, persistence, harness)
- [[code-as-harness]] — the umbrella concept; code as the medium that unifies harness components
- [[code-as-agent-harness-ning-2026]] — the 2026 survey (§3 = mechanisms; §2 = interface)
- [[agentic-harness-engineering-lin-2026]] — the operational 6-component view + harness-as-optimization-target
- [[hooks-in-modern-agents]] — the ~30-hook taxonomy across 5 production agents
- [[memory-management]] — the lifecycle for the memory subsystem
- [[common-features-of-modern-agents]] — the 8 universal features (overlaps but is broader than just harness)

## References

- [[code-as-agent-harness-ning-2026]] — Ning et al. 2026, the canonical survey
- [[agentic-harness-engineering-lin-2026]] — Lin et al. 2026, the 6-artifact operational view
- [[agent-three-layer-model]] — the L1/L2/L3 framework this wiki uses to organize agent architecture
