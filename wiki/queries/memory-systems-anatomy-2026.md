---
title: "Memory Systems Anatomy (2026)"
type: query
created: 2026-05-28
updated: 2026-05-28
sources:
  - memgpt-packer-2023.md
  - memorybank-zhong-2023.md
  - generative-agents-park-2023.md
  - em-llm-fountas-2024.md
  - titans-behrouz-2025.md
  - a-mem-xu-2025.md
  - hipporag2-jimenez-gutierrez-2025.md
  - sleep-time-compute-lin-2025.md
  - mem0-chhikara-2025.md
  - mirix-wang-2025.md
  - memory-r1-yan-2025.md
  - mem-alpha-wang-2025.md
  - chronos-sen-2026.md
  - code-as-agent-harness-ning-2026.md
  - agentic-harness-engineering-lin-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Memory Systems Anatomy (2026)

A three-part synthesis answering an evolving conversation about how modern AI agent memory systems are built. The wiki has ingested 11+ memory-specific papers (2023–2026) plus two 2026 surveys that synthesize the field; this page brings them together.

The three parts answer:
1. **Composition** — what every modern memory system is built from
2. **Scope** — how user-specific and domain-specific long-term memory are managed
3. **Beyond scope** — the other axes the wiki distinguishes (cognitive role, function, substrate, generation, temporal binding)

---

# Part 1 — What modern memory systems are composed of

## The reframing first

"Memory" in 2024–2026 agent systems is **not "context window + vector DB."** It is a **state-management layer** with its own lifecycle. The [[code-as-agent-harness-ning-2026|Ning 2026 survey]] phrases it: memory is "not a larger context window or vector database but a state-management layer."

That move distinguishes modern memory from 2020-era RAG. RAG is one operation (retrieve) inside a wider lifecycle of write / store / compact / consolidate / evict / curate.

## The 6 functional layers (Ning 2026 taxonomy)

Every modern memory system implements some subset of these six functional roles. The biggest 2025–2026 systems implement all six or are explicit about which they skip:

| Layer | What it stores | Time horizon | Canonical examples |
|---|---|---|---|
| **Working memory** | Current task state, active hypotheses, recent tool outputs | This turn → this session | [[memgpt]] "main context"; SWE-agent's truncated last-5-observations buffer |
| **Semantic memory** | Concepts, entities, repo-level facts, world knowledge | Sessions → forever | RepoCoder, CodeRAG; [[mem-alpha]] semantic component |
| **Experiential / episodic** | Specific past events, "what happened when" | Days → forever | [[a-mem]], MemGovern, ExpeL; [[generative-agents]] memory stream; [[em-llm]] |
| **Long-term** | Durable cross-session facts and skills | Forever | [[memgpt]], MemoryOS, MemCoder; [[mem0]] |
| **Multi-agent** | Shared substrate across agents | Forever | [[mirix]] (8 agents over 6 stores); ChatDev shared repo; G-Memory |
| **Context compaction / state offloading** | Cross-cutter: shrinking or externalizing the others | Continuous | LongCodeZip, SWE-Pruner; [[sleep-time-compute]]; [[memorybank]] forgetting |

## The 7-operation lifecycle

Per [[memory-management]], every system implements these 7 operations — explicitly or implicitly:

| Operation | Question it answers | Mechanism examples |
|---|---|---|
| **Write** | What's worth recording? | [[mem0]] selective surfacing; [[generative-agents]] "importance" scoring; [[memory-r1]] learned ADD |
| **Store** | Where does it live? | [[memgpt]] main/recall/archival; [[mirix]] six typed stores; [[chronos]] dual calendars |
| **Retrieve** | When and how recalled? | Embedding similarity; [[bm25]]; graph PPR ([[hipporag-2]]); KV-cache events ([[em-llm]]); agent-driven grep/search ([[chronos]]) |
| **Compact** | How to make it smaller? | Summarization, queue eviction, LongCodeZip |
| **Consolidate** | When does ephemeral become durable? | [[reflection-mechanism]]; [[sleep-time-compute]]; [[memory-evolution]] in [[a-mem]] |
| **Evict / Archive** | What to remove? | [[ebbinghaus-forgetting]] decay ([[memorybank]]); surprise-gated forgetting ([[titans]]); learned DELETE ([[memory-r1]]) |
| **Curate** | How to keep stored memory consistent? | Neighbor updates ([[a-mem]]); conflict-resolution + dedup ([[mem0]]); UPDATE consolidation ([[memory-r1]]) |

