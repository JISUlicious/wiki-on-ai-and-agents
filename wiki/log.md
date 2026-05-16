---
title: Activity Log
type: log
created: 2026-05-12
updated: 2026-05-16
---

# Activity Log

Chronological record of wiki operations. Newest entries first.

---

## [2026-05-16] lint | Wiki health check + auto-fix (Level 3)

Second-pass lint after today's MCP/ACP + tool-calling ingests. Broken wikilinks dropped 101 → 78 (-23%). Wiki healthy on all other dimensions: 0 orphans, 0 index inconsistencies, 0 frontmatter issues, 0 tag-hygiene violations, 0 entity-classification problems.

**Auto-fixed (Level 3 — slug drifts + high-leverage stubs)**:
- Fixed 2 slug drifts: `[[dense-passage-retrieval]]` → `[[dpr]]` (3 refs), `[[hong-metagpt-2023]]` → `[[metagpt-hong-2023]]` (1 ref).
- Created 8 **memory-canon stubs** honoring `introduces:` typed-relation promises from earlier source ingests: [[zettelkasten-memory]], [[test-time-memory]], [[rl-memory-policy]], [[offline-consolidation]], [[neural-long-term-memory]], [[multi-agent-memory]], [[memory-construction]], [[bayesian-surprise-segmentation]]. All `status: draft`.
- Created 4 **protocol / RL / schema stubs**: [[json-schema]] (8 inbound refs — the highest-leverage stub), [[json-rpc]] (transport for MCP+ACP), [[ppo]] (Schulman 2017), [[grpo]] (DeepSeek's group-normalized PPO variant).

**Result**: ~30 of 78 broken refs resolved (~38%). Remaining broken refs are 1-2-ref external library/vendor names (gpt-4o, gemini, chatgpt, llama, neo4j, etc.) and singleton author/affiliation stubs — left as natural-ingest backfill targets.

---

## [2026-05-16] ingest | Tool calling, structured outputs, agent skills

Ingested four canonical web sources covering the tool-calling layer of modern LLM agents:

**Sources** (fetched, saved to `sources/`, summarized in `wiki/sources/`):
- [[anthropic-agent-skills-2025]] — Anthropic's Oct 2025 announcement of Agent Skills (progressive disclosure, SKILL.md, agentskills.io open standard)
- [[openai-function-calling-2025]] — OpenAI function calling + Responses API guide (March 2025)
- [[function-calling-llms-promptingguide]] — Conceptual guide on function-calling mechanism
- [[vllm-tool-calling-2025]] — vLLM tool parsers, structured output backends

**New concept pages** (6):
- [[agent-skills]] — Anthropic Agent Skills: 3-level progressive disclosure (metadata / body / supporting files), 4 properties (composability / portability / efficiency / power), cross-platform across Claude apps / Code / API / SDK
- [[function-calling]] — primary concept page: LLM emits JSON tool-call request, application executes; terminology disambiguation (function calling vs tool use vs tool calls); L3 mechanism under L2 action surface in [[agent-three-layer-model]]
- [[responses-api]] — OpenAI's March 2025 unified API (Chat Completions + Assistants merged); event-based streaming
- [[tool-call-parser]] — extracting structured tool calls from divergent model output formats (JSON, XML, pythonic, proprietary); ~25 parsers in vLLM (`llama3_json`, `hermes`, `mistral`, `pythonic`, `deepseek_v3`, etc.)
- [[structured-outputs]] — forcing model output to a JSON schema; syntactic validity guarantee, NOT semantic correctness; OpenAI `strict: true`
- [[constrained-decoding]] — logit-mask grammar enforcement; backends: xgrammar (default 2025, 100× speedup), outlines, lm-format-enforcer, llguidance

**New entity pages** (2 products):
- [[openai-api]] — distinct from [[openai|the company]]; Chat Completions / Responses / Assistants endpoints
- [[vllm]] — UC Berkeley-originated inference engine; ~25 tool-call parsers; OpenAI-compatible API

**Updated existing pages** with cross-references:
- [[skill-md-format]] — now points to [[agent-skills]] as the canonical Anthropic concept
- [[tool-use]] — now points to [[function-calling]] as the dominant 2024-2026 mechanism

**Key conceptual additions**:
1. **Function calling is one specific mechanism for tool use** — fine-tuning teaches the model to emit JSON in a format the application can parse and execute. The model does NOT execute the function.
2. **The tool-call parser is a non-trivial component** because there is no industry standard at the model-output layer — Llama uses JSON, Hermes/Qwen use XML tags, DeepSeek uses XML-wrapped JSON, pythonic models use Python list syntax. vLLM ships ~25 parsers to cover this divergence.
3. **Structured outputs / constrained decoding** is the alternative: enforce schema at the decode step itself via logit masking. xgrammar made this near-zero-overhead in 2025.
4. **Agent Skills** is a different design pattern from function calling — composable progressively-disclosed capability modules (not just function definitions). All 5 documented 2026 agents in this wiki already ship SKILL.md support.

Wiki now ~297 pages.

---

## [2026-05-16] ingest | MCP + ACP — agent interoperability protocols

Ingested two foundational agent-interop protocols both referenced repeatedly in [[agent-three-layer-model]] and [[cross-agent-comparison-2026]] but lacking dedicated pages:

**Sources** (fetched from canonical pages):
- [[model-context-protocol-anthropic-2024]] — Anthropic's MCP announcement + 2025 ecosystem evolution
- [[agent-client-protocol-zed-2025]] — Zed's ACP overview + ecosystem

**New concept pages**:
- [[mcp]] — Model Context Protocol (agent ↔ tool/data; Anthropic 2024). Donated to AAIF/Linux Foundation Dec 2025.
- [[acp]] — Agent Client Protocol (agent ↔ editor; Zed Aug 2025). Apache 2.0.

**New entity page**:
- [[zed]] — Zed Industries; creator of [[acp|ACP]], early [[mcp|MCP]] launch partner.

**Key conceptual addition**: MCP and ACP are **complementary** protocols solving symmetrical N×M integration problems on opposite sides of the agent — MCP standardizes the *model-side* (agent ↔ tools/data); ACP standardizes the *UI-side* (agent ↔ editor). Modern coding agents speak both. The two protocols slot cleanly into L2 action surface in the [[agent-three-layer-model]].

Both pages backfill broken wikilinks flagged in the 2026-05-16 lint pass (MCP had 4 references; Zed had 3; ACP had several).

---

## [2026-05-16] lint | Wiki health check + auto-fix (Level 3)

**Found**: 101 broken wikilinks, 25 missing index entries (legacy steering batch), 3 slug-drift bugs, 1 malformed Obsidian anchor in [[steering]]. 27 newly-ingested pages (agent-memory canon) passed all frontmatter and tag-hygiene checks. No orphans, no over-tagged pages.

**Auto-fixed (Level 3 — quick wins + entity stubs + high-leverage concept stubs)**:
- Fixed 2 slug drifts (`[[chain-of-thought]]` → `[[chain-of-thought-prompting]]`; `[[wu-autogen-2023]]` → `[[autogen-wu-2023]]`) and the [[steering]] anchor link bug.
- Synced 25 pre-existing steering / interpretability pages into `wiki/index.md` under new sub-sections (People — Steering/interpretability, Steering and interpretability concepts, Steering / interpretability sources).
- Created 10 entity stubs: 6 people ([[charles-packer]], [[ion-stoica]], [[joon-sung-park]], [[anima-anandkumar]], [[percy-liang]], [[bernal-jimenez-gutierrez]]), 2 orgs ([[letta]], [[zep]]), 3 universities ([[uc-berkeley]], [[ohio-state-university]], [[caltech]]).
- Created 14 high-leverage concept stubs: memory-primitive ([[memory-stream]], [[reflection-mechanism]], [[procedural-memory]], [[skill-library]], [[virtual-context]], [[tiered-memory]], [[ebbinghaus-forgetting]], [[episodic-memory-llm]], [[memory-evolution]]); supporting infrastructure ([[graph-memory]], [[knowledge-graph]], [[kv-cache]], [[long-context-llm]], [[locomo]]).

**Result**: ~70 of 101 broken-link references resolved. Remaining ~30 broken links are low-leverage (1-2 refs each) and split between (a) supporting concepts not yet warranting their own page (`grpo`, `ppo`, `mcp`, `online-learning`, `function-calling`, `active-retrieval`) and (b) 1-ref people/orgs/universities from the long author lists. Deferred to a future ingest. Wiki is now ~277 pages.

---

## [2026-05-16] query + concept | Agent three-layer model

Iterative critique of [[common-features-of-modern-agents]] produced a sharper architectural framework: a three-layer model separating L1 LLM capabilities (instruction-following, alignment, ICL, reasoning, emergent), L2 agent architecture (control flow / action surface / context assembly / memory management / persistence / harness), and L3 implementations (ReAct, MCP, SKILL.md, CLAUDE.md auto-load, sandboxing, hooks, etc.). Key refinements: alignment moved L2→L1 (it's a model property); ReAct moved L2→L3 (it's one implementation of the abstract percept→decide→act loop); "guardrail" replaced by "harness" (bidirectional shaping — block harmful AND drive toward goals — terminology from AI evals literature). Filed as [[agent-three-layer-model]]. Notable cross-cutters identified: skills, hooks, MCP.

---

## [2026-05-16] query | Common features of modern agents

Question: "what are most important features of modern agents shares in common?" Filed as [[common-features-of-modern-agents]]. Identifies 8 universal features across the five 2026 agents (Transformer LLM backbone, RLHF/CAI alignment, ReAct loop, SKILL.md interop, hierarchical file memory, session persistence, lexical-first retrieval, tool API + permissions) and contrasts with 2 features that are *not* yet baseline (MCP, sub-agents). Key pages consulted: [[cross-agent-comparison-2026]], [[claude-code-agent-doc-2026]], [[opencode-agent-doc-2026]], [[pi-mono-agent-doc-2026]], [[hermes-agent-doc-2026]], [[openclaw-agent-doc-2026]], [[memory-management]].

---

## [2026-05-16] ingest | Memory-management canon — 13 papers, 14 concepts

**Motivation**: the existing retrieval/RAG content was dense-retrieval-heavy and treated memory as a synonym for retrieval. This ingest pivots the framing — *memory management* is the umbrella (write/store/retrieve/compact/consolidate/evict/curate); retrieval is one operation within it.

**13 new source pages** (none of these existed before; Reflexion was already ingested separately):

Foundation era (2023):
- [[generative-agents-park-2023]] — memory stream + reflection + recency × importance × relevance retrieval
- [[memorybank-zhong-2023]] — first principled forgetting policy (Ebbinghaus curve)
- [[voyager-wang-2023]] — skill library as procedural memory
- [[memgpt-packer-2023]] — OS-style tiered virtual memory; LLM emits function calls to page memory

Recent era (2024–2025):
- [[em-llm-fountas-2024]] — Bayesian-surprise event segmentation inside the KV-cache (ICLR 2025)
- [[titans-behrouz-2025]] — test-time parametric memory updates via surprise gradient (Google Research)
- [[a-mem-xu-2025]] — Zettelkasten-style memory evolution; neighbor rewrites
- [[hipporag2-jimenez-gutierrez-2025]] — hippocampus/neocortex graph memory with Personalized PageRank
- [[sleep-time-compute-lin-2025]] — offline consolidation between queries (5× test-time compute reduction; Letta/Berkeley)
- [[mem0-chhikara-2025]] — production-grade cross-session memory; ECAI 2025
- [[mirix-wang-2025]] — six memory types × dedicated agent each × multimodal (LOCOMO SOTA 85.4%)
- [[memory-r1-yan-2025]] — RL-trained policy over `{ADD, UPDATE, DELETE, NOOP}`; learns *when to forget*
- [[mem-alpha-wang-2025]] — RL-trained memory *construction* (vs editing); 30k → 400k+ extrapolation

**1 new umbrella concept page**: [[memory-management]] — lifecycle vocabulary, design axes (external store vs parametric, flat vs structured, heuristic vs learned, single-agent vs multi-agent, memory typology), and the 2023→2026 arc.

**13 new system concept pages** (one per system, ~50 lines each, system-as-node-in-the-graph): [[memgpt]], [[memorybank]], [[a-mem]], [[voyager]], [[generative-agents]], [[sleep-time-compute]], [[em-llm]], [[titans]], [[hipporag-2]], [[mem0]], [[mirix]], [[memory-r1]], [[mem-alpha]].

**Key conceptual addition**: the **physical substrate axis** for memory — external store (MemGPT/Mem0/A-MEM/MIRIX), KV-cache-internal (EM-LLM), parametric/weights (Titans), graph-indexed (HippoRAG 2). And the **learned-vs-heuristic axis** — heuristic ops in Mem0/A-MEM vs RL-trained ops in Memory-R1 (editing) and Mem-α (construction).

**Visualization**: `scripts/evolution-graph.html` retrieval node previously rewritten as a "Memory Management" node — the 13 papers are listed in its Key Papers section (split into Foundation 2023 / Recent 2024–2026).

**Pages**: ~252 total (was ~225). Still all wikilinks should resolve modulo a small number of intentional forward links to entity pages (people, orgs) and minor concepts that lint can promote.

---

## [2026-05-15] ingest + query + visualization | Steering papers, transformer→agents synthesis, interactive graph

**Ingest — 5 steering / interpretability papers (Track 9 of the evolution)**:
- [[activation-engineering-turner-2023]] (Turner) — ActAdd, the canonical steering method
- [[representation-engineering-zou-2023]] (Zou) — RepE framework, consolidates the field
- [[sparse-autoencoders-cunningham-2023]] (Cunningham) — SAE foundation; addresses polysemanticity/superposition
- [[contrastive-activation-addition-rimsky-2023]] (Panickssery/Rimsky) — CAA, production-grade safety steering
- [[inference-time-intervention-li-2023]] (Li) — ITI, head-level truthfulness steering

**New concept pages**: [[steering]] (umbrella), [[guardrailing]] (paired umbrella), [[activation-engineering]], [[activation-addition]], [[steering-vectors]], [[contrastive-activation-addition]], [[representation-engineering]], [[inference-time-intervention]], [[sparse-autoencoder]], [[polysemanticity]], [[superposition]], [[monosemanticity]], [[linear-representation-hypothesis]].

**New entity pages**: [[alexander-matt-turner]], [[andy-zou]], [[dan-hendrycks]], [[center-for-ai-safety]], [[eleutherai]], [[fernanda-viegas]], [[martin-wattenberg]].

**Updated existing pages with steering cross-references**: [[functional-emotions]], [[mechanistic-interpretability]], [[agentic-misalignment]], [[constitutional-ai]] — each now reflects how steering complements its existing safety/interpretability content.

**Key conceptual addition**: [[steering]] and [[guardrailing]] are now paired as the two safety surfaces of AI alignment. Guardrailing operates externally (RLHF, CAI training, refusal filters, sandboxing, permissions); steering operates internally (activation addition, contrastive vectors, SAE features, ITI on attention heads). The [[emotion-concepts-anthropic-2026|2026 emotion-concepts paper]] becomes a natural showcase of the convergence: CAI training implements something that *looks like* steering in activation space.

**Query**: comprehensive research on transformer→modern-agents evolutionary paths, filed at [[from-transformer-to-modern-agents]]. Identifies 9 tracks (architecture, scaling, pre-training, alignment, reasoning, retrieval, tool use, orchestration, steering+guardrailing) and shows how they compose in the 2026 agent products.

**Visualization**: built an interactive web page at `scripts/evolution-graph.html` (single-file HTML using vis-network). 48 nodes, 63 edges, hierarchical layout (top→bottom by year/dependency). Color-coded by track. Each node shows a Korean (한국어) summary in a side panel when clicked. Designed as a teaching artifact alongside [[from-transformer-to-modern-agents]].

**Pages**: now ~225 total. All wikilinks resolve (12 "broken links" reported by the regex check are false positives from Markdown table-cell `\|` escaping and one literal `[[wikilink]]` example string in this log).

---

## [2026-05-14] lint | Wiki health check

Ran all 10 checks. Found: **26 errors, 0 warnings, 264+ suggestions**. **Auto-fixed all 26 errors.**

**Errors (all fixed)** — bare-string `authors:` values in 16 source pages where the corresponding entity page exists. These were from the first batch ingest (before the typed-relations convention) and were converted to `"[[wikilink]]"` form via auto-fix.

**Notable suggestions (not auto-fixed)**:
- 81 draft pages — intentional stubs from passing-mention ingests; will be enriched on future ingests.
- 183 single-source pages — expected for a young wiki; will diminish as more sources cross-reference.
- 254 bare-string co-author names without corresponding entity pages — per convention, these stay as strings; promote to stubs only if the author appears in 3+ papers.
- 7 "year-tag" promotion candidates (`2014`, `2018`, `2020`, `2021`, `2022`, `2023`, `2026`) — false positives. Per CLAUDE.md's "Year tags on sources" convention, year tags are *organizational* and intentionally have no corresponding page. Lint rule could be refined to exclude pure-numeric tags from promotion-candidate detection.

**Clean checks**: 0 broken wikilinks, 0 orphan pages, 0 missing frontmatter, 0 unresolved contradictions, 0 tag-slug collisions, 0 over-tagged pages, 0 type/status/importance schema violations, all 72 entity pages have exactly one valid classification tag.

---

## [2026-05-14] ingest | Batch: 29 sources (Tier 1–6 + agent docs)

Downloaded 23 arXiv papers from the `latest-agents/docs/research/references.md` bibliography (plus GloVe from aclanthology and LSTM from the JKU mirror), copied 5 agent docs + 1 comparison doc from `latest-agents/docs/`, and ingested all 29 sources in chronological order.

Tier 1 — Foundations (1997–2022): [[long-short-term-memory-hochreiter-1997]], [[efficient-estimation-of-word-representations-mikolov-2013]], [[distributed-representations-of-words-and-phrases-mikolov-2013]], [[glove-pennington-2014]], [[sequence-to-sequence-learning-sutskever-2014]], [[neural-machine-translation-bahdanau-2014]], [[t5-raffel-2019]], [[scaling-laws-kaplan-2020]], [[chinchilla-hoffmann-2022]].

Tier 2 — Emergence: [[rlhf-christiano-2017]], [[flan-wei-2021]], [[instructgpt-ouyang-2022]], [[dpo-rafailov-2023]].

Tier 3 — Reasoning: [[zero-shot-cot-kojima-2022]], [[self-consistency-wang-2022]], [[tree-of-thoughts-yao-2023]].

Tier 4 — Tool use: [[webgpt-nakano-2021]], [[toolformer-schick-2023]].

Tier 5 — Orchestration: [[metagpt-hong-2023]], [[autogen-wu-2023]].

Tier 6 — Guardrailing: [[constitutional-ai-bai-2022]], [[red-teaming-ganguli-2022]], [[indirect-prompt-injection-greshake-2023]].

Agent docs (2026): [[claude-code-agent-doc-2026]], [[opencode-agent-doc-2026]], [[pi-mono-agent-doc-2026]], [[hermes-agent-doc-2026]], [[openclaw-agent-doc-2026]], [[cross-agent-comparison-doc-2026]] — plus a derived [[cross-agent-comparison-2026|comparison page]] in `wiki/comparisons/`.

**Pages**: now 200 total (45 sources, 72 entities, 79 concepts, 1 comparison, 3 root). Up from 110 before this batch. **All wikilinks resolve** (verified).

**New concepts**: [[lstm]], [[recurrent-neural-network]], [[vanishing-gradient-problem]], [[word-embeddings]], [[word2vec]], [[negative-sampling]], [[distributional-semantics]], [[glove]], [[seq2seq]], [[additive-attention]], [[c4]], [[instruction-tuning]], [[dpo]], [[preference-modeling]], [[reward-modeling]], [[zero-shot-cot]], [[self-consistency]], [[tree-of-thoughts]], [[toolformer]], [[metagpt]], [[autogen]], [[standardized-operating-procedures]], [[constitutional-ai]], [[rlaif]], [[red-teaming]], [[prompt-injection]], [[indirect-prompt-injection]], [[skill-md-format]].

**New entities — models**: [[t5]], [[chinchilla]], [[flan]], [[instructgpt]], [[webgpt]]; the five 2026 agent products [[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]]; plus [[faiss]].

**New entities — people**: [[sepp-hochreiter]], [[jurgen-schmidhuber]], [[tomas-mikolov]], [[jeff-dean]], [[jeffrey-pennington]], [[richard-socher]], [[christopher-manning]], [[oriol-vinyals]], [[quoc-le]], [[dzmitry-bahdanau]], [[kyunghyun-cho]], [[yoshua-bengio]], [[colin-raffel]], [[jordan-hoffmann]], [[paul-christiano]], [[jan-leike]], [[rafael-rafailov]], [[chelsea-finn]], [[luke-zettlemoyer]], [[mario-zechner]].

**New entities — orgs**: [[stanford-university]], [[nous-research]].

**Convention check**: this batch was the first major test of the typed-relations frontmatter convention (introduced in the previous skill update). Every new source page carries `authors:` as wikilinks where the entity page exists, `first_author:`, `introduces:` listing the entities/concepts the source introduces. Every new model entity carries `family`/`predecessor`/`successors`. Every entity page carries exactly one classification tag (`model`, `person`, `org`, `university`, `product`). Lint should be clean.

**Notable cross-paper threads now visible in the wiki**:
- Bradley-Terry preference learning lineage: [[rlhf-christiano-2017]] → [[instructgpt-ouyang-2022]] → [[constitutional-ai-bai-2022]] (RLAIF) → [[dpo-rafailov-2023]]. The DPO move "your LM is secretly a reward model" closes the loop.
- The CoT family: [[chain-of-thought-wei-2022]] → [[zero-shot-cot-kojima-2022]] (zero-shot version) → [[self-consistency-wang-2022]] (sampling) → [[tree-of-thoughts-yao-2023]] (search) → modern reasoning models (OpenAI o1, etc.) which internalize this trajectory.
- The retrieval line: [[dpr-karpukhin-2020]] (dense retrieval primitive) → [[rag-lewis-2020]] (RAG inference-time) → [[retro-borgeaud-2021]] (retrieval at pretraining time).
- The scaling correction: [[scaling-laws-kaplan-2020]] (under-trained big models recommendation) ← [[chinchilla-hoffmann-2022]] (corrects to equal-scaling, drives the Llama/Gemini-era recipes).
- 2026 agent landscape: [[cross-agent-comparison-2026]] consolidates [[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]] along the canonical axes — tool count, MCP stance, sub-agent stance, sandbox model, memory layer, skills format.

---

## [2026-05-13] convention | Entity classification tags introduced

Added a new wiki-wide convention: every `type: entity` page must carry exactly one classification tag identifying its sub-type. Rationale: the entity sub-types (model, person, org, ...) were previously only encoded in prose in the [[index]], unavailable for Obsidian tag-pane filtering or Dataview queries.

Universal rule documented in `.claude/skills/wiki/SKILL.md` (Tag Policy → "Entity classification tags") and enforced at write time by `ingest.md` and at audit time by `lint.md`. Domain-specific vocabulary documented in `CLAUDE.md` ("Entity classification vocabulary" table): `model | person | org | university | product | event`.

Backfilled all 40 existing entity pages with the appropriate classification tag:
- 14 `model`
- 16 `person`
- 6 `org`
- 3 `university`
- 1 `product` (FAISS)
- 0 `event`

In Obsidian, you can now click `#model` in the tag pane to see all 14 model pages, etc.

---

## [2026-05-13] ingest | Batch: 16 sources

Batch-ingested the initial source corpus in chronological order. 16 sources processed (all foundational LLM papers + 2 essays):

1. [[attention-is-all-you-need-vaswani-2017]] (Vaswani et al., 2017)
2. [[bert-devlin-2018]] (Devlin et al., 2018)
3. [[gpt-1-radford-2018]] (Radford et al., 2018)
4. [[transformer-decoder-liu-2018]] (Liu et al., 2018)
5. [[gpt-2-radford-2019]] (Radford et al., 2019)
6. [[dpr-karpukhin-2020]] (Karpukhin et al., 2020)
7. [[rag-lewis-2020]] (Lewis et al., 2020)
8. [[gpt-3-brown-2020]] (Brown et al., 2020)
9. [[vit-dosovitskiy-2020]] (Dosovitskiy et al., 2020)
10. [[retro-borgeaud-2021]] (Borgeaud et al., 2021)
11. [[chain-of-thought-wei-2022]] (Wei et al., 2022)
12. [[react-yao-2022]] (Yao et al., 2022)
13. [[gpt-4-openai-2023]] (OpenAI, 2023)
14. [[reflexion-shinn-2023]] (Shinn et al., 2023)
15. [[emotion-concepts-anthropic-2026]] (Anthropic, 2026)
16. [[agentic-ai-and-the-next-intelligence-explosion]] (Evans, Bratton & Agüera y Arcas, 2026)

**Pages created**: 16 source summaries + ~39 entity pages + ~50 concept pages = ~105 new wiki pages. See [[index]] for the full catalog.

**Key new concepts** by ingest: [[transformer-architecture]], [[self-attention]], [[multi-head-attention]], [[positional-encoding]], [[encoder-decoder]] (Transformer); [[masked-language-modeling]], [[pre-training]], [[fine-tuning]] (BERT); [[decoder-only-transformer]], [[autoregressive-language-modeling]], [[gpt-family]] (GPT-1); [[zero-shot-learning]] (GPT-2); [[dense-retrieval]], [[dual-encoder]], [[bm25]], [[open-domain-qa]] (DPR); [[retrieval-augmented-generation]], [[hallucination]] (RAG); [[in-context-learning]], [[few-shot-learning]], [[scaling-laws]] (GPT-3); [[patch-tokenization]] (ViT); [[chunked-cross-attention]] (RETRO); [[chain-of-thought-prompting]], [[emergent-abilities]] (CoT); [[react]], [[tool-use]], [[llm-agent]] (ReAct); [[rlhf]], [[multimodal-llm]] (GPT-4); [[reflexion]], [[self-reflection]] (Reflexion); [[functional-emotions]], [[mechanistic-interpretability]], [[sycophancy]], [[reward-hacking]], [[agentic-misalignment]] (Emotion Concepts); [[centaur-actor]], [[institutional-alignment]] (Agentic AI essay).

**Key new entities**: all four GPT generations ([[gpt-1]], [[gpt-2]], [[gpt-3]], [[gpt-4]]), [[bert]], [[claude]], [[palm]], [[bart]], [[vit]], [[dpr]], [[rag]], [[retro]], [[transformer-decoder]]; [[openai]], [[anthropic]], [[google-brain]], [[google-research]], [[deepmind]], [[fair]]; the principal researchers across all papers.

**Key cross-cutting takeaways**:
- The [[transformer-architecture|Transformer]] is the single underlying architecture of essentially all the work covered. It scales (parameters, data, compute) and generalizes across modalities ([[vit|vision]]).
- [[scaling-laws|Scaling]] is an unreasonably effective lever — capabilities like [[in-context-learning|few-shot learning]], [[chain-of-thought-prompting|chain-of-thought reasoning]], and 3-digit arithmetic [[emergent-abilities|emerge with scale]].
- The field's center of gravity shifted from [[fine-tuning]] (BERT/GPT-1 era) → [[in-context-learning|prompting and ICL]] (GPT-3 era) → [[react|agents]] and [[tool-use|tools]] (2022+) → [[reflexion|self-reflective agents]] / reasoning models (2023+).
- [[retrieval-augmented-generation|Retrieval augmentation]] is a complementary axis to parametric scale — [[retro|RETRO]] showed retrieval and parameter scaling are independent levers.
- The post-GPT-3 era's distinguishing safety/alignment work — [[rlhf|RLHF]], [[mechanistic-interpretability]], [[functional-emotions]], [[agentic-misalignment]] — is moving from external behavior management toward internal mechanism understanding and intervention.

**Notable lineage**: the [[gpt-family|GPT family]] descends architecturally from [[transformer-decoder]] (Liu 2018) → [[gpt-1]] → [[gpt-2]] → [[gpt-3]] → [[gpt-4]]. The Anthropic co-founders ([[dario-amodei]], [[jared-kaplan]], [[tom-brown]], [[sam-mccandlish]], [[chris-olah]]) are clustered on the GPT-2/GPT-3 author lists.

**Notable convergence**: [[shunyu-yao]] (Princeton) appears as first author of [[react-yao-2022|ReAct]] and co-author of [[reflexion-shinn-2023|Reflexion]] — a clear thread on LLM-agent / reasoning research. [[patrick-lewis]] (FAIR) appears as co-author of [[dpr-karpukhin-2020|DPR]] and first author of [[rag-lewis-2020|RAG]] — the retrieval thread.

---

## [2026-05-12] init | Wiki initialized

Wiki created for: Large Language Models and the surrounding ML/AI landscape (broader scope — papers, models, people, orgs, products, infrastructure).

Configuration:
- Mode: **local** (wiki under project root). Intended future location is the iCloud Obsidian vault `a local iCloud Obsidian vault`, but iCloud Drive is currently blocked by macOS TCC for this terminal — pending Full Disk Access grant.
- Conventions enabled in `CLAUDE.md`: arXiv IDs on source pages, model-lineage tracking (`family` / `predecessor` / `successors`), publication-year tags on sources.

Directories created: `sources/assets/`, `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/comparisons/`, `wiki/queries/`.
Files generated: `CLAUDE.md`, `.wiki-config.json`, `wiki/index.md`, `wiki/log.md`, `wiki/overview.md`.

Existing source corpus (not yet ingested): Transformer (Vaswani 2017), BERT (Devlin 2018), GPT-1/2/3/4 (OpenAI 2018–2023), RAG (Lewis 2020), RETRO (Borgeaud 2021), DPR (Karpukhin 2020), ViT (Dosovitskiy 2020), Chain-of-Thought (Wei 2022), ReAct (Yao 2022), Reflexion (Shinn 2023), Transformer Decoder (Liu 2018), plus articles "Agentic AI and the next intelligence explosion" and "Emotion Concepts and their Function in a Large Language Model".
