---
title: Activity Log
type: log
created: 2026-05-12
updated: 2026-05-20
---

# Activity Log

Chronological record of wiki operations. Newest entries first.

## [2026-05-25] ingest | Code-as-Harness — survey + 8 cited papers (2022–2026)

Ingest of the major 2026 survey [[code-as-agent-harness-ning-2026]] (Ning et al., UIUC/Meta/Stanford, arXiv:2605.18747, 66 pages, 500+ references) plus its **8 most-citable / load-bearing references** that were not yet in the wiki.

**Survey thesis**: code is the *operational substrate* of the agent harness, not merely an output the LLM produces. Programs are executable (verifiable), inspectable (structured traces), and stateful (persistent across steps) — unifying reasoning, action, environment modeling, and verification under one abstraction. The bottleneck of autonomy is not the base model but harness reliability.

**Sources (9 new)**:
- [[code-as-agent-harness-ning-2026]] — the survey itself
- [[pal-gao-2023]] — PAL: Program-aided Language Models (ICML 2023) — origin of code-for-reasoning
- [[code-as-policies-liang-2022]] — Code as Policies (Liang et al., Google Robotics, ICRA 2023) — origin of code-for-acting in embodied settings
- [[codeact-wang-2024]] — Executable Code Actions Elicit Better LLM Agents (Wang et al., UIUC, ICML 2024) — Python as action space, +20% absolute over JSON tool calls
- [[swe-agent-yang-2024]] — SWE-agent (Yang et al., Princeton, NeurIPS 2024) — Agent-Computer Interface abstraction; SWE-bench 12.5%
- [[openhands-wang-2024]] — OpenHands open-source platform (Wang et al., UIUC/CMU, ICLR 2025); SWE-bench Lite 26.7%
- [[agentcoder-huang-2023]] — Programmer + Test-Designer + Test-Executor triad (Huang et al., HKU 2023); most-cited paper in the survey body (19 citations)
- [[chatdev-qian-2023]] — Communicative agents for SW dev (Qian et al., Tsinghua/Modelbest, ACL 2024)
- [[agentic-harness-engineering-lin-2026]] — Telemetry-driven harness evolution (Lin et al., Fudan/Peking, 2026); 69.7% → 77.0% on Terminal-Bench 2 over 10 iterations

**New concept (1)**: [[code-as-harness]] — the umbrella concept the survey names; cross-links all 9 sources.

**Key conceptual additions**:
1. **Code as the L2 substrate**. Every L2 abstraction in [[agent-three-layer-model]] (control flow, action surface, context, memory, persistence, harness) can be reinstantiated with code as the medium — this is an *axis* across L2, not a layer.
2. **Three-element decomposition**: model-internal capabilities (L1) vs. system-provided harness infrastructure vs. **agent-initiated code artifacts** (the novel third category — tests, temporary tools, DSL programs, intermediate program states).
3. **Verifiable-by-running collapses the reward-model pipeline**. Where correctness is mechanically checkable, the reward model disappears entirely — pairs with [[deepseek-r1]] / [[search-r1]] / [[swe-gym]] outcome-RL recipes.
4. **Agentic Harness Engineering** (Lin 2026) is the 2026 frontier: the harness itself becomes the object of optimization. Empirical descendant of [[godel-machine]] / [[darwin-godel-machine]] applied at the *harness* level rather than the agent level. Pairs with [[agentic-context-engineering]] (prompt-level evolution).

**Wiki size**: 397 → 406 pages.

**Open dangling refs** (left for future passes): [[evolution-agent]], [[plan-execute-verify-loop]], [[code-world-model]], [[program-of-thoughts]], [[agent-skills]] (already exists; checked) — surfaced by the survey but not the prime target.

---

## [2026-05-22] query | Policy optimization methods

Filed [[policy-optimization-methods]] — a cross-cutting synthesis answering "what is policy optimization, what entities/concepts belong, how do they differ?"

**Coverage** (drawn from 9 concept pages + 7 source pages already in the wiki):
- **RL family**: [[ppo]] (Schulman 2017), [[grpo]] (DeepSeek 2024), TRPO, REINFORCE, RLOO
- **Direct-preference family**: [[dpo]] (Rafailov 2023), IPO, KTO, ORPO, SimPO
- **Pipelines**: [[rlhf]] (Christiano/Ouyang), [[rlaif]] / [[constitutional-ai]] (Bai 2022), [[agentic-rl]] (DeepSeek-R1, Search-R1, SWE-Gym)
- **Underlying machinery**: [[preference-modeling]] (Bradley-Terry), [[reward-modeling]]