## Compositional building blocks

A modern memory system is **composed** of these primitives:

### A. Storage substrate
- **Flat fact store** — one bag of text/embeddings
- **Tiered store** — main + recall + archival ([[memgpt]])
- **Typed compositional store** — multiple structurally distinct stores per type ([[mirix]] = 6; [[mem-alpha]] = 3)
- **Temporal-event store** — dual calendars indexed by time ([[chronos]])
- **Graph store** — knowledge graph with typed relations + PPR ([[hipporag-2]])
- **Working-memory buffer** — bounded queue in active context

### B. Content representation
- Raw turn / chat-history strings
- Atomic factual statements ([[mem-alpha]] semantic atoms)
- Subject-verb-object event tuples with time ranges ([[chronos]])
- Reflections / summaries ([[generative-agents]], [[memorybank]])
- Skill / procedure with steps ([[voyager]] skill library, [[mirix]] procedural)
- Multimodal distillations ([[mirix]] ScreenshotVQA)

### C. Indexing & retrieval
- Dense vector (cosine over embeddings)
- Sparse / lexical ([[bm25]], grep — [[grep-all-you-need-sen-2026]] argues this is undersold)
- Graph traversal (PPR — [[hipporag-2]])
- Time-indexed scan ([[chronos]])
- Hybrid (dense → rerank → adjacency expansion)
- KV-cache events ([[em-llm]] with Bayesian-surprise segmentation)
- Agent-driven retrieval (multiple tools the agent picks)

### D. Update / write policy
- Append-only
- Heuristic LLM-controlled ([[mem0]] extract-update, [[mirix]] Meta Manager)
- **RL-learned operations** ([[memory-r1]], [[mem-alpha]] — 2025 frontier)
- Delta-update with helpful/harmful counters ([[agentic-context-engineering]])
- Decay-based ([[memorybank]] Ebbinghaus)
- Surprise-gated ([[titans]])

### E. Curation / forgetting
- TTL / capacity eviction
- Decay curve ([[memorybank]])
- Surprise-driven ([[titans]])
- Learned DELETE ([[memory-r1]])
- Conflict resolution + dedup ([[mem0]], [[memory-r1]])
- Sleep-time consolidation ([[sleep-time-compute]])

### F. Coordination architecture (multi-agent)
- Shared store
- Per-agent stores + Meta Manager ([[mirix]])
- Hierarchical / blackboard (CoALA, G-Memory)

## What each 2024–2026 system is composed of

| System | Storage | Representation | Retrieval | Update | Eviction |
|---|---|---|---|---|---|
| [[memgpt]] (2023) | Tiered (main/recall/archival) | Raw + summaries | Embedding | Heuristic LLM page-in/out | Capacity-driven |
| [[memorybank]] (2023) | Flat fact store | Raw + summaries | Embedding | Append + summarize | Ebbinghaus decay |
| [[em-llm]] (2024) | KV-cache event segments | KV activations | Surprise-segmented + similarity | Implicit | KV eviction |
| [[titans]] (2025) | Neural long-term memory module | Learned state | Attention-based gating | Online training | Surprise-gated forget |
| [[a-mem]] (2025) | Self-organizing notes graph | Notes + links | Hybrid | LLM-controlled + neighbor updates | Self-organized |
| [[hipporag-2]] (2025) | Knowledge graph | S-R-O triples | Personalized PageRank | Graph append | Append-only |
| [[sleep-time-compute]] (2025) | Working + consolidated (any) | Any | Any | Offline LLM reprocessing | Consolidation merges |
| [[mem0]] (2025) | Flat + optional graph | Atomic facts | Embedding | Heuristic LLM extract-update | Conflict-driven dedup |
| [[mirix]] (2025) | **Six typed stores** | Per-type schemas | Active retrieval (top-10 per store) | 8-agent coordination | Per-type policies |
| [[memory-r1]] (2025) | Flat store | Atomic facts | Embedding + distillation | **RL-learned** ADD/UPDATE/DELETE/NOOP | Learned DELETE |
| [[mem-alpha]] (2025) | **Three components** (core/semantic/episodic) | Atomic + summary + events | BM25 | **RL-learned construction** (GRPO) | Tool-call-driven |
| [[chronos]] (2026) | **Dual calendars** (events + turns) | `<subj, verb, obj>` + ISO 8601 | Top-100 vector → rerank → ±1 neighbor | LLM-extracted events | Append + dedup |

