---
title: "Code as Agent Harness: Toward Executable, Verifiable, and Stateful Agent Systems"
type: source
created: 2026-05-25
updated: 2026-05-25
sources:
  - code-as-agent-harness-ning-2026.md
arxiv_id: "2605.18747"
authors:
  - Xuying Ning
  - Katherine Tieu
  - Dongqi Fu
  - Tianxin Wei
  - Zihao Li
  - Jiaru Zou
  - Yuanchen Bei
  - Mengting Ai
  - Zhining Liu
  - Ting-Wei Li
  - Bingxuan Li
  - Cheng Qian
  - Gaotang Li
  - Xiao Lin
  - Zhichen Zeng
  - Yifan Sun
  - Xiyuan Yang
  - Ruida Wang
  - Rui Pan
  - Chenyuan Yang
  - Lingjie Chen
  - Yanjun Zhao
  - Ke Yang
  - Ruizhong Qiu
  - Sirui Chen
  - Dylan Zhang
  - Liri Fang
  - Zikun Cui
  - Yang Cao
  - Pan Chen
  - Dorothy Sun
  - Mahesh Srinivasan
  - Pan Lu
  - Nipun Mathur
  - Ren Chen
  - Hong Li
  - Hong Yan
  - Lingming Zhang
  - Yinglong Xia
  - Tong Zhang
  - Hanghang Tong
  - Jingrui He
first_author: Xuying Ning
corresponding_author: Jingrui He
year: 2026
introduces:
  - "[[code-as-harness]]"
  - "[[agentic-harness-engineering]]"
  - "[[plan-execute-verify-loop]]"
tags:
  - 2026
  - survey
status: complete
importance: high
---

# Code as Agent Harness: Toward Executable, Verifiable, and Stateful Agent Systems