**Key conceptual takeaways**:
1. All methods optimize the same KL-regularized objective `E[r(x,y)] − β·KL(π_θ ‖ π_ref)` — they differ in *how reward is sourced* and *how the loop is structured*.
2. PPO vs GRPO: GRPO drops the value network for a per-prompt group baseline. Cheaper, more stable, now the default for reasoning/agent RL.
3. DPO vs PPO/GRPO: DPO is one supervised loss on preference pairs (no on-policy sampling). Cost ≈ plain SFT.
4. **2023–2026 trajectory = eliminating moving parts**: drop the critic (GRPO), drop the reward model (DPO), drop the human entirely (outcome-RL like DeepSeek-R1).

**Pages updated**: `wiki/index.md` (Queries section + count).

---

## [2026-05-21] lint | Level-2 auto-fix (errors + warnings)

Lint audit found 3 error classes, 4 warnings, 10 suggestions across 387 pages. Level-2 auto-fix executed.

**Errors resolved**:
- **23 backfilled pages added to `wiki/index.md`** under Sources/Concepts/Entities/Looped-Transformers/Foundations sections.
- **9 stubs created** for top broken-link targets:
  - Concepts: [[chain-of-thought]] (soft alias to [[chain-of-thought-prompting]]), [[online-learning]], [[adas]], [[agent-benchmark]], [[rasp-l]]
  - Persons: [[kangwook-lee]] (UW-Madison; LT thread senior author), [[nikunj-saunshi]] (Google Research; latent-thoughts framing), [[dimitris-papailiopoulos]] (UW-Madison; LT thread)
  - Org: [[bytedance-seed]] (ByteDance's foundation-model research division)
- **2 frontmatter wikilink targets cleaned**: removed unresolvable `[[self-evolving-context]]` and `[[looped-language-model]]` from `introduces:` fields where the existing concept pages already cover the territory.

**Warnings resolved**:
- **4 orphan pages back-linked from peer pages**:
  - [[jeff-dean]] ← [[google-brain]] (notable-affiliations bullet)
  - [[downsides-of-looped-transformers]] ← [[looped-transformer]] (Related section)
  - [[hooks-in-modern-agents]] ← [[agent-three-layer-model]] (References section)
  - [[agentic-misalignment-lynch-2025]] ← [[agentic-misalignment]] (References section)

**Tag hygiene + frontmatter + contradictions**: all clean (no fixes needed).

**Impact**:
- Top dangling-ref count: **6 → 4** (`chain-of-thought` resolved by alias stub; person/org refs resolved by entity stubs)
- Orphan pages: **4 → 0**
- Total broken wikilink references: **266 → ~240** (still many specific person/concept names referenced once or twice — left for future passes)
- Wiki size: **387 → 396 pages**.

**False alarm noted**: the lint agent initially claimed all 107 entity pages lacked classification tags. Verified directly — 107/107 have a valid tag from `{model, person, org, university, product, event}`. The agent's regex was wrong.

**Top draft pages by inbound link count** (not auto-fixed; future ingest material):
[[google-research]] (31), [[tool-use]] (30), [[google-brain]] (24), [[grpo]] (22), [[dario-amodei]] (20), [[ilya-sutskever]] (19), [[swe-gym]] (18), [[bm25]] (18), [[fair]] (18), [[ppo]] (16).

---

## [2026-05-21] ingest | Backfill — 5 sources + 19 concept stubs for top-cited dangling refs

Wiki audit found 144 draft pages and 163 dangling wikilinks. Top dangling refs (15× `swe-gym`, 12× `chronos`, 10× `set-of-mark`/`adaptive-computation-time`, etc.) were back-filled in one pass.

**New sources (5)** — papers cited from many existing pages but never ingested:
- [[chronos-sen-2026]] — Sen et al., PwC 2026 (arXiv:2603.16862) — temporal-event memory agent; companion to [[grep-all-you-need-sen-2026]]; Chronos-High hits **95.6% on LongMemEval-S**
- [[swe-bench-jimenez-2023]] — Jimenez et al., Princeton, ICLR 2024 (arXiv:2310.06770) — the foundational coding-agent benchmark; 2,294 issue→PR pairs across 12 Python repos
- [[godel-machine-schmidhuber-2003]] — Schmidhuber (arXiv:cs/0309048) — the theoretical ancestor of [[darwin-godel-machine]]; proof-driven self-rewriting with no local maxima
- [[ui-tars-1-qin-2025]] — Qin et al., ByteDance Seed (arXiv:2501.12326) — predecessor of [[ui-tars-2]]; the original native GUI agent (Qwen-2-VL backbone, OSWorld 24.6)
- [[adaptive-computation-time-graves-2016]] — Graves, DeepMind (arXiv:1603.08983) — ACT mechanism; the foundation reused by every looped/recurrent-depth transformer since

**New concept pages (18)**:
- LT thread: [[chronos]], [[adaptive-computation-time]], [[godel-machine]], [[ui-tars-1]], [[swe-gym]], [[swe-bench]], [[parcae]], [[inference-time-compute]], [[reinforcement-learning]], [[set-of-mark]], [[trace-of-mark]], [[timestep-encoding]], [[length-generalization]], [[grokking]], [[parameter-sharing]]
- Agent thread: [[gui-agent]], [[embodied-agent]], [[osworld]]

**New entity (1)**: [[microsoft]] — parent of [[microsoft-research]].

**Impact**: top-dangling-ref count for the highest-traffic ref dropped from **15 → 6** (`chain-of-thought` is now the leader, easily fixed by a soft alias to existing [[chain-of-thought-prompting]] in a future pass). Wiki size: 362 → 387 pages.

---

## [2026-05-21] query | Downsides of Looped Transformer families

Filed [[downsides-of-looped-transformers]] — a cross-source synthesis answering "what are downsides of looped transformer families?"

**8 downside categories identified** (each with ≥1 source citation):
1. Training instability at scale (residual explosion, ρ(A)≥1) — [[parcae-scaling-laws-looped-2026]]
2. Expressivity gap from weight-tying (Lemma 4.1) — [[expressive-power-looped-transformers-xu-2024]]
3. Halting / step-count + overthinking — [[loop-think-generalize-2026]], [[looped-transformers-length-generalization-fan-2024]]
4. Perplexity / memorization loss vs iso-FLOP non-looped — [[latent-thoughts-looped-transformers-saunshi-2025]]
5. OOD simplicity-bias brittleness — [[looped-transformers-learning-algorithms-yang-2023]]
6. Loses to CoT on stochastic / approximate-inference (FPRAS ≠ FPTAS separation) — [[cot-or-loop-xu-2025]]
7. Compute, memory, and recipe complexity — all sources
8. Hand-construction vs. SGD-learnability gap — [[looped-transformers-programmable-computers-giannou-2023]]

**Bottom line**: looped TFs win structurally on deterministic depth-bounded computation and empirically on reasoning, but lose structurally on stochastic inference and need a delicate, still-evolving recipe to train at scale.

**Pages updated**: `wiki/index.md` (Queries section + total count).

---

## [2026-05-20] ingest | Looped Transformers — 10 papers (2018, 2023–2026)

Seeded from [kyegomez/OpenMythos](https://github.com/kyegomez/OpenMythos)'s curated bibliography for the recurrent-depth / looped-transformer thread, supplemented with the canonical theory + empirical papers identified independently.

**Sources (10 new)**:
- [[universal-transformer-dehghani-2018]] — Universal Transformers: recurrent block + ACT; the spiritual ancestor (Dehghani et al., Google Brain/DeepMind, ICLR 2019)
- [[looped-transformers-programmable-computers-giannou-2023]] — Hand-constructed 13-layer looped transformer simulates a programmable computer via FLEQ (Giannou et al., UW-Madison + Princeton, ICML 2023)
- [[looped-transformers-learning-algorithms-yang-2023]] — Looped transformer matches standard on in-context regression at ~1/12 params (Yang, Lee, Nowak, Papailiopoulos, ICLR 2024)
- [[looped-transformers-length-generalization-fan-2024]] — Input-adaptive halting → length generalization on RASP-L-expressible tasks (Fan, Du, Ramchandran, Lee, ICLR 2025)
- [[expressive-power-looped-transformers-xu-2024]] — Approximation-rate bound + timestep encoding fix (Xu & Sato, ICML 2025)
- [[looped-transformers-multi-step-gd-gatmiry-2024]] — Optimal population-loss solution = multi-step preconditioned GD; gradient-dominance convergence (Gatmiry, Saunshi, Reddi, Jegelka, Kumar 2024)
- [[latent-thoughts-looped-transformers-saunshi-2025]] — k×L looping ≈ kL depth; "latent thoughts" implicitly simulate CoT (Saunshi, Dikkala, Li, Kumar, Reddi, ICLR 2025)
- [[cot-or-loop-xu-2025]] — Formal CoT-vs-loop separation; depth-recursion wins on DAG-parallel deterministic tasks, CoT wins on self-reducible/approx-inference (Xu & Sato 2025)
- [[parcae-scaling-laws-looped-2026]] — LTI-system framing + negative-diagonal A → ρ(A)<1 stability; clean scaling laws (Prairie, Novack, Berg-Kirkpatrick, Fu, UCSD/Together AI, 2026)
- [[loop-think-generalize-2026]] — Recurrent-Depth Transformer with implicit reasoning + grokking dynamics (Kohli, Parthasarathy, Sun, Yao, OSU-NLP 2026)

**New concept pages (4)**:
- [[looped-transformer]] — umbrella concept anchoring the thread
- [[universal-transformer]] — the 2018 predecessor as its own concept
- [[recurrent-depth-transformer]] — the modern 2026 rebranding (RDT)
- [[latent-reasoning]] — silent-CoT via depth-recursion; the depth-recursion counterpart to [[chain-of-thought-prompting|token-recursion]]

**New entity (1)**: [[openmythos]] — open-source reference implementation by kyegomez (⭐13k+).

**Key conceptual additions**:
1. **Depth-recursion vs token-recursion** as the central design axis. Looped transformers (depth) and CoT (tokens) are *complementary* — Xu-Sato 2025 formalizes which tasks favor which.
2. **Latent reasoning** as a distinct mechanism from explicit CoT. The same compute happens but no tokens are emitted; o1/DeepSeek-R1-style internal thinking has a depth-recursion analogue.
3. **Looping at pretraining scale is now stable.** Parcae's LTI / ρ(A)<1 recipe + Kohli's tail-only supervision unlock looped LMs at GPT-2/3 scale, matched-PPL beating non-looped at lower params.
4. The thread connects directly to [[deepseek-r1]] / [[agentic-rl]] — both are about scaling test-time compute, just via different recursion axes.

**Wiki size**: 347 → 362 pages.

---



## [2026-05-17] ingest | Is Grep All You Need? (Sen et al., PwC 2026)

Ingest of [[grep-all-you-need-sen-2026]] (arXiv:2605.15184, May 14 2026) — an empirical study of how retriever (grep vs. vector), agent harness (custom vs. provider-native CLI), and tool-result delivery mode (inline vs. file-based) jointly determine end-to-end agentic-search accuracy on a 116-question [[longmemeval|LongMemEval-S]] slice.

**Headline findings**:
- **Inline grep > inline vector for every harness-model pair tested** (10/10). Lexical search is undersold in agentic-RAG literature.
- **Harness effect ≈ retriever effect.** Same Claude Opus 4.6 backbone: 93.1% on [[chronos|Chronos]] vs. 76.7% on [[claude-code|Claude Code]] with grep-only.
- **Tool-delivery mode can invert the lexical advantage.** Programmatic (file-based) delivery makes vector beat grep on 5/10 pairs. Sharpest regression: Codex+GPT-5.4, 93.1% inline grep → 55.2% programmatic grep.

**Pages created (4)**:
- Source: [[grep-all-you-need-sen-2026]]
- Concept: [[agentic-search]] — the umbrella concept the paper names and measures
- Concept stub: [[longmemeval]] — the benchmark (full ICLR 2025 paper not yet ingested)
- Entity: [[pwc]] — PricewaterhouseCoopers, the authors' affiliation

**Index updated** with all 4 new pages.

**Threads it closes**: connects [[memory-management]] retrieval mechanisms ([[bm25]], [[dense-retrieval]], [[hipporag-2]]) to the harness L2 abstraction in [[agent-three-layer-model]] — making explicit that the same retriever performs differently in-loop than as a standalone pipeline. Provides empirical justification for the lexical-search tools (grep, ripgrep, FTS5) that [[claude-code]] and [[pi-mono]] ship with by default.

**Open follow-ups**: [[chronos]] paper (arXiv:2603.16862) cited but not yet ingested. LongMemEval (Wu et al., ICLR 2025) is a stub.

---

## [2026-05-17] ingest | Agentic capabilities — 10 papers (2023, 2025–2026)

Web-search-driven ingest of canonical 2025–2026 agentic-capability papers (plus GAIA as a foundational 2023 benchmark cited by every later agent paper). Verified arXiv IDs on each.

**Sources (10 new)**:
- [[deepseek-r1-2025]] — DeepSeek-R1: pure-RL elicits emergent reasoning; the open-weights substrate for the 2025 agentic-RL wave (arXiv:2501.12948)
- [[search-r1-jin-2025]] — Search-R1: RL recipe for multi-turn tool-use; retrieved-token masking + outcome rewards (arXiv:2503.09516, COLM 2025)
- [[darwin-godel-machine-zhang-2025]] — DGM: empirical self-improving agent; rewrites its own scaffold, SWE-bench 20→50% (arXiv:2505.22954)
- [[ui-tars-2-wang-2025]] — UI-TARS-2: multi-turn RL GUI agent; OSWorld 47.5 (arXiv:2509.02544)
- [[swe-gym-pan-2024]] — SWE-Gym: 2,438 Python tasks with unit tests for SWE-agent training (arXiv:2412.21139, ICML 2025)
- [[swe-bench-live-zhang-2025]] — SWE-bench Goes Live: monthly-refreshed contamination-resistant SWE-bench (arXiv:2505.23419)
- [[agentic-context-engineering-zhang-2025]] — ACE: Generator/Reflector/Curator triad evolves the prompt as a delta-updated playbook (arXiv:2510.04618, ICLR 2026)
- [[agentic-misalignment-lynch-2025]] — Insider-threat agentic misalignment across 16 frontier models; the "real vs test" gap (arXiv:2510.05179, Anthropic)
- [[magma-yang-2025]] — Magma: unified VLA foundation model for GUI + robotic manipulation; Set-of-Mark / Trace-of-Mark (arXiv:2502.13130, CVPR 2025)
- [[gaia-mialon-2023]] — GAIA general-AI-assistant benchmark; human 92% vs GPT-4+plugins 15% (arXiv:2311.12983)

**New concept pages (12)**: [[agentic-rl]], [[deepseek-r1]], [[search-r1]], [[darwin-godel-machine]], [[self-improving-agent]], [[ui-tars-2]], [[magma]], [[vision-language-action-model]], [[computer-use-agent]], [[swe-bench-live]], [[gaia]], [[agentic-context-engineering]].

**New entity stubs (11)**: orgs [[deepseek]], [[sakana-ai]], [[bytedance]], [[meta]], [[microsoft-research]], [[huggingface]], [[sambanova]]; products [[autogpt]]; people [[jeff-clune]], [[graham-neubig]]; universities [[cmu]].

**Key conceptual additions**:
1. **Agentic RL** as a 2025 paradigm distinct from RLHF (chat-pref training) and SFT-on-tool-traces — trains the agent end-to-end on multi-step traces with outcome rewards. DeepSeek-R1's open weights + GRPO recipe became the substrate; Search-R1 the canonical tool-use variant.
2. **Self-improving agents are now empirical**: Darwin Gödel Machine ships the Gödel-machine vision as a working algorithm (population archive, self-scaffold-rewrite, SWE-bench 20→50%). Closes a 20-year theoretical loop.
3. **Computer-use agents** consolidated as a category: UI-TARS-2 (specialist) and Magma (unified GUI+robotics VLA) bracket the design space.
4. **Benchmark contamination** is now first-class: SWE-bench Live shows 43% → 19% drop on identical config when tasks are post-training-cutoff. Plus GAIA backfilled as the foundational general-assistant eval cited by everything since.
5. **Agentic Misalignment** (Anthropic 2025) is the canonical empirical safety paper: 16 models, simulated corporate settings, blackmail at 96% under pressure, **55% real-mode vs 6.5% test-mode** blackmail rate — implying standard evals systematically underestimate deployment misalignment.

**Wiki size**: ~340 pages (was ~309). Source/concept/entity counts updated in index.

---

## [2026-05-17] query | Hooks and callbacks in modern agent architecture

Question: "what are some important hooks/callbacks exist in modern agent architecture". Filed as [[hooks-in-modern-agents]]. Cross-cuts the 5 agent doc sources to catalog ~30 named hooks organized by lifecycle phase (tool · session · message · LLM-call · stop · compaction · sub-agent · permission · environment). Surfaces two architectural axes: (1) first-class harness hook ([[claude-code]]) vs plugin-system hook ([[opencode]], [[openclaw]], [[hermes-agent]]) vs no-hooks-in-core ([[pi-mono]]); (2) observational → modifying → blocking → compositional behavior. Notes interesting omissions across the field (`on-error`, `pre-respond`, streaming-output hooks all absent or loosely covered). Pages consulted: [[claude-code-agent-doc-2026]], [[opencode-agent-doc-2026]], [[hermes-agent-doc-2026]], [[openclaw-agent-doc-2026]], [[pi-mono-agent-doc-2026]], [[cross-agent-comparison-2026]].

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