## The 2025–2026 frontier (three trends)

1. **Typed compositional stores** — [[mirix]] (6 types), [[mem-alpha]] (3 components), [[chronos]] (events + turns) all reject the flat-store assumption. Each memory type gets its own schema, write policy, and retrieval mechanism. This is the cognitive-architecture lineage (CoALA, Sumers 2023) finally getting empirical instantiations.

2. **RL-learned memory operations** — [[memory-r1]] and [[mem-alpha]] train the *write policy* with [[grpo]] using downstream task correctness as reward. The agent learns when to ADD vs UPDATE vs DELETE — not by rule, not by prompt, but from outcomes. Memory-R1: +28% F1 over Mem0 on LoCoMo with only 152 training pairs. Mem-α: 30k → 474k tokens (13× generalization) by learning *management principles* rather than surface patterns.

3. **Temporal-aware structured event memory** — [[chronos]] makes time a first-class index, not metadata. Events as `<subj, verb, obj>` with ISO 8601 ranges + lexical aliases. Retrieval pipeline (top-100 vector → Cohere rerank → ±1 neighbor) hits **95.6% on LongMemEval-S** — current SOTA.

## A composite 2026-canonical recipe

```
   Storage      = typed compositional (6 stores ~ MIRIX)
                  OR dual time-indexed (events + turns ~ Chronos)
   Representation = structured tuples + time ranges + lexical aliases
   Retrieval    = hybrid: vector → rerank → adjacency expansion
                  + agent-driven grep/search for misses
   Write        = RL-learned policy over {ADD, UPDATE, DELETE, NOOP}
   Consolidate  = sleep-time-compute offline pass
   Forget       = surprise-gated OR learned DELETE
   Coordination = per-store typed managers + Meta agent (MIRIX pattern)
```

No published system combines all of these. The synthesis hasn't shipped — each paper picks a slice.

---

# Part 2 — How long-term memory, user-specific and domain-specific, is managed

Long-term memory faces three problems short-term doesn't: **persistence across sessions**, **scope separation** (which user / which project?), and **eventual consistency** (the same fact edited or contradicted weeks later).

## The two scoping axes

Modern systems separate long-term memory along **two coordinates**:

| Axis | "User-specific" | "Domain-specific" |
|---|---|---|
| **What it remembers** | Identity, preferences, history, relationships | Schemas, code conventions, terminology, domain rules |
| **Lifetime** | Forever (across sessions) | Forever or until the domain changes |
| **Index** | `user_id` | `project_id` / repo |
| **Curation cost** | High — humans care if preferences are remembered wrong | Lower — facts about a repo update when the code does |
| **Privacy weight** | High — PII, credentials | Variable — public code vs. proprietary IP |

Production systems usually combine these: each memory record is `(user × project)`-scoped, not flat-namespaced.

## Storage patterns

### Core memory blocks (user-specific persona)

From [[memgpt]] (2023), inherited by [[mirix]] as "Core Memory" and [[mem-alpha]] as "core":

- A **bounded text block** (≤ 512 tokens in Mem-α; up to a few KB in MemGPT)
- **Always in active context** — never demoted to recall
- Holds name, preferred communication style, persistent identity, long-running goals
- Updated via **holistic rewrite** (not patch; whole block rewritten)
- Forever, but contents evolve