**Source file**: [[code-as-agent-harness-ning-2026]]
**arXiv**: [arXiv:2605.18747](https://arxiv.org/abs/2605.18747) (v1, 18 May 2026)
**Authors**: Xuying Ning, Katherine Tieu, Dongqi Fu, Tianxin Wei, Zihao Li, Jiaru Zou, Yuanchen Bei (core contributors) + ~35 others; corresponding author Jingrui He
**Affiliations**: [[uiuc|University of Illinois Urbana-Champaign]] (primary), [[meta]], [[stanford-university]]
**Companion repository**: [Awesome-Code-as-Agent-Harness-Papers](https://github.com/YennNing/Awesome-Code-as-Agent-Harness-Papers)
**Length**: 66 pages, ~500+ citations
**Year**: 2026

## Summary

This survey argues for a unifying reframing of agentic AI: **code is not just an output that LLMs produce — it is the operational substrate of the agent harness itself**. The authors distinguish three coupled elements of long-running agentic systems: (1) **model-internal capabilities** (reasoning, perception, planning, simulation), (2) **system-provided harness infrastructure** (tools, APIs, sandboxes, memory, validators, permission boundaries, telemetry — the focus of harness engineering), and (3) **agent-initiated code artifacts** (regression tests, temporary tools, DSL programs, executable workflows, reusable skills, intermediate program states) — and locate the survey's novelty in the third, comparatively underexplored category. Programs are the medium through which agents *reason* (executing computation rather than narrating it), *act* (calling tools, driving robots, manipulating GUIs), and *model environments* (repositories, traces, simulators, tests as inspectable state).

The thesis is grounded in three properties that code uniquely possesses as a harness interface: **executability** (the harness can verify what the model intended by running it), **inspectability** (intermediate computation is exposed as structured traces the harness can read, store, and act upon), and **statefulness** (the evolving program represents task progress in persistent, modifiable form). This is contrasted with natural-language reasoning, where intermediate states are opaque, transient, and unverifiable. The survey takes a deliberately narrow definition: "code" means executable or machine-checkable artifacts — programs, scripts, formal specifications, proof scripts, API schemas, tool definitions, tests, repositories, simulators, configuration files, and logs/traces consumed by executable systems — not "code" as a loose metaphor for structured language.

The work strongly parallels and extends the [[agent-three-layer-model]] L2 (harness) abstraction. Where the three-layer model identifies the harness as the policy-governed software layer between L1 (model) and L3 (sub-agents/tools), this survey makes the **code-centric** claim that, across reasoning, acting, environment modeling, planning, memory, tool use, control, and multi-agent orchestration, the harness is *operationally* a programming-language and execution-environment problem. Programs are simultaneously the interface, the state, the action, and the verifier.

The survey is organized into three connected layers that follow "how code becomes an operational medium inside the agent loop": **(§2) Harness Interface** — how code enters the loop as reasoning substrate, action interface, and environment representation; **(§3) Harness Mechanisms** — planning, memory, tool use, the Plan–Execute–Verify (PEV) control loop, and Agentic Harness Engineering (AHE) for adaptive harness optimization; **(§4) Scaling the Harness** — multi-agent orchestration over shared code artifacts, including a new "Position" on building a shared code-centric harness substrate. §5 surveys five application domains (code assistants, GUI/OS agents, embodied agents, scientific discovery, personalization) and outlines seven open problems.

## Key Conceptual Claims

- **Three properties make code functional as a harness interface**: executable, inspectable, stateful. These are not properties of code as notation but as substrate — they convert model output into something the harness can verify, diagnose, and persist.
- **The bottleneck of autonomy is not the base model's reasoning ability, but the reliability of the surrounding harness** — the software layer of tools, sandboxes, memory, validators, permission boundaries, execution loops, and feedback channels that turn a stateless LLM into a long-running agent.
- **Agent-initiated code artifacts are the underexplored frontier**: regression tests, temporary tools, DSL programs, executable workflows, reusable skills, and intermediate program states that agents create, execute, observe, revise, persist, and share within the task execution loop.
- **The Plan-Execute-Verify (PEV) loop unifies planning, debugging, verification, and escalation** as a single cybernetic control process operating over executable program state. Plans become contracts, execution is sandboxed and permissioned, verification uses deterministic sensors plus HITL gates.
- **Agentic Harness Engineering (AHE) is a new discipline** that treats the harness infrastructure itself as the object of optimization (separate from prompt engineering or context engineering). An "Evolution Agent" uses deep telemetry to propose, evaluate, and promote governed mutations to harness components.
- **Multi-agent coordination should converge on a shared code-centric substrate** — repositories, tests, traces, blackboards, execution states — rather than file-only or message-passing protocols. The shared substrate enables inspectability, collective verification, and coordination across functional roles (planner, coder, reviewer, tester).
- **Code is the natural lingua franca across application domains**: in GUI/OS agents, code defines observations, actions, evaluators, and increasingly the world model itself; in scientific discovery, hypotheses are programs, experiments are protocols, labs are runtimes; in embodied agents, skills are reusable code modules with multimodal pre/postconditions.
- **Verification through executable feedback can create false confidence**: a green test is not the full specification. The survey calls for an "evidence bundle" model in which every accepted action declares what was checked, what was not, and what residual risk remains.

## Section Overviews

### §2 Harness Interface: Code for Reasoning, Acting, Environment Modeling

The interface layer asks the most fundamental question: *what medium connects the model to its task environment?* The argument is that code is the answer, organized around three roles.

- **§2.1 Code for Reasoning** (Program-Delegated, Formal Verification, Iterative Code-Grounded). Surveys the trajectory from [[program-of-thoughts]]/PAL [[chain-of-thought-prompting]] delegation through symbolic/formal verification (Lean, Coq, Isabelle, DeepSeek-Prover, Kimina-Prover, Goedel-Prover-V2, Lean4Agent) to iterative execution-grounded reasoning (NExT, CodePRM, CYCLE, RLEF, EG-CFG, R1-Code-Interpreter). The shift is from "code as external calculator" to "execution artifacts (traces, variable states, control flow, tests) as reusable reasoning signals."
- **§2.2 Code for Acting** (Grounded Skill Selection, Programmatic Policy Generation, Lifelong Code-Based Agents). Reviews SayCan and KnowNo (skill selection with affordance grounding and conformal uncertainty), Code-as-Policies and RoboCodeX (programmatic policy generation), Voyager and LYRA and ViReSkill (lifelong skill libraries that double as memory), and AutoHarness as the explicit harness-mediation pattern. The framing: code is "both the action to be executed and the executable boundary that connects model intent to perception, controllers, APIs, actuators, and safety constraints."
- **§2.3 Code for Environment** (Structured World Representations, Execution-Trace World Modeling, Code-Grounded Evaluation, Verifiable Environment Construction). Covers ViStruct, FactoredScenes, PoE-World, Code2World (structured world); SemCoder, [[code-world-model|CWM]], WorldCoder, RWML (trace-based dynamics); [[swe-bench]], [[swe-gym|SWE-Gym]], [[swe-bench-live]], InterCode, CRUXEval, LiveCodeBench, AgentBench (code-grounded evaluation); and SWE-smith + EnvScaler (programmatic environment construction). Environments themselves become harness artifacts to synthesize and validate.

### §3 Harness Mechanisms: Planning, Memory, Tool Use, Control, Optimization

This is the longest section and the operational heart of the survey. Five interacting mechanism categories.

- **§3.1 Planning** (Linear / Structure-grounded / Search-based / Orchestration-based). Planning is reframed as *harness control*, not a model-internal capability. Linear decomposition (Self-Planning, [[react|ReAct]], WebAgent, Plan-And-Act; and the modern "PLAN.md/Implement.md as filesystem-backed control object" pattern) gives way to structure-grounded planning over repo/circuit/knowledge graphs (CodePlan, GraphCodeAgent, VerilogCoder), search-based planning over thought/trajectory/code spaces (Tree-of-Code, ReThinkMCTS, SWE-Search, CodeTree, Meta-Harness), and orchestration-based planning ([[metagpt]], MapCoder, Blueprint2Code, plus Anthropic's long-running harnesses, Cursor's planner-worker designs, and Natural-Language Agent Harnesses with an "Intelligent Harness Runtime").
- **§3.2 Memory & Context Engineering** (Working / Semantic / Experiential / Long-Term / Multi-Agent / Compaction). Memory is "not a larger context window or vector database but a state-management layer." Six functional roles: working memory (SWE-agent, CodeMem, RepairAgent), semantic/repo memory (AutoCodeRover, RepoCoder, CodeRAG), experiential/episodic (MemGovern, ExpeL, [[a-mem]]), long-term ([[memgpt]], MemoryOS, MemCoder, TALM), multi-agent ([[mirix]], [[metagpt|ChatDev]], G-Memory), and cross-cutting context compaction/state offloading (LongCodeZip, SWE-Pruner, SWEZZE) — typically with MCP-style resource interfaces for durable offload. Strongly parallels the wiki's [[memory-management]] taxonomy.
- **§3.3 Tool Use** (Function-Oriented / Environment-Interaction / Verification-Driven / Workflow-Orchestration). Tools are "the action and observation layer." Four functional roles: function-oriented (ToolCoder, CodeAgent, RAG-for-code), environment-interaction (SWE-agent, RepairAgent — making the agent-computer interface explicit), verification-driven (AgentCoder, VeriGuard — tools as deterministic sensors), and workflow-orchestration (MapCoder, ToolNet, [[opencode]], [[claude-code]], OpenHands). Lifecycle hooks (pre-use validation, post-use sanitization) and [[mcp|MCP]]-style typed schemas govern tool use as a "monitored transition."
- **§3.4 Plan-Execute-Verify Loop** (Debugging as harness control, Planning as Contract Formation, Sandboxed Execution + Permissioned State Transition, Verification through Deterministic Sensors). Recasts iterative debugging (Self-Debugging, [[reflexion]], LDB, Iterative Refinement, QualityFlow) as a cybernetic governor over executable program state. Plans declare invariants and rollback points; execution happens in tiered permission sandboxes (read-only / sandbox-edit / full-access, with HITL gates for the last tier and audit logs throughout); verification composes linters, type checkers, tests, fuzzers, static analyzers, CI, and runtime monitors. AGENTS.md, MCP server registries, gateway/policy layers (LiteLLM) are cited as contract layers.
- **§3.5 Adaptive Harness Optimization** (Deep Telemetry, Evolution Agent, Governed Mutation). Introduces **Agentic Harness Engineering (AHE)** as a distinct discipline from prompt or context engineering: the operating environment itself (tool schemas, planning artifacts, memory policies, sandbox configuration, verification sensors, permission tiers, routing rules, workflows) is the object of analysis. An *Evolution Agent* observes telemetry, diagnoses failure modes, proposes mutations, evaluates against held-out tasks, and only promotes verified-non-regressing changes. Cites AutoHarness, Meta-Harness, AHE, GEPA, EvoMAC, SEW, Live-SWE-Agent. Observability stacks (Langfuse, OpenLLMetry, Promptfoo) supply the telemetry substrate.

### §4 Scaling the Harness: Multi-Agent Orchestration over Code

Motivated by three single-agent limitations: context window constraints, specialization requirements, and the absence of independent verification channels.

- **§4.1 Functional Roles + Interaction + Topology**. Surveys role-specialized MAS ([[metagpt|ChatDev/MetaGPT]], AgentCoder, MAGIS, HyperAgent, MAGE, Self-Collaboration) across program synthesis, program understanding, planner/manager, reviewer, tester, debugger, security agent roles; interaction modes (collaborative-synthesis, critique-repair, debate, delegation, search/selection, adversarial/red-teaming); and topologies (chain, cyclic, hierarchical, star, blackboard, plus dynamic/object-driven adaptive topologies).
- **§4.2 Execution Feedback + Shared-Harness Synchronization**. Compile-and-test, runtime-and-profiling, simulation/fuzzing as collective sensors. Synchronization protocols across file-only, repository-based, execution-based, blackboard substrates.
- **§4.3 Position — The Shared Code-Centric Harness Substrate**. The survey's normative claim: the code substrate (repositories, tests, traces, execution states) should be elevated to a first-class shared object, with explicit harness-state convergence around correctness, security, performance, and consensus.
- **§4.4 Patterns and Trends**: convergence toward adaptive topologies, learned coordination, and shared blackboards over rigid pipelines.

### §5 Emerging Fields & Open Problems

§5.1 surveys five application domains as instantiations of the code-as-harness thesis:
- **Code Assistants**: [[claude-code]], Codex, OpenHands, SWE-agent, and the "AGENTS.md / PLAN.md as filesystem harness state" pattern; SDK-level packaging of tools, sessions, guardrails (Anthropic Agent SDK, [[opencode]], [[hermes-agent|Hermes]]).
- **GUI/OS Agents as a Program World**: Formalized as a POMDP whose transition function is *executed* (browser engine, Android runtime, OS) rather than learned. Code defines observations (DOM, AXTree, screenshots+SoM), actions (CodeAct-style Python emitting click/type primitives), evaluators (per-task Python checkers), and memory (skill libraries, knowledge-centric memory graphs like PlugMem). Cites WebArena, OSWorld, AndroidWorld, [[osworld]], WindowsAgentArena, Mind2Web, [[ui-tars-2]], CogAgent, AutoGLM, Cradle, plus production deployments ([[computer-use-agent|Claude Computer Use]], OpenAI Operator/CUA, Project Mariner). Code2World makes the program-world stance explicit at the model level.
- **Autonomous Embodied Agents**: Code as control boundary connecting agents to the world. Layered harness (semantic reasoning → admissibility boundary via typed robot APIs → perception/state estimation → low-level controllers). Skills as embodied memory ([[voyager]], LYRA, ViReSkill, SkillsCrafter); coordinated/auditable deployment (GenSwarm, NormCode, RACAS).
- **Agents for Scientific Discovery as Program Worlds**: Hypotheses as differential equations/generative models; protocols as XDL/Opentrons scripts; instruments as Python APIs; analyses as Jupyter notebooks. AI Scientist v1/v2, AI co-scientist, Virtual Lab, Biomni, ChemCrow, Coscientist; the self-driving lab (Berkeley A-Lab, Chemputer/XDL, MatPilot) as the production system. AlphaProof and AlphaEvolve as the goal-conditioned extreme. Benchmarks: MLAgentBench, MLE-bench, ScienceAgentBench, DiscoveryBench.
- **Agent Personalization**: Preference state as editable artifact; feedback pipelines as part of the harness; instruction-following personalization; structured (vs. embedding-only) preference memory.

### Open Problems (§5.2)

The seven open problems the survey identifies for harness engineering:

1. **Harness-Level Evaluation and Oracle Adequacy** — end-task success conflates model, harness, tools, feedback, and environment. Need metrics for trajectory efficiency, verification strength, recovery, state consistency, safety compliance, and replayability.
2. **Semantic Verification Beyond Executable Feedback** — green tests are not full specifications. Move from binary pass/fail to *evidence bundles* (what was checked, what was not, what residual risk). Composable verifier stacks (tests, static analysis, fuzzers, property-based testing, formal specs, human review) each declaring scope and confidence.
3. **Self-Evolving Harnesses without Regression** — automated harness evolution exists (AutoHarness, Meta-Harness, AHE, GEPA, Live-SWE-Agent); the hard problem is improving without overfitting, hiding failures, weakening safety, or regressing on rare tasks. Every harness mutation should carry a *change contract*.
4. **Transactional Shared Program State and Semantic Conflict Resolution** — multi-agent systems sync artifacts but not assumptions. Need transactional semantics: read/write sets, version dependencies, semantic-merge, rollback, dependency-aware locking, belief-state reconciliation. Analogies to version control, databases, CRDTs, but with semantic conflicts they don't see.
5. **Human-in-the-Loop Safety and Accountability as Harness State** — HITL is not an occasional interruption but durable harness state. Multi-tier permissions (read-only / sandbox-edit / full-access); context-sensitive policies; *executable accountability* with auditable approval records.
6. **Multimodal Code-Harness Systems** — most harnesses are textual; GUI, embodied, and scientific environments are multimodal. Need multi-level memory (raw frames + structured annotations + symbolic summaries), grounding contracts coupling perception/action/verification, calibrated multimodal verification, and skills that bundle multimodal preconditions with executable actions and expected postconditions.
7. **Toward a Science of Harness Engineering** — the central object of study is no longer the model or the program but the complete closed-loop system. Calls for benchmarks exposing long-horizon failures, telemetry making trajectories auditable, metrics that isolate harness components, and design principles for persistent program worlds.

## Notable Quotes

> "Code is not only an artifact generated by LLMs, but also an executable, inspectable, and stateful medium through which agents reason, act, observe feedback, and verify progress. We refer to this view as code as agent harness."

> "The bottleneck of autonomy is not only the reasoning ability of the base model, but also the reliability of the system that connects model outputs to long-horizon actions and persistent states."

> "We use code broadly, but not metaphorically. ... raw perception, physical state, human intent, and model-internal latent reasoning are not themselves code. They may be sensed, estimated, serialized, verified, or acted upon through code, but they should not be conflated with the code interface."

> "Recasting iterative debugging as the PEV loop emphasizes that reliability comes from governed state transitions, not simply from better repair prompts."

> "A harness can become overconfident precisely because it has executable feedback: the agent sees a green test, but the green test is not the full specification."

## Entities Mentioned

- **Institutions** — [[uiuc|University of Illinois Urbana-Champaign]] (primary affiliation; corresponding author Jingrui He), [[meta]], [[stanford-university]]
- **Products / Coding agents** — [[claude-code]], [[opencode]], [[hermes-agent]], OpenHands, Codex, Cursor, [[anthropic]] Agent SDK, LangChain
- **Computer-use products** — [[computer-use-agent|Claude Computer Use]], OpenAI Operator/CUA, Project Mariner, [[ui-tars-2|UI-TARS]], AutoGLM, AppAgent
- **Companies** — [[anthropic]], [[openai]], [[microsoft-research]], [[deepmind]], [[bytedance]], [[deepseek]] (DeepSeek-Prover lineage), [[sakana-ai]] (AI Scientist authors)
- **Scientific discovery systems** — AI Scientist v1/v2, AI co-scientist (Google), Virtual Lab, Biomni, ChemCrow, Coscientist, AlphaProof, AlphaEvolve

## Concepts Discussed

The survey is a synthesis layer over a vast portion of the wiki. Most relevant cross-links:

- **New concepts the survey introduces or names** — `code-as-harness`, `agentic-harness-engineering`, `plan-execute-verify-loop`, `evolution-agent`, `shared-code-centric-harness-substrate`, `executable-accountability`
- **Wiki concepts the survey directly maps onto** — [[agent-three-layer-model]] (the harness layer made code-explicit), [[react]], [[chain-of-thought]], [[function-calling]], [[tool-use]], [[memory-management]], [[multi-agent-memory]], [[mcp]], [[acp]], [[skill-library]], [[skill-md-format]], [[gui-agent]], [[computer-use-agent]], [[vision-language-action-model]], [[embodied-agent]], [[self-improving-agent]], [[darwin-godel-machine]], [[godel-machine]], [[reflexion]], [[self-reflection]], [[tree-of-thoughts]], [[swe-bench]], [[swe-gym]], [[swe-bench-live]], [[osworld]], [[gaia]], [[react]], [[voyager]], [[ui-tars-2]], [[metagpt]], [[autogen]], [[mirix]], [[memgpt]], [[a-mem]], [[mem0]], [[generative-agents]], [[agentic-context-engineering]], [[agentic-rl]], [[search-r1]], [[deepseek-r1]], [[magma]], [[red-teaming]], [[reward-hacking]], [[guardrailing]], [[hallucination]]

## References to Existing Wiki Sources

The survey directly cites or builds on work the wiki has already ingested:

- [[react-yao-2022]] — ReAct as the canonical interleaved-reasoning baseline for harness control loops
- [[reflexion-shinn-2023]] — verbal-feedback reflection in the PEV verify phase
- [[voyager-wang-2023]] — the lifelong skill library archetype for code-as-acting
- [[swe-bench-jimenez-2023]], [[swe-gym-pan-2024]], [[swe-bench-live-zhang-2025]] — repository-level code-grounded evaluation
- [[metagpt-hong-2023]] — role-specialized MAS for code generation (§4)
- [[autogen-wu-2023]] — multi-agent orchestration framework
- [[chain-of-thought-wei-2022]], [[zero-shot-cot-kojima-2022]] — natural-language reasoning baseline against which program-delegated reasoning is contrasted
- [[memgpt-packer-2023]], [[mirix-wang-2025]], [[mem-alpha-wang-2025]], [[mem0-chhikara-2025]], [[a-mem-xu-2025]], [[memorybank-zhong-2023]] — memory-management lineage that §3.2 builds on
- [[model-context-protocol-anthropic-2024]] — MCP as the typed-tool-schema / state-offload protocol cited throughout §3.3 and §3.4
- [[claude-code-agent-doc-2026]], [[opencode-agent-doc-2026]], [[hermes-agent-doc-2026]], [[pi-mono-agent-doc-2026]], [[openclaw-agent-doc-2026]] — production harness instances exemplifying §3 mechanisms
- [[darwin-godel-machine-zhang-2025]] — self-improving agent that the survey's "Evolution Agent" §3.5.2 echoes
- [[deepseek-r1-2025]], [[search-r1-jin-2025]] — RL-based reasoning agents
- [[ui-tars-2-wang-2025]], [[magma-yang-2025]] — VLA/GUI agent lineage feeding §5.1.2
- [[gaia-mialon-2023]] — general AI assistant benchmark referenced in agent evaluation discussion
- [[generative-agents-park-2023]] — memory-stream antecedent for §3.2

## Significance for the Wiki

This is a **definitive synthesis** of the agent harness literature through May 2026 and arguably the most important survey to ingest for any reader navigating the gap between L1 (model) and L3 (sub-agents/tools) in the [[agent-three-layer-model]]. It explicitly names and operationalizes concepts the wiki has been tracking piecemeal: the harness as a distinct engineering object, the role of code as its medium, the PEV loop as the unifying control abstraction, and AHE as a discipline parallel to (but distinct from) prompt and context engineering. Future ingests of harness-related papers, production agent documentation, and multi-agent system designs should cite this survey as the canonical taxonomy.