Production instances:
- [[claude-code]] `CLAUDE.md` (project) + `USER.md` (user) auto-loaded
- [[letta]]: persistent persona block + recall as user-facing primitives
- [[hermes-agent]]: dynamic core memory edited by a curator sub-agent

### Semantic / fact memory (atomic, queryable)

What [[mem-alpha]] calls "semantic", [[mirix]] "Semantic Memory", [[mem0]] uses as primary store:

- **Atomic factual statements**: "User adopted Buddy on 2024-03-12", "Project uses pnpm"
- Individual records, retrievable by embedding similarity or [[bm25]]
- Updated via extract-update pipeline ([[mem0]]) or RL-learned ops ([[memory-r1]])
- Both user- and domain-specific go here, distinguished by tags

The most contested layer. Heuristic LLM-based (mem0, MIRIX) is the production default; RL-trained policies ([[memory-r1]] +28% F1) is the 2025 frontier.

### Procedural memory (how-to workflows)

Often skipped in 2023, explicitly broken out by [[mirix]] in 2025:

- Step-by-step procedures with ordered actions
- Examples: "How to deploy this repo" (domain), "How user prefers reviews structured" (user × domain)
- Updated when the procedure proves successful (reinforcement) or fails (revision)
- Closest to [[voyager]]'s skill library

Domain-specific procedural memory is the closest thing modern agents have to **institutional knowledge**.

### Resource memory (full documents)

[[mirix]] introduces this in 2025 as the bridge between memory and file storage. Documents the agent has read or generated and might revisit. Indexed by topic, summary, or metadata. Often delegated to filesystem + metadata rather than embedded.

### Knowledge Vault (sensitive)

[[mirix]]'s sixth type — items needing **sensitivity labels**: API keys, credentials, PII, health info. Stored separately so retrieval can apply different access policies. Important because user-specific memory accumulates PII over time; treating it as a distinct type with stricter handling is a 2025 architectural innovation.

### Episodic / event memory (temporal)

[[chronos]] (2026) is the canonical implementation; [[a-mem]] and [[generative-agents]] earlier variants:

- Subject-verb-object event tuples with ISO 8601 time ranges
- Both user actions and domain events
- Retrieved by time-window queries OR by topic + temporal expansion
- Bridges "what did the user say last Tuesday" with "when did the schema migration happen"

## How updates are managed

Three patterns in production:

**A. Heuristic LLM-driven** ([[mem0]], [[mirix]], [[a-mem]]): an LLM reads conversation + current memory, emits `ADD/UPDATE/DELETE/NOOP` operations. Dedup enforced, conflicts trigger UPDATE. [[mirix]]'s twist: 8 specialized agents (Meta + 6 typed + Chat) route fact types in parallel.

**B. RL-learned operations** ([[memory-r1]], [[mem-alpha]]): a Memory Manager agent trained with [[grpo]], reward = downstream QA correctness. The motivating example for Memory-R1: "I adopted Buddy" + later "I adopted Scout" — vanilla managers DELETE + ADD (losing context); the RL-trained manager learns a single UPDATE consolidating both.

**C. Delta-update on curated artifact** ([[agentic-context-engineering]]): Generator/Reflector/Curator triad with bullet-level delta updates + helpful/harmful counters. Same pattern can apply to memory: each fact is a bullet with usage-counter; validated bullets get promoted, contradicted ones demoted.

## How retrieval is scoped

Retrieval must respect the scope:

1. **Tag-filtered embedding search** — every record has `user_id` + `project_id`; filter first, rank by similarity
2. **Per-scope indices** — [[mirix]] gives each user × domain a dedicated set of 6 typed stores; retrieval is intra-scope by construction
3. **Active retrieval with scope-aware prompting** — [[chronos]]'s pattern: meta-prompt generates per-question search hints that include scope cues; agent picks among 4 tools (`search_turns`, `search_events`, `grep_turns`, `grep_events`)

## How forgetting is managed

User-specific memory: don't lose what the user cares about, don't surface stale data.

| Mechanism | Where used |
|---|---|
| TTL hard expiry | Caches, working memory only |
| Ebbinghaus decay | [[memorybank]] |
| Surprise-gated forgetting | [[titans]] |
| Learned DELETE | [[memory-r1]] |
| Conflict resolution + dedup | [[mem0]], [[mirix]] |
| Sleep-time consolidation | [[sleep-time-compute]] |
| Explicit user gates ("forget that I said X") | [[letta]] product, [[zep]] |

For user-specific memory, **explicit forgetting controls** are increasingly important (GDPR, right to be forgotten, user trust). Letta and Zep expose user-facing "forget" APIs.

For domain-specific memory, **automatic conflict resolution** dominates — when code changes, the stale fact should be revised.

## Production patterns across the 5 documented agents

| Agent | User memory | Domain memory | Mechanism |
|---|---|---|---|
| [[claude-code]] | `~/.claude/USER.md` + session JSONL | Project `CLAUDE.md` with nested overrides | File-based, deterministic |
| [[opencode]] | event-log per user | event-log per session | SQLite + event sourcing |
| [[pi-mono]] | JSONL session tree only | Manual via [[agent-skills]] | Minimal — refuses durable cross-session memory in core |
| [[hermes-agent]] | SQLite + curator sub-agent | SQLite + curator | Self-editing curator updates core |
| [[openclaw]] | per-channel store | per-channel store | Per-channel namespace |

[[claude-code]]'s `CLAUDE.md` hierarchy is the most concrete answer: **a hierarchy of files at session / project / user scopes, auto-loaded into context on session start, with nested overrides**. No vector DB. No embedding similarity. Pure file structure.

The hidden insight: **a markdown file you can grep is itself a memory system**. The agent reads it every session; the user edits it; conflicts resolve via git. The whole apparatus of "managed memory" is replaced by editable text.

## Production infrastructure offerings

- **[[letta]]** — productized MemGPT. Persistent persona + recall + archival as a managed runtime.
- **[[zep]]** — long-term memory infrastructure (Zep Cloud + Graphiti). Indexes user-scoped conversation history.
- **Anthropic memory tooling** — cross-session conversation memory keyed to user accounts.
- **OpenAI memory** — per-user opt-in memory in ChatGPT; selectively writes user-surfaced facts.

---

# Part 3 — Kinds of memory beyond user and domain

User and domain are **scoping** axes — they answer *who/what is this memory about*. There are several other axes the wiki distinguishes, each carving the space differently. The richest 2025–2026 systems combine 5–6 of these.

## 1. By cognitive role (CS-inspired taxonomy)

The CoALA / [[mirix]] / [[mem-alpha]] line of work maps human-memory psychology onto agent memory:

| Type | What it stores | Example |
|---|---|---|
| **Episodic** | Specific time-stamped events | "Deployed schema-v2 on 2026-03-12 at 14:32" — [[chronos]], [[a-mem]], [[generative-agents]] |
| **Semantic** | General facts decoupled from when learned | "Postgres uses MVCC" — [[hipporag-2]], [[mem0]] |
| **Procedural** | How-to / workflows / skills | "To deploy: build → push → migrate → flip" — [[voyager]], [[mirix]] procedural |
| **Working** | Current task state, active hypotheses | In-context summary, SWE-agent's last-5 buffer |
| **Sensory / perceptual** | Recent raw input streams | [[mirix]] screenshot pipeline (1.5s capture interval); camera frames in embodied agents |
| **Affective** *(emerging)* | Emotional dispositions | [[emotion-concepts-anthropic-2026]] emotion concepts could in principle map to preferences |

**Procedural** is the most distinct from user/domain — a procedure may be useful across users and across domains ("how to write a clean PR").

## 2. By function in the agent loop

This cut is more pragmatic — memories play **different roles** regardless of content:

| Function | Role | Where it lives |
|---|---|---|
| **Identity / core** | Persistent persona, top-level goals | [[memgpt]] persona block; [[mirix]] Core; [[hermes-agent]] curator-edited core |
| **Skill / capability** | Reusable agent-acquired competence | [[voyager]] skill library; [[agent-skills]] `SKILL.md`; MetaGPT SOPs |
| **Reflection / self-critique** | Past failures, things to avoid | [[reflexion]] verbal feedback; [[generative-agents]] reflection layer |
| **Audit / governance** | What the agent did, when, with whose permission | claude-code session JSONL; opencode event log; Hermes SQLite |
| **Telemetry / observability** | Statistics on agent behavior | [[agentic-harness-engineering-lin-2026]] experience-observability corpus |
| **Multimodal / sensory** | Visual / audio / sensor data distilled | [[mirix]] ScreenshotVQA; embodied-agent perception buffers |
| **Sensitive / vault** | PII, credentials, access tokens | [[mirix]] Knowledge Vault |
| **Resource / artifact** | Documents, generated code, references | [[mirix]] Resource Memory; [[code-as-harness]] agent-initiated code artifacts |
| **Evolution / harness-history** | Past versions of the harness + fitness scores | [[agentic-harness-engineering-lin-2026]]; [[darwin-godel-machine]] population archive |
| **Shared / multi-agent** | Substrate for cross-agent coordination | [[mirix]] Meta Manager; ChatDev shared repo; G-Memory blackboard |

**Evolution / harness-history** is genuinely new in 2025–2026 — agents that improve themselves over time need memory of their own past designs and which performed better.

## 3. By substrate (physical / data-structure form)

Not *what* is remembered, but *how* it's stored:

| Substrate | Implementation | Strengths |
|---|---|---|
| Text in context | Prepended to prompt | Cheap, always available; bounded by window |
| Vector embeddings | Embed + ANN search ([[dpr]]) | Semantic similarity, scales |
| Knowledge graph | Nodes + typed edges + traversal | Relational queries, [[hipporag-2]] PPR |
| Tuple / event store | `<subj, verb, obj>` + time | [[chronos]] dual calendars |
| File system | Files in a directory tree | Editable by humans, grep-able, version-controllable; [[claude-code]] `CLAUDE.md` |
| SQLite / structured DB | Tables + queries | [[opencode]], [[hermes-agent]] event logs |
| Neural parametric | Weights inside a network | [[titans]] long-term memory module |
| KV-cache events | Spans of the model's own KV cache | [[em-llm]] (Bayesian-surprise segmentation) |
| OS-native (tiered) | main/recall/archival | [[memgpt]] |

**Neural parametric** is the most under-discussed — what a model knows from pre-training is itself memory. [[titans]] makes this explicit by training a *separate* memory module that updates during inference.

**File system** is the most under-rated — see [[claude-code]]'s `CLAUDE.md` hierarchy. A memory system disguised as documentation.

## 4. By generation / provenance

Where does the memory come from? Softer axis but matters for trust:

- **Observed** — what the agent saw happen (tool outputs, environment state, user inputs)
- **Self-generated** — what the agent decided, planned, or wrote
- **Communicated** — what the user explicitly told it
- **Synthesized** — derived via reflection / consolidation ([[reflection-mechanism]], [[sleep-time-compute]])
- **Retrieved** — pulled from external knowledge, then promoted

[[mem0]] and [[memory-r1]] distinguish provenance levels — facts derived from long conversation treated differently from facts dictated by the user.

## 5. By temporal binding

Mostly orthogonal to content type:

- **Immutable / append-only** — audit logs, version-controlled history
- **Latest-wins / overwrite** — Core memory, latest user preference
- **Versioned** — each fact has a history
- **Decaying** — strength decays unless reinforced ([[memorybank]])
- **Surprise-gated** — kept while informative ([[titans]])

## 6. The 2025–2026 emerging kinds

Didn't exist as named categories before recent work:

| Kind | What it does | Source |
|---|---|---|
| **Harness-history memory** | Records every mutation the harness made, predicted vs actual outcome | [[agentic-harness-engineering-lin-2026]] |
| **Code-artifact memory** | Tests, temporary tools, DSL programs the agent created, retained for reuse | [[code-as-harness]] |
| **Multimodal-distilled memory** | Continuous visual/audio compressed into structured text | [[mirix]] ScreenshotVQA |
| **Sub-agent fork memory** | What past sub-agent forks did + outcomes | [[claude-code]] sub-agent system |
| **Self-evolution archive** | Population of past agent scaffolds with fitness scores | [[darwin-godel-machine]] |
| **Prompt-as-evolving-artifact** | System prompt as delta-edited playbook with helpful/harmful counters | [[agentic-context-engineering]] |

## 7. Cross-cutting metadata layers

Properties memories carry, regardless of kind:

- **Provenance** — where each fact came from
- **Confidence / trust score** — how reliable
- **Recency / staleness** — last-touched timestamp
- **Usage counters** — helpful_count, harmful_count ([[agentic-context-engineering]] bullets)
- **Sensitivity label** — public / private / credentialed ([[mirix]] Knowledge Vault)
- **Source citation** — which session / tool produced this

## Putting it together

A typical 2025–2026 production agent runs **6+ distinct kinds of memory simultaneously**, only two of which are user-scoped or domain-scoped:

```
User-scoped:    Core (persona) + Episodic (history) + Preferences (semantic)
Domain-scoped:  Repo-Semantic + Procedural (workflows) + Resource (docs)
Cross-cutting:  Vault (sensitive) + Audit (governance)
System-level:   Working memory + Skill library + Reflection log
Frontier:       Harness-history + Code-artifact + Multimodal distillation
                + Self-evolution archive
```

The taxonomy isn't settled — different papers carve it differently. But the **set of distinct, named, separately-managed kinds** has grown from ~3 in 2023 (working, persona, recall) to **10+ in 2026**. The trend is unambiguous: **more types, each with its own schema, write policy, and retrieval mechanism**, rather than one bigger vector store.

---

## Conceptual takeaways across all three parts

1. **"Long-term" ≠ "vector DB"**. Modern agent memory is a *typed compositional store*, a *file hierarchy*, or a *temporal event log* — not just embeddings.

2. **Scope is structural, not just metadata**. Production systems separate user × domain via dedicated stores or indices.

3. **Update policy is the hardest part**. Heuristic LLM-driven is the baseline; RL-learned ops is the 2025 frontier.

4. **Privacy / sensitivity is becoming first-class**. MIRIX Knowledge Vault, Anthropic PII handling, Letta forget APIs.

5. **File-based wins for domain memory**. [[claude-code]] `CLAUDE.md` files outperform vector stores for project-level knowledge because the user can read + edit them directly.

6. **The taxonomy is exploding**. 2023: 3 named kinds (working, persona, recall). 2026: 10+ distinct named kinds.

## Related pages

- [[memory-management]] — the umbrella lifecycle
- [[mirix]] / [[memory-r1]] / [[mem-alpha]] / [[chronos]] — the 2025–2026 frontier systems
- [[memgpt]] / [[memorybank]] / [[generative-agents]] — the 2023 foundations
- [[em-llm]] / [[titans]] — neural / KV-cache approaches
- [[hipporag-2]] — graph-based retrieval
- [[sleep-time-compute]] — consolidation lifecycle
- [[a-mem]] / [[mem0]] — heuristic LLM-driven updates
- [[longmemeval]] — canonical evaluation benchmark
- [[agentic-search]] — in-loop retrieval companion
- [[code-as-agent-harness-ning-2026]] — survey §3.2 organizes memory into 6 functional layers
- [[agent-skills]] — file-based domain memory pattern
- [[letta]] / [[zep]] — productized memory infrastructure
- [[agentic-harness-engineering-lin-2026]] — harness-history as a new memory kind
- [[darwin-godel-machine]] — self-evolution archive memory
- [[code-as-harness]] — code-artifact memory

## References

All 15 wiki sources listed in the frontmatter, plus the umbrella concept pages above.
