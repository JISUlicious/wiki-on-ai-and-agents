---
title: Wiki Index
type: index
created: 2026-05-12
updated: 2026-05-17
---

# Wiki Index

A catalog of all pages in this wiki, organized by category. The wiki currently covers 231 sources, 133 entities, 201 concepts, 2 comparisons, 7 queries.

## Entities

### Models — Foundational architectures
- [[transformer]] — Original encoder-decoder Transformer (Vaswani 2017)
- [[transformer-decoder]] — Decoder-only Transformer for Wikipedia (Liu 2018), architectural ancestor of GPT
- [[bert]] — Bidirectional encoder, MLM pre-training (Devlin 2018)
- [[t5]] — Text-to-text encoder-decoder Transformer; introduced the C4 corpus (Raffel 2019)

### Models — GPT family
- [[gpt-1]] — First GPT, 117M params (Radford 2018)
- [[gpt-2]] — 1.5B params, zero-shot transfer thesis (Radford 2019)
- [[gpt-3]] — 175B params, in-context learning emerges (Brown 2020)
- [[gpt-4]] — Multimodal, RLHF-aligned, undisclosed architecture (OpenAI 2023)
- [[instructgpt]] — GPT-3 + SFT/RLHF; direct predecessor of ChatGPT (Ouyang 2022)
- [[webgpt]] — GPT-3 fine-tuned to operate a web browser (Nakano 2021)

### Models — Other LLMs
- [[claude]] — Anthropic's model family
- [[claude-opus-4-8]] — Anthropic's mid-2026 flagship Opus; enables dynamic workflows in Claude Code
- [[glm-5-2]] — Z.ai's coding-first 1M-context MoE (744B/40B-active) with IndexShare sparse attention (2026)
- [[mai-thinking-1]] — Microsoft AI's 1T/35B-active MoE "hill-climbing machine"; trained only on human data (2026)
- [[palm]] — Google's 540B model (Chowdhery 2022)
- [[chinchilla]] — DeepMind's 70B compute-optimal model (Hoffmann 2022)
- [[flan]] — 137B instruction-tuned LM and family (Wei 2021)
- [[bart]] — Denoising seq2seq Transformer (Lewis 2019)
- [[vit]] — Vision Transformer (Dosovitskiy 2020)
- [[dpr]] — Dense Passage Retriever (Karpukhin 2020)
- [[rag]] — Original RAG system (Lewis 2020)
- [[retro]] — Retrieval-Enhanced Transformer at trillion-token scale (Borgeaud 2021)

### Agent products (2026 era)
- [[claude-code]] — Anthropic's terminal coding agent
- [[opencode]] — sst/opencode, server-architected open-source coding agent
- [[pi-mono]] — badlogic's minimalist coding agent (7 tools)
- [[hermes-agent]] — Nous Research's self-improving multi-surface agent
- [[openclaw]] — Single-operator multi-channel personal AI assistant
- [[oh-my-claude]] — Third-party Claude Code plugin (TechDufus); reference implementation of the ultrawork keyword
- [[faiss]] — Facebook AI Similarity Search library
- [[openai-api]] — OpenAI's API (Chat Completions / Responses / Assistants) — function-calling reference implementation
- [[vllm]] — High-throughput LLM inference engine; ~25 tool-call parsers
- [[autogpt]] — Early popular open-source autonomous agent (2023); GAIA co-author

### People — Foundational researchers (pre-Transformer)
- [[sepp-hochreiter]] — LSTM (1997); JKU Linz
- [[jurgen-schmidhuber]] — LSTM co-author; IDSIA / KAUST
- [[tomas-mikolov]] — word2vec (2013); Google → FAIR → CIIRC
- [[jeff-dean]] — Google Senior Fellow; co-founder of Google Brain
- [[jeffrey-pennington]] — GloVe (2014); Stanford
- [[richard-socher]] — GloVe co-author; founded MetaMind, you.com
- [[christopher-manning]] — Stanford NLP director; GloVe co-author
- [[ilya-sutskever]] — seq2seq (2014), GPT-1; OpenAI co-founder
- [[oriol-vinyals]] — seq2seq, Chinchilla, AlphaStar; DeepMind
- [[quoc-le]] — seq2seq, FLAN, AutoML; Google Brain
- [[dzmitry-bahdanau]] — Soft attention (2014)
- [[kyunghyun-cho]] — NMT pioneer; NYU
- [[yoshua-bengio]] — Turing Award winner; Mila Montreal

### People — Transformer / pre-training era
- [[ashish-vaswani]] — Transformer first author
- [[noam-shazeer]] — Transformer co-author; scaled dot-product attention
- [[jacob-devlin]] — BERT first author; Google AI Language
- [[alec-radford]] — GPT-1/2 lead author; OpenAI
- [[colin-raffel]] — T5 co-first author

### People — Scaling and emergence
- [[tom-brown]] — GPT-3 first author; Anthropic co-founder
- [[jared-kaplan]] — Scaling laws first author; Anthropic co-founder
- [[dario-amodei]] — Anthropic CEO; was OpenAI VP of Research
- [[sam-mccandlish]] — Scaling laws co-lead; Anthropic co-founder
- [[jordan-hoffmann]] — Chinchilla first author; DeepMind
- [[chris-olah]] — Mechanistic interpretability founder; Anthropic co-founder

### People — Reasoning / agents
- [[jason-wei]] — Chain-of-thought; FLAN; OpenAI (was Google Brain)
- [[shunyu-yao]] — ReAct, Tree-of-Thoughts; Princeton PhD
- [[paul-christiano]] — Foundational RLHF paper (2017); Alignment Research Center
- [[jan-leike]] — RLHF co-author; OpenAI Superalignment (now Anthropic)
- [[rafael-rafailov]] — DPO; Stanford
- [[chelsea-finn]] — DPO advisor; Stanford; Physical Intelligence co-founder
- [[luke-zettlemoyer]] — Toolformer senior author; UW / Meta AI

### People — Retrieval / RAG
- [[patrick-lewis]] — RAG first author; DPR co-author
- [[danqi-chen]] — DPR co-author; Princeton NLP

### People — Steering / interpretability
- [[alexander-matt-turner]] — ActAdd first author; activation engineering
- [[andy-zou]] — Representation Engineering (RepE) first author
- [[dan-hendrycks]] — Center for AI Safety director; RepE co-author
- [[fernanda-viegas]] — Visualization researcher; Google PAIR
- [[martin-wattenberg]] — Visualization researcher; Google PAIR / Harvard

### People — Agent memory systems
- [[charles-packer]] — MemGPT / Sleep-time Compute first author; Letta co-founder
- [[ion-stoica]] — UC Berkeley Sky Computing director; Letta / Anyscale / Databricks co-founder
- [[joon-sung-park]] — Generative Agents first author (Stanford 2023)
- [[anima-anandkumar]] — Voyager senior author; Caltech / formerly NVIDIA
- [[percy-liang]] — Stanford NLP / CRFM; Generative Agents co-author
- [[bernal-jimenez-gutierrez]] — HippoRAG / HippoRAG 2 first author; OSU NLP

### People — Agentic capabilities (2025)
- [[jeff-clune]] — UBC / Vector / Sakana AI; senior author Darwin Gödel Machine; open-ended evolution
- [[graham-neubig]] — CMU professor; All Hands AI / OpenHands co-founder; SWE-Gym senior author

### People — Looped Transformers
- [[kangwook-lee]] — UW-Madison; senior author on multiple LT papers (Giannou 2023, Yang 2023, Fan 2024)
- [[dimitris-papailiopoulos]] — UW-Madison; senior co-author on LT papers (Giannou 2023, Yang 2023)
- [[nikunj-saunshi]] — Google Research; "latent thoughts" framing (Saunshi 2025, Gatmiry 2024)

### People — Other
- [[blaise-aguera-y-arcas]] — Google research VP; Paradigms of Intelligence team
- [[james-evans]] — Computational social scientist; UChicago + Santa Fe Institute
- [[mario-zechner]] — pi-mono creator; "badlogic"
- [[thariq-shihipar]] — Anthropic MTS; co-author of the dynamic-workflows engineering post
- [[sid-bidasaria]] — Anthropic MTS; co-author of the dynamic-workflows engineering post

### Organizations
- [[anthropic]] — Founded 2021 by Amodei et al.; Claude family
- [[openai]] — Source of GPT family, ChatGPT, the OpenAI API
- [[deepmind]] — UK AI lab; merged with Google Brain in 2023
- [[google-brain]] — Google's deep-learning research org until merger
- [[google-research]] — Google's broader research org
- [[fair]] — Facebook AI Research / Meta AI
- [[nous-research]] — Decentralized AI research collective; Hermes models / agent
- [[z-ai]] — Zhipu AI; maker of the GLM model family (GLM-5.2)
- [[nvidia]] — GPU/accelerated-computing company; Nemotron, Cosmos, Megatron
- [[ant-group]] — Chinese fintech/tech company; AI research (OPRD with Zhejiang U.)
- [[center-for-ai-safety]] — CAIS, AI safety org (Hendrycks); RepE co-authorship
- [[eleutherai]] — Open-source AI research collective (Pythia, etc.)
- [[letta]] — Berkeley spinout productizing MemGPT; agent runtime with stateful memory
- [[zep]] — Long-term memory infrastructure for LLM agents (Zep Cloud, Graphiti)
- [[zed]] — Zed Industries; Rust collaborative editor; creator of the [[acp|Agent Client Protocol]]
- [[deepseek]] — Open-weights frontier-model lab (DeepSeek-V3, DeepSeek-R1, GRPO)
- [[sakana-ai]] — Tokyo AI lab focused on evolutionary / open-ended ML (Darwin Gödel Machine)
- [[bytedance]] — Hosts ByteDance Seed research; produced UI-TARS-1 / UI-TARS-2
- [[meta]] — Meta Platforms (distinct from [[fair]] research arm)
- [[microsoft-research]] — Microsoft's research arm (SWE-bench Live, Magma)
- [[huggingface]] — Open-source ML model hub; co-authored GAIA
- [[sambanova]] — AI accelerator company; co-authored Agentic Context Engineering
- [[pwc]] — PricewaterhouseCoopers; applied-AI research on long-horizon agent memory (Chronos, agentic-search)
- [[openmythos]] — Open-source reference implementation of recurrent-depth / looped transformers (kyegomez)
- [[microsoft]] — Tech giant; parent of [[microsoft-research]]
- [[bytedance-seed]] — ByteDance's foundation-model research division (UI-TARS-1/2)

### Universities
- [[university-of-toronto]] — Hinton's institutional home
- [[stanford-university]] — Stanford NLP; many papers in this wiki
- [[princeton-university]] — Princeton NLP
- [[university-of-washington]] — UW NLP
- [[uc-berkeley]] — EECS / Sky Computing Lab; origin of MemGPT and Sleep-time Compute
- [[ohio-state-university]] — OSU NLP under Yu Su; HippoRAG / HippoRAG 2
- [[caltech]] — Computing & Math Sciences; Voyager co-author institution (Anandkumar)
- [[cmu]] — Carnegie Mellon; Graham Neubig's home; SWE-Gym co-author institution
- [[mit]] — Massachusetts Institute of Technology; LILO / DreamCoder lineage (Tenenbaum, Andreas)
- [[uiuc]] — UI Urbana-Champaign; Heng Ji's group (CodeAct, OpenHands, CRAFT)
- [[zhejiang-university]] — Home of the agent-skills survey (Xu & Yan 2026)
- [[tsinghua-university]] — ExpeL (Gao Huang's group); ChatDev
- [[shanghai-jiao-tong-university]] — SJTU; co-corresponding institution on SkillOpt (skill optimization)
- [[uc-santa-barbara]] — UCSB; led the realistic agent-skills benchmark (with MIT)

### People — Agent skills / tool creation
- [[josh-tenenbaum]] — MIT; computational cognitive science, program synthesis (DreamCoder/LILO)
- [[jacob-andreas]] — MIT CSAIL; language + program synthesis (LILO)
- [[heng-ji]] — UIUC; leads the CodeAct / OpenHands / CRAFT group
- [[xingyao-wang]] — UIUC; first author CodeAct + OpenHands
- [[hao-peng]] — UIUC; CodeAct / CRAFT co-author
- [[gao-huang]] — Tsinghua; corresponding author of ExpeL
- [[tianle-cai]] — LATM first author; Princeton / Google DeepMind

## Concepts

### Core architectures and components
- [[transformer-architecture]] — The architectural pattern: attention-only sequence model
- [[decoder-only-transformer]] — The dominant variant for modern LLMs
- [[mixture-of-experts]] — Sparse expert routing; decouples total capacity from per-token compute (active vs total params)
- [[world-model]] — Learned model of environment dynamics; predict latent states, not just tokens (Cosmos 3, Mirage, JEPA)
- [[encoder-decoder]] — Two-stack architecture for seq2seq tasks
- [[attention-mechanism]] — Query/key/value weighted aggregation
- [[self-attention]] — Intra-sequence attention; defining op of Transformers
- [[multi-head-attention]] — Parallel attention in projected subspaces
- [[scaled-dot-product-attention]] — The specific attention function in the Transformer
- [[additive-attention]] — The original Bahdanau attention (2014); pre-Transformer
- [[positional-encoding]] — Injecting order without recurrence
- [[residual-connection]] — Skip connections for deep network training
- [[layer-normalization]] — Standard normalization in Transformers
- [[lstm]] — Long short-term memory; pre-Transformer dominant
- [[recurrent-neural-network]] — RNN family
- [[vanishing-gradient-problem]] — The training pathology LSTMs/Transformers fix
- [[seq2seq]] — Sequence-to-sequence framework (Sutskever 2014)

### Tokenization and embeddings
- [[byte-pair-encoding]] — BPE subword tokenization
- [[wordpiece]] — BERT's tokenization (likelihood-based BPE cousin)
- [[patch-tokenization]] — Treating image patches as tokens (ViT)
- [[word-embeddings]] — Dense vector representations of words
- [[word2vec]] — Mikolov's Skip-gram/CBOW embeddings
- [[glove]] — Stanford's global-co-occurrence embedding alternative
- [[distributional-semantics]] — "Know a word by the company it keeps"

### Training paradigms
- [[pre-training]] — Self-supervised training on large corpora
- [[fine-tuning]] — Task-specific adaptation
- [[autoregressive-language-modeling]] — Next-token prediction
- [[masked-language-modeling]] — BERT's bidirectional cloze objective
- [[next-sentence-prediction]] — BERT's auxiliary sentence-pair objective
- [[instruction-tuning]] — Fine-tuning on (instruction, response) pairs
- [[rlhf]] — Reinforcement learning from human feedback
- [[rlaif]] — RL from AI feedback (Constitutional AI)
- [[dpo]] — Direct Preference Optimization (simplifies RLHF)
- [[ppo]] — Proximal Policy Optimization (Schulman 2017); workhorse for LLM RL
- [[grpo]] — Group Relative Policy Optimization (DeepSeek); group-normalized advantages
- [[preference-modeling]] — Bradley-Terry preference learning
- [[reward-modeling]] — Learning reward signals from preferences
- [[constitutional-ai]] — Anthropic's harmlessness training method
- [[negative-sampling]] — Contrastive learning objective from word2vec

### Capabilities and prompting
- [[in-context-learning]] — Learning from prompt examples (GPT-3)
- [[zero-shot-learning]] — Task from instruction only
- [[few-shot-learning]] — Task from instruction + a handful of examples
- [[chain-of-thought-prompting]] — Prompting with intermediate reasoning steps
- [[zero-shot-cot]] — "Let's think step by step" zero-shot
- [[self-consistency]] — Sample-and-marginalize over reasoning paths
- [[tree-of-thoughts]] — Explicit search over reasoning paths
- [[emergent-abilities]] — Capabilities that appear with scale
- [[scaling-laws]] — Power-law relationships between loss and resources

### Agents, tools, and orchestration
- [[llm-agent]] — LLM in a perception-action loop with tools
- [[agent-three-layer-model]] — Three-layer model (L1 LLM capabilities / L2 agent architecture / L3 implementation)
- [[mcp]] — Model Context Protocol (Anthropic 2024): agent ↔ tool/data standard
- [[acp]] — Agent Client Protocol (Zed 2025): agent ↔ editor standard
- [[agent-skills]] — Anthropic Agent Skills: composable, progressively-disclosed capability modules
- [[skill-acquisition]] — How agents acquire skills: RL-trained / autonomous discovery / compositional synthesis / human-authored
- [[skill-optimization]] — 2026 thread: automatically improving a skill artifact (text-space edits / RL); SkillOpt, SkillMOO, SkillReducer, Skill1, SAGE
- [[skill-evaluation]] — Measuring skill usefulness in realistic settings (retrieve from ~34k real skills)
- [[tool-creation]] — Agents authoring + caching their own reusable tools (LATM, CRAFT)
- [[library-learning]] — Program-synthesis lineage: growing reusable code abstractions (DreamCoder → LILO)
- [[experiential-learning]] — Distilling reusable NL insights from trajectories (ExpeL); the non-code skill thread
- [[dreamcoder]] — Wake-sleep Bayesian library learning (Ellis 2020); pre-LLM ancestor
- [[context-assembly]] — L2 abstraction: how each turn's prompt is constructed
- [[function-calling]] — The mechanism: LLM emits JSON tool-call request; app executes
- [[responses-api]] — OpenAI Responses API (March 2025): unified Chat Completions + Assistants
- [[tool-call-parser]] — Converts model output (varying syntax per family) to structured tool calls
- [[structured-outputs]] — Forcing model output to a JSON schema via constrained decoding
- [[constrained-decoding]] — Logit-mask grammar enforcement (xgrammar, outlines, lm-format-enforcer)
- [[json-schema]] — IETF spec for describing JSON data; the format every tool definition uses
- [[json-rpc]] — Stateless RPC protocol over JSON; transport substrate for MCP and ACP

### Agentic capabilities (2025)
- [[agentic-rl]] — Training agents end-to-end with RL on multi-step tool-use / reasoning traces
- [[deepseek-r1]] — Pure-RL reasoning model + multi-stage variant; open-weights inflection (DeepSeek 2025)
- [[search-r1]] — Canonical tool-use RL recipe; retrieved-token masking + outcome rewards
- [[swe-gym]] — Training environment for SWE agents (2,438 Python tasks with unit tests)
- [[swe-bench-live]] — Monthly-refreshed contamination-resistant SWE-bench (Microsoft 2025)
- [[darwin-godel-machine]] — Empirical self-improving agent; rewrites its own scaffold (Sakana / UBC 2025)
- [[self-improving-agent]] — Umbrella: agents that modify their own skills / prompts / scaffold / weights
- [[computer-use-agent]] — VLM/VLA agents that drive a screen (UI-TARS-2, Magma, Anthropic CU)
- [[ui-tars-2]] — ByteDance Seed's multi-turn-RL GUI agent (OSWorld SOTA 47.5)
- [[magma]] — Microsoft's unified VLA foundation model for GUI + robotic manipulation
- [[vision-language-action-model]] — Foundation models that emit actions conditioned on vision + language
- [[agentic-context-engineering]] — Generator/Reflector/Curator triad evolves the prompt as a delta-updated playbook
- [[gaia]] — Meta+HF+AutoGPT general-AI-assistant benchmark (2023); cited by every 2024+ agent paper
- [[agentic-search]] — In-loop RAG: retriever × harness × tool-delivery as a single jointly-evaluated system
- [[longmemeval]] — ICLR 2025 benchmark for long-term interactive memory in chat assistants
- [[chronos]] — PwC's temporal-event memory agent (Sen 2026); LongMemEval-S 95.6%
- [[swe-bench]] — Jimenez 2023 foundational coding-agent benchmark (2,294 issue→PR pairs)
- [[swe-gym]] — Pan 2024 SWE-agent training environment (2,438 Python tasks with unit tests)
- [[osworld]] — Computer-use agent benchmark; OSWorld
- [[gui-agent]] — VLM agents driving a screen; subset of [[computer-use-agent]]
- [[embodied-agent]] — Robot/physical-world counterpart to GUI agents
- [[set-of-mark]] — Magma's clickable element grounding via marked overlays
- [[trace-of-mark]] — Magma's motion-planning via marked trajectories
- [[ui-tars-1]] — ByteDance Seed's first-gen native GUI agent (Qin 2025)
- [[agent-benchmark]] — Umbrella concept for agent evaluation benchmarks
- [[inference-time-compute]] — Allocating more compute at inference (CoT, looping, search)
- [[speculative-decoding]] — Lossless inference acceleration: fast draft model + parallel target verification
- [[reinforcement-learning]] — Umbrella RL concept
- [[chain-of-thought]] — Soft alias for [[chain-of-thought-prompting]]
- [[online-learning]] — Incremental training as new data arrives
- [[tool-use]] — LLMs invoking external tools / functions

### Code as Harness (executable agents)
- [[code-as-harness]] — Code as the operational substrate of the agent harness (Ning 2026 survey)
- [[codeact]] — Executable Python as agent action space (Wang 2024)
- [[pal]] / [[program-aided-reasoning]] — Code-for-reasoning origin (Gao 2023)
- [[code-as-policies]] — Code-for-acting in embodied settings (Liang 2022)
- [[swe-agent]] — Agent-Computer Interface for SE (Yang 2024)
- [[agent-computer-interface]] — ACI abstraction: purpose-built tool surface for LLM agents
- [[openhands]] — Open-source generalist software-agent platform (formerly OpenDevin)
- [[agentcoder]] — Programmer + Test-Designer + Test-Executor triad (Huang 2023)
- [[chatdev]] — Waterfall-style multi-agent SE pipeline (Qian 2023)
- [[agentic-harness-engineering]] — The harness as an object of optimization: telemetry-driven evolution, program search, self-synthesis, scaling laws (2026 concept; 8-source cluster)
- [[dynamic-workflows]] — Claude Code writes its own multi-agent harness as a JS orchestration script per task (Anthropic 2026)
- [[reasoning-effort]] — the `effort` parameter (low/medium/high/xhigh/max); behavioral signal over all token spend
- [[ultracode]] — official Claude Code `/effort` mode: `xhigh` + standing permission to auto-launch workflows
- [[ultrawork]] — unofficial community trend/keyword (e.g. oh-my-claude) for max parallel autonomous execution

### Looped Transformers (depth-recursion)
- [[looped-transformer]] — Umbrella: transformer whose forward pass is re-applied over the same layers; k×L loops ≈ kL depth at ~1/L parameters
- [[universal-transformer]] — Dehghani 2018: recurrent-in-depth transformer with Adaptive Computation Time; spiritual ancestor
- [[recurrent-depth-transformer]] — Modern 2026 rebranding for pretraining-scale looped LMs (OpenMythos / Parcae)
- [[latent-reasoning]] — Silent CoT via loop iteration; "latent thoughts" — depth-recursion counterpart to token-recursion
- [[adaptive-computation-time]] — Graves 2016 mechanism for input-adaptive depth; foundation for the entire LT thread
- [[timestep-encoding]] — Xu & Sato 2024's fix for weight-tying expressivity gap
- [[parameter-sharing]] — Reusing the same weights across iterations/layers
- [[length-generalization]] — Generalizing to longer inputs than training; LT is one approach
- [[grokking]] — Sudden generalization after extended training; observed in RDTs
- [[rasp-l]] — Length-restricted RASP complexity class (Fan 2024)
- [[parcae]] — LTI-stable looped LM training recipe (Prairie 2026)
- [[godel-machine]] — Schmidhuber's 2003 theoretical ancestor of self-improving agents
- [[adas]] — Hu, Lu, Clune 2024 — automated design of agentic systems; precursor to DGM
- [[react]] — Interleaving reasoning and acting (Yao 2022)
- [[reflexion]] — Verbal-RL agent improvement via self-reflection
- [[toolformer]] — Self-supervised tool use training (Schick 2023)
- [[self-reflection]] — LLM critiquing / revising its own output
- [[centaur-actor]] — Human-AI hybrid actor
- [[metagpt]] — Multi-agent framework with standardized operating procedures
- [[autogen]] — Microsoft's open-source multi-agent framework
- [[standardized-operating-procedures]] — Multi-agent design pattern
- [[skill-md-format]] — Cross-agent skill packaging format

### Retrieval
- [[retrieval-augmented-generation]] — RAG paradigm: condition generation on retrieved text
- [[dense-retrieval]] — Learned vector retrieval
- [[dual-encoder]] — Two-tower architecture enabling efficient retrieval
- [[bm25]] — Classical sparse retrieval baseline
- [[maximum-inner-product-search]] — MIPS / vector index serving
- [[chunked-cross-attention]] — RETRO's mechanism for retrieved-chunk integration
- [[open-domain-qa]] — Retriever + reader QA from a corpus

### Memory management (2023–2026)

**Umbrella + memory-primitive concepts:**
- [[memory-management]] — Umbrella: write/store/retrieve/compact/consolidate/evict/curate lifecycle
- [[memory-stream]] — Append-only NL memory log (Generative Agents)
- [[reflection-mechanism]] — Periodic synthesis of higher-level memories
- [[procedural-memory]] — Memory of skills / how-to (vs declarative facts)
- [[skill-library]] — Growing collection of executable code skills (Voyager)
- [[virtual-context]] — MemGPT's OS-paging-over-prompt abstraction
- [[tiered-memory]] — Hot / warm / cold memory tiers
- [[ebbinghaus-forgetting]] — Exponential forgetting curve applied to LLM memory
- [[episodic-memory-llm]] — Event-based memory in agent designs
- [[memory-evolution]] — A-MEM's pattern: new memories rewrite neighbors
- [[zettelkasten-memory]] — A-MEM's note-taking-derived memory pattern
- [[graph-memory]] — Memory as a relational graph
- [[knowledge-graph]] — Entity/relation/entity triples; semantic web heritage
- [[kv-cache]] — Transformer-internal cache; substrate of EM-LLM's episodic memory
- [[long-context-llm]] — Very long context as memory representation
- [[locomo]] — Long-term Conversational Memory benchmark (Maharana 2024)
- [[bayesian-surprise-segmentation]] — EM-LLM's event boundaries via prediction-violation detection
- [[test-time-memory]] — Memory updated during inference (Titans)
- [[neural-long-term-memory]] — Memory stored in parameters; surprise-driven test-time updates (Titans)
- [[offline-consolidation]] — Memory processing in idle time between queries (Sleep-time Compute)
- [[multi-agent-memory]] — Memory subsystems specialized per memory type (MIRIX)
- [[multi-agent]] — Coordinating multiple LLM agents (debate, roles, recursion); 2026 equal-compute skepticism on whether MAS beats a single agent
- [[memory-construction]] — Building structured memory from a stream (Mem-α)
- [[rl-memory-policy]] — RL-learned memory operations (Memory-R1)

**Memory systems (one page per system):**
- [[generative-agents]] — Memory stream + reflection, recency × importance × relevance retrieval (Park 2023)
- [[memorybank]] — First principled forgetting policy (Ebbinghaus curve) (Zhong 2023)
- [[voyager]] — Skill library as procedural memory (Wang 2023)
- [[memgpt]] — OS-style tiered virtual context with function-call paging (Packer 2023)
- [[em-llm]] — Bayesian-surprise event segmentation inside the KV-cache (Fountas 2024)
- [[titans]] — Test-time parametric memory updates via surprise gradient (Behrouz 2025)
- [[a-mem]] — Zettelkasten memory evolution (Xu 2025)
- [[hipporag-2]] — Hippocampus/neocortex graph memory with Personalized PageRank (Jiménez Gutiérrez 2025)
- [[sleep-time-compute]] — Offline consolidation between queries (Lin/Snell 2025)
- [[mem0]] — Production-grade cross-session memory (Chhikara 2025)
- [[mirix]] — Six-type, multi-agent, multimodal memory (Wang & Chen 2025)
- [[memory-r1]] — RL-learned policy over ADD/UPDATE/DELETE/NOOP (Yan 2025)
- [[mem-alpha]] — RL-learned memory construction with 13× length extrapolation (Wang 2025)

### Multimodal
- [[multimodal-llm]] — LLMs accepting / producing multiple modalities

### Benchmarks
- [[glue]] — General NLP understanding suite
- [[squad]] — Reading-comprehension QA
- [[mmlu]] — Multi-subject multiple-choice
- [[humaneval]] — Python function-completion code benchmark
- [[terminal-bench]] — Terminal/shell agent benchmark; the de-facto yardstick for harness engineering

### Behavior, safety, interpretability
- [[hallucination]] — Confident fabrication
- [[mechanistic-interpretability]] — Reverse-engineering trained networks
- [[functional-emotions]] — Behaviorally-effective emotion representations
- [[sycophancy]] — LLM agreeing-with-user failure mode
- [[reward-hacking]] — Optimizing the proxy at the goal's expense
- [[agentic-misalignment]] — Misaligned behavior in agentic settings
- [[institutional-alignment]] — Aligning agent ecosystems via roles/norms
- [[red-teaming]] — Adversarial probing of LLMs
- [[prompt-injection]] — LLM applications hijacked by attacker text
- [[indirect-prompt-injection]] — Prompt injection via third-party content

### Steering and interpretability (internal-mechanism safety)
- [[steering]] — Umbrella: inference-time activation interventions
- [[guardrailing]] — Paired umbrella: external boundary enforcement
- [[activation-engineering]] — Activation-level intervention framework
- [[activation-addition]] — Adding steering vectors to residual stream
- [[steering-vectors]] — Learned direction vectors used for activation addition
- [[contrastive-activation-addition]] — CAA: steering via contrastive pair differences
- [[representation-engineering]] — RepE: top-down reading/writing of representations
- [[inference-time-intervention]] — ITI: per-head intervention on truthful directions
- [[sparse-autoencoder]] — Decomposes activations into sparse monosemantic features
- [[polysemanticity]] — Single neuron responding to multiple unrelated features
- [[superposition]] — Encoding more features than dimensions via interference
- [[monosemanticity]] — Single neuron / feature with one consistent meaning
- [[linear-representation-hypothesis]] — Concepts are linear directions in activation space

### Datasets and corpora
- [[c4]] — Colossal Clean Crawled Corpus (T5)

### Model families and lineage
- [[gpt-family]] — The OpenAI GPT line: GPT-1 → GPT-4 and beyond

## Sources

### Foundations (1997–2014)
- [[long-short-term-memory-hochreiter-1997]] — LSTM (Hochreiter & Schmidhuber 1997)
- [[efficient-estimation-of-word-representations-mikolov-2013]] — word2vec architectures (Mikolov 2013a)
- [[distributed-representations-of-words-and-phrases-mikolov-2013]] — Negative sampling + phrases (Mikolov 2013b)
- [[glove-pennington-2014]] — GloVe word embeddings (Pennington, Socher, Manning 2014)
- [[sequence-to-sequence-learning-sutskever-2014]] — seq2seq (Sutskever, Vinyals, Le 2014)
- [[neural-machine-translation-bahdanau-2014]] — Soft attention (Bahdanau, Cho, Bengio 2014)

### Transformer + pre-training era (2017–2019)
- [[attention-is-all-you-need-vaswani-2017]] — Transformer (Vaswani et al. 2017)
- [[rlhf-christiano-2017]] — Foundational RLHF paper (Christiano et al. 2017)
- [[bert-devlin-2018]] — BERT (Devlin et al. 2018)
- [[gpt-1-radford-2018]] — GPT-1 (Radford et al. 2018)
- [[transformer-decoder-liu-2018]] — Decoder-only Transformer (Liu et al. 2018)
- [[gpt-2-radford-2019]] — GPT-2 (Radford et al. 2019)
- [[t5-raffel-2019]] — T5 (Raffel et al. 2019)

### Scaling and emergence (2020)
- [[scaling-laws-kaplan-2020]] — Kaplan scaling laws (Kaplan et al. 2020)
- [[dpr-karpukhin-2020]] — Dense Passage Retrieval (Karpukhin et al. 2020)
- [[rag-lewis-2020]] — Retrieval-Augmented Generation (Lewis et al. 2020)
- [[gpt-3-brown-2020]] — GPT-3 / Few-Shot Learners (Brown et al. 2020)
- [[vit-dosovitskiy-2020]] — Vision Transformer (Dosovitskiy et al. 2020)

### Reasoning and agents (2021–2022)
- [[retro-borgeaud-2021]] — RETRO retrieval LM (Borgeaud et al. 2021)
- [[flan-wei-2021]] — FLAN instruction tuning (Wei et al. 2021)
- [[webgpt-nakano-2021]] — WebGPT (Nakano et al. 2021)
- [[instructgpt-ouyang-2022]] — InstructGPT (Ouyang et al. 2022)
- [[chinchilla-hoffmann-2022]] — Chinchilla scaling correction (Hoffmann et al. 2022)
- [[chain-of-thought-wei-2022]] — Chain-of-thought prompting (Wei et al. 2022)
- [[zero-shot-cot-kojima-2022]] — Zero-shot CoT (Kojima et al. 2022)
- [[self-consistency-wang-2022]] — Self-consistency (Wang et al. 2022)
- [[react-yao-2022]] — ReAct (Yao et al. 2022)
- [[constitutional-ai-bai-2022]] — Constitutional AI (Bai et al. 2022)
- [[red-teaming-ganguli-2022]] — Red-teaming LLMs (Ganguli et al. 2022)

### Tool use, orchestration, deeper alignment (2023)
- [[toolformer-schick-2023]] — Toolformer (Schick et al. 2023)
- [[indirect-prompt-injection-greshake-2023]] — Indirect prompt injection (Greshake et al. 2023)
- [[gpt-4-openai-2023]] — GPT-4 Technical Report (OpenAI 2023)
- [[reflexion-shinn-2023]] — Reflexion (Shinn et al. 2023)
- [[tree-of-thoughts-yao-2023]] — Tree of Thoughts (Yao et al. 2023)
- [[dpo-rafailov-2023]] — Direct Preference Optimization (Rafailov et al. 2023)
- [[metagpt-hong-2023]] — MetaGPT (Hong et al. 2023)
- [[autogen-wu-2023]] — AutoGen (Wu et al. 2023)

### Steering / interpretability (2023)
- [[activation-engineering-turner-2023]] — Activation Engineering / ActAdd (Turner et al. 2023)
- [[contrastive-activation-addition-rimsky-2023]] — CAA (Rimsky/Panickssery et al. 2023)
- [[representation-engineering-zou-2023]] — RepE (Zou et al. 2023)
- [[inference-time-intervention-li-2023]] — ITI (Li et al. 2023)
- [[sparse-autoencoders-cunningham-2023]] — SAE foundations (Cunningham et al. 2023)

### Memory systems (2023–2025)
- [[generative-agents-park-2023]] — Generative Agents: memory stream + reflection (Park et al. 2023)
- [[memorybank-zhong-2023]] — MemoryBank: Ebbinghaus forgetting curve (Zhong et al. 2023)
- [[voyager-wang-2023]] — Voyager: skill library as procedural memory (Wang et al. 2023)
- [[memgpt-packer-2023]] — MemGPT: LLM as OS, tiered virtual memory (Packer et al. 2023)
- [[em-llm-fountas-2024]] — EM-LLM: episodic memory inside KV-cache (Fountas et al. 2024)
- [[titans-behrouz-2025]] — Titans: test-time parametric memory (Behrouz et al. 2025)
- [[a-mem-xu-2025]] — A-MEM: Zettelkasten memory evolution (Xu et al. 2025)
- [[hipporag2-jimenez-gutierrez-2025]] — HippoRAG 2: graph memory + PPR (Jiménez Gutiérrez et al. 2025)
- [[sleep-time-compute-lin-2025]] — Sleep-time compute: offline consolidation (Lin & Snell 2025)
- [[mem0-chhikara-2025]] — Mem0: production cross-session memory (Chhikara et al. 2025)
- [[mirix-wang-2025]] — MIRIX: six-typed multi-agent memory (Wang & Chen 2025)
- [[memory-r1-yan-2025]] — Memory-R1: RL-trained memory operations (Yan et al. 2025)
- [[mem-alpha-wang-2025]] — Mem-α: RL-trained memory construction (Wang et al. 2025)
- [[diagnosing-retrieval-vs-utilization-bottlenecks-su-2026]] — Retrieval dominates write-strategy in agent memory; raw chunks beat lossy extraction on LoCoMo (Su et al., ICLR 2026)
- [[memex-rl-wang-2026]] — Memex(RL): indexed experience memory, compress-without-discard + RL read/write (Wang et al., Accenture 2026)

### Frontier-era (2026)
- [[emotion-concepts-anthropic-2026]] — Functional emotions in Claude Sonnet 4.5 (Anthropic)
- [[agentic-ai-and-the-next-intelligence-explosion]] — Evans, Bratton, Agüera y Arcas (2026)
- [[claude-code-agent-doc-2026]] — Claude Code architecture writeup
- [[opencode-agent-doc-2026]] — opencode architecture writeup
- [[pi-mono-agent-doc-2026]] — pi-mono architecture writeup
- [[hermes-agent-doc-2026]] — Hermes Agent architecture writeup
- [[openclaw-agent-doc-2026]] — OpenClaw architecture writeup
- [[cross-agent-comparison-doc-2026]] — Five-agent comparison source
- [[speculative-speculative-decoding-kumar-2026]] — SSD: parallelize drafting + verification; Saguaro ~30% over SD baselines (Kumar et al., Stanford/Princeton/Together 2026)
- [[autoharness-lou-2026]] — AutoHarness: LLM synthesizes its own code harness via Thompson-sampling tree search (Lou et al., Google DeepMind 2026)
- [[agentir-reasoning-aware-retrieval-chen-2026]] — AgentIR: jointly embed reasoning trace + query for Deep Research retrieval (Chen et al., Waterloo 2026)
- [[skillnet-liang-2026]] — SkillNet: open infra to create/evaluate/connect 200k+ AI skills (Liang et al., ZJU consortium 2026)
- [[karl-knowledge-agents-databricks-2026]] — KARL: enterprise search agents via off-policy multi-task RL + KARLBench (Databricks AI Research 2026)

### Agent interoperability protocols
- [[model-context-protocol-anthropic-2024]] — Introducing MCP (Anthropic 2024-11-25)
- [[agent-client-protocol-zed-2025]] — Agent Client Protocol page (Zed, Aug 2025)

### Tool calling, structured outputs, agent skills (2024–2025)
- [[anthropic-agent-skills-2025]] — Anthropic Agent Skills announcement (Oct 2025)
- [[openai-function-calling-2025]] — OpenAI function-calling + Responses API guide
- [[function-calling-llms-promptingguide]] — Conceptual guide: LLM emits JSON, app executes
- [[vllm-tool-calling-2025]] — vLLM tool parsers, constrained decoding backends

### Agent Skills — authoritative docs, specs & academic (2023–2026)
- [[anthropic-engineering-skills-2025]] — Anthropic engineering deep-dive: filesystem mechanics + code-execution model
- [[agentskills-io-spec-2025]] — agentskills.io formal open standard: six-field frontmatter schema (Dec 2025)
- [[openai-codex-skills-2025]] — OpenAI Codex's convergent SKILL.md adoption
- [[agent-skills-survey-xu-2026]] — Agent Skills for LLMs: the dedicated academic survey (Xu & Yan, Zhejiang U 2026)
- [[llm-as-tool-makers-cai-2023]] — LATM: agents author + cache reusable tools (Cai et al., ICLR 2024)
- [[craft-yuan-2023]] — CRAFT: create-then-retrieve specialized toolsets (Yuan et al., ICLR 2024)
- [[lilo-grand-2023]] — LILO: learning interpretable code libraries (Grand et al., MIT, ICLR 2024)
- [[expel-zhao-2023]] — ExpeL: experiential learning via distilled NL insights (Zhao et al., AAAI 2024)

### Skill optimization (2025–2026)
- [[skillopt-yang-2026]] — SkillOpt: controllable text-space optimizer for agent skills; 52/52 best-or-tied (Yang et al., Microsoft/SJTU 2026)
- [[skillmoo-gong-2026]] — SkillMOO: multi-objective (pass-rate × cost) skill optimization via NSGA-II (Gong et al., KCL 2026)
- [[skillreducer-gao-2026]] — SkillReducer: token-efficiency compression of skills (Gao et al., HKUST 2026)
- [[skill1-shi-2026]] — Skill1: unified RL over skill selection/utilization/distillation (Shi et al., USTC 2026)
- [[sage-skill-library-wang-2025]] — SAGE: RL skill-library accumulation across sequential tasks (Wang et al., AWS 2025)
- [[agentic-skills-in-the-wild-liu-2026]] — realistic skill-usage benchmark over ~34k real skills (Liu et al., UCSB/MIT 2026)

### Agentic capabilities (2023, 2025–2026)
- [[gaia-mialon-2023]] — GAIA general-assistant benchmark (Mialon et al. 2023)
- [[deepseek-r1-2025]] — DeepSeek-R1: RL elicits emergent reasoning (DeepSeek 2025)
- [[search-r1-jin-2025]] — Search-R1: RL for multi-turn search (Jin et al. 2025)
- [[magma-yang-2025]] — Magma: unified VLA foundation model (Yang et al., CVPR 2025)
- [[swe-gym-pan-2024]] — SWE-Gym training env + verifier inference scaling (Pan et al., ICML 2025)
- [[swe-bench-live-zhang-2025]] — SWE-bench Live: contamination-resistant (Zhang et al. 2025)
- [[darwin-godel-machine-zhang-2025]] — Darwin Gödel Machine: self-improving agent (Zhang et al. 2025)
- [[ui-tars-2-wang-2025]] — UI-TARS-2 technical report (ByteDance Seed 2025)
- [[agentic-context-engineering-zhang-2025]] — Generator/Reflector/Curator (Zhang et al., ICLR 2026)
- [[agentic-misalignment-lynch-2025]] — Insider-threat agentic misalignment (Lynch et al., Anthropic 2025)
- [[grep-all-you-need-sen-2026]] — Is Grep All You Need? Retriever × harness × tool-delivery on LongMemEval (Sen et al., PwC 2026)
- [[chronos-sen-2026]] — Chronos: temporal-event memory agent (Sen et al., PwC 2026)
- [[swe-bench-jimenez-2023]] — SWE-bench foundational benchmark (Jimenez et al., Princeton, ICLR 2024)
- [[ui-tars-1-qin-2025]] — UI-TARS-1: native GUI agent (Qin et al., ByteDance Seed 2025)

### Code as Harness (2022–2026)
- [[code-as-agent-harness-ning-2026]] — Code as Agent Harness survey (Ning et al., UIUC/Meta/Stanford, 2026, 66 pages)
- [[pal-gao-2023]] — PAL: Program-aided Language Models (Gao et al., ICML 2023)
- [[code-as-policies-liang-2022]] — Code as Policies for embodied control (Liang et al., ICRA 2023)
- [[codeact-wang-2024]] — Executable Code Actions Elicit Better LLM Agents (Wang et al., ICML 2024)
- [[swe-agent-yang-2024]] — SWE-agent: Agent-Computer Interfaces (Yang et al., NeurIPS 2024)
- [[openhands-wang-2024]] — OpenHands platform (Wang et al., ICLR 2025)
- [[agentcoder-huang-2023]] — Multi-agent code generation with iterative testing (Huang et al. 2023)
- [[chatdev-qian-2023]] — Communicative agents for SW dev (Qian et al., ACL 2024)
- [[agentic-harness-engineering-lin-2026]] — Observability-driven harness evolution (Lin et al., Fudan, 2026)
- [[claude-code-dynamic-workflows-2026]] — Introducing dynamic workflows in Claude Code (Anthropic blog, 2026-05-28)
- [[claude-code-dynamic-workflows-harness-2026]] — "A harness for every task": dynamic-workflow patterns + failure modes (Shihipar & Bidasaria, Anthropic, 2026-06-02)
- [[claude-effort-and-ultracode-docs-2026]] — Anthropic effort-parameter docs; defines effort levels + ultracode (2026)
- [[oh-my-claude-ultrawork-2026]] — oh-my-claude plugin README; the unofficial ultrawork mechanic (TechDufus, 2026)
- [[agent-harness-survey-meng-2026]] — "Agent Harness for LLM Agents: A Survey" (Meng et al. 2026; abstract-only — preprints.org CDN-blocked)
- [[externalization-in-llm-agents-zhou-2026]] — review of externalization: memory/skills/protocols/harness engineering (Zhou et al. 2026)
- [[adapting-the-interface-not-the-model-xu-2026]] — LIFE-HARNESS: runtime harness adaptation for deterministic agents (Xu et al., PKU 2026)
- [[retrospective-harness-optimization-pan-2026]] — RHO: harness optimization via self-preference over rollouts (Pan et al. 2026)
- [[evolving-excellence-automated-optimization-brookes-2025]] — Artemis: no-code evolutionary optimization of LLM agents (Brookes et al., TurinTech 2025)

### Foundations (2003, 2016)
- [[godel-machine-schmidhuber-2003]] — Gödel Machines: self-referential universal problem solvers (Schmidhuber 2003)
- [[adaptive-computation-time-graves-2016]] — Adaptive Computation Time for RNNs (Graves, DeepMind 2016)

### Looped Transformers (2018, 2023–2026)
- [[universal-transformer-dehghani-2018]] — Universal Transformers: recurrent transformer + ACT (Dehghani et al., ICLR 2019)
- [[looped-transformers-programmable-computers-giannou-2023]] — Hand-constructed 13-layer transformer simulates a programmable computer (Giannou et al., ICML 2023)
- [[looped-transformers-learning-algorithms-yang-2023]] — Looped transformer matches standard on in-context regression at ~1/12 params (Yang et al., ICLR 2024)
- [[looped-transformers-length-generalization-fan-2024]] — Variable-step looping for length generalization on RASP-L tasks (Fan et al., ICLR 2025)
- [[expressive-power-looped-transformers-xu-2024]] — Approximation rate + timestep encoding fix (Xu & Sato, ICML 2025)
- [[looped-transformers-multi-step-gd-gatmiry-2024]] — Optimal looped solution = multi-step preconditioned GD (Gatmiry et al. 2024)
- [[latent-thoughts-looped-transformers-saunshi-2025]] — k×L looping ≈ kL depth; "latent thoughts" simulate CoT (Saunshi et al., ICLR 2025)
- [[cot-or-loop-xu-2025]] — Formal depth-recursion vs token-recursion separation (Xu & Sato 2025)
- [[parcae-scaling-laws-looped-2026]] — Parcae: LTI stability + scaling laws for looped LMs (Prairie et al. 2026)
- [[loop-think-generalize-2026]] — Recurrent-Depth Transformer with implicit reasoning + grokking (Kohli et al., OSU 2026)

### Newsletter backfill — alphaXiv / NLP digests (2026 Q2)

Surfaced from three months of alphaXiv and NLP-newsletter digests (ingested 2026-06-03). Grouped by theme.

**Agent skills & skill learning**
- [[skill-zero-lu-2026]] — SKILL0: in-context agentic RL for skill internalization (Lu et al. 2026)
- [[scaling-coding-agents-via-atomic-skills-ma-2026]] — composing coding-agent behavior from atomic, reusable skills (Ma et al. 2026)
- [[webxskill-skill-learning-web-agents-wang-2026]] — WebXSkill: skill learning for autonomous web agents (Wang, UNC/Microsoft 2026)
- [[skill-rag-failure-state-aware-retrieval-wei-2026]] — Skill-RAG: hidden-state probing + skill routing for failure-aware retrieval (Wei et al. 2026)
- [[from-skill-text-to-skill-structure-liang-2026]] — scheduling-structural-logical representation for agent skills (Liang et al. 2026)
- [[skillos-learning-skill-curation-self-evolving-agents-ouyang-2026]] — SkillOS: learned skill curation for self-evolving agents (Ouyang et al. 2026)
- [[co-evolving-decision-and-skill-bank-agents-wu-2026]] — COS-PLAY: co-evolving decision policy + skill bank (Wu et al. 2026)
- [[memento-skills-let-agents-design-agents-memento-team-2026]] — Memento: skills let agents design other agents (Memento team 2026)

**Multi-agent systems**
- [[single-agent-vs-multi-agent-tran-2026]] — single agent beats MAS on multi-hop reasoning under equal token budgets (Tran et al. 2026)
- [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]] — same-base agents converge, eroding ensembling/debate diversity (Chen et al. 2026)
- [[drop-the-hierarchy-and-roles-dochkina-2026]] — self-organizing agents outperform designed role hierarchies (Dochkina et al. 2026)
- [[reliability-limits-llm-multi-agent-planning-ao-2026]] — error-compounding limits of multi-agent planning chains (Ao et al. 2026)
- [[reaching-agreement-among-reasoning-llm-agents-ruan-2025]] — consensus/agreement protocols among reasoning agents (Ruan et al. 2025)
- [[theory-of-mind-internal-beliefs-multi-agent-kostka-2026]] — evaluating ToM + internal beliefs in MAS (Kostka et al. 2026)
- [[latent-agents-internalized-multi-agent-debate-yi-2026]] — post-training that internalizes multi-agent debate into one model (Yi et al. 2026)
- [[recursive-multi-agent-systems-yang-2026]] — agents that recursively spawn sub-agents (Yang et al. 2026)
- [[brain-inspired-graph-multi-agent-systems-hao-2026]] — graph-topology message passing for MAS reasoning (Hao et al. 2026)
- [[hyperagents-zhang-2026]] — HyperAgents: scalable multi-agent coordination (Zhang et al. 2026)
- [[multi-user-llm-agents-yang-2026]] — agents serving multiple concurrent users (Yang et al. 2026)
- [[mass-rag-multi-agent-synthesis-rag-xiao-2026]] — MASS-RAG: multi-agent synthesis RAG (Xiao et al. 2026)
- [[autonomous-evolution-eda-tools-multi-agent-abc-yu-2026]] — multi-agent self-evolved ABC for EDA tools (Yu & Ren, DAC 2026)
- [[from-skills-to-talent-organising-heterogeneous-agents-yu-2026]] — organising heterogeneous agents as a real-world company (Yu et al. 2026)

**Self-evolving / self-improving agents**
- [[auton-agentic-ai-framework-cao-2026]] — Auton: declarative architecture for spec/governance/runtime of autonomous agents (Cao et al. 2026)
- [[autogenesis-self-evolving-agent-protocol-zhang-2026]] — Autogenesis: a self-evolving agent protocol (Zhang et al. 2026)
- [[reward-free-self-evolution-via-world-knowledge-exploration-zhang-2026]] — reward-free self-evolution via world-knowledge exploration (Zhang et al. 2026)
- [[agent-world-real-world-environment-synthesis-dong-2026]] — Agent-World: self-evolving environment+task synthesis arena (Dong et al., RUC/ByteDance 2026)
- [[metaclaw-just-talk-agent-meta-learns-xia-2026]] — MetaClaw: an agent that meta-learns and evolves from conversation (Xia et al. 2026)
- [[openclaw-rl-train-agent-by-talking-wang-2026]] — OpenClaw-RL: train any agent simply by talking (Wang et al. 2026)

**Agent memory**
- [[parammem-parametric-reflective-memory-yao-2026]] — ParamMem: parametric reflective memory for language agents (Yao et al. 2026)
- [[memory-intelligence-agent-qiao-2026]] — Memory Intelligence Agent (Qiao et al. 2026)
- [[lightthinker-plus-plus-zhu-2026]] — LightThinker++: from reasoning compression to memory management (Zhu et al. 2026)
- [[memcollab-chang-2026]] — MemCollab: cross-model memory collaboration via contrastive trajectory distillation (Chang et al. 2026)
- [[memfactory-guo-2026]] — MemFactory: unified inference+training framework for agent memory (Guo et al. 2026)
- [[memory-transfer-learning-coding-agents-kim-2026]] — how memories transfer across domains in coding agents (Kim et al. 2026)
- [[ocr-memory-optical-context-retrieval-li-2026]] — OCR-Memory: optical context retrieval for long-horizon agent memory (Li et al. 2026)
- [[stateless-decision-memory-for-enterprise-ai-agents-srinivasan-2026]] — stateless decision memory for enterprise agents (Srinivasan et al. 2026)
- [[understanding-lora-as-knowledge-memory-back-2026]] — empirical analysis of LoRA as knowledge memory (Back et al. 2026)
- [[do-language-models-need-sleep-lee-2026]] — offline recurrence ("sleep") for improved online inference (Lee et al. 2026)

**Coding agents & harnesses**
- [[opendev-terminal-coding-agent-bui-2026]] — OpenDev: scaffolding/harness/context engineering for terminal coding agents (Bui et al. 2026)
- [[coding-agents-effective-long-context-processors-cao-2026]] — coding agents as effective long-context processors (Cao et al. 2026)
- [[effective-strategies-asynchronous-swe-agents-geng-2026]] — strategies for asynchronous software-engineering agents (Geng et al. 2026)
- [[dive-into-claude-code-design-space-liu-2026]] — design space of today's and future AI agent systems, anchored on Claude Code (Liu, MBZUAI 2026)
- [[meta-harness-lee-2026]] — Meta-Harness: end-to-end optimization of model harnesses (Lee et al. 2026)
- [[natural-language-agent-harnesses-pan-2026]] — natural-language agent harnesses (Pan et al. 2026)
- [[composer-2-technical-report-cursor-2026]] — Composer 2 technical report (Cursor 2026)
- [[structuredagent-andor-tree-planning-lobo-2026]] — StructuredAgent: AND/OR-tree planning for long-horizon web tasks (Lobo et al. 2026)
- [[autonomous-long-horizon-ml-research-engineering-chen-2026]] — AiScientist: autonomous long-horizon ML research engineering (Chen et al., RUC 2026)
- [[embarrassingly-simple-self-distillation-zhang-2026]] — simple self-distillation improves code generation (Zhang et al. 2026)

**Agentic RL, post-training & distillation**
- [[pivotrl-high-accuracy-agentic-post-training-yi-2026]] — PivotRL: high-accuracy agentic post-training at low compute (Yi et al. 2026)
- [[self-distilled-rlvr-yang-2026]] — RLSD: RLVR with self-distillation (Yang et al. 2026)
- [[rethinking-on-policy-distillation-li-2026]] — phenomenology/mechanism/recipe of on-policy distillation (Li et al. 2026)
- [[agenticqwen-small-agentic-lms-with-dual-data-flywheels-lyu-2026]] — AgenticQwen: small agentic LMs via dual data flywheels (Lyu et al. 2026)
- [[vector-policy-optimization-bahlous-boldi-2026]] — VPO: training for diversity improves test-time search (Bahlous-Boldi et al. 2026)

**Reasoning & latent reasoning**
- [[the-latent-space-yu-2026]] — the latent space: foundation, evolution, mechanism, ability, outlook (Yu et al. 2026)
- [[generative-recursive-reasoning-baek-2026]] — generative recursive reasoning (Baek et al. 2026)
- [[adaptive-loops-and-memory-in-transformers-frey-2026]] — looped depth vs memory storage trade-off (Frey et al. 2026)
- [[numina-lean-agent-liu-2026]] — Numina-Lean-Agent: agentic reasoning for formal mathematics (Liu et al. 2026)
- [[when-to-retrieve-during-reasoning-guo-2026]] — adaptive retrieval timing for large reasoning models (Guo et al. 2026)
- [[bayesian-teaching-probabilistic-reasoning-qiu-2025]] — Bayesian teaching enables probabilistic reasoning in LLMs (Qiu et al. 2025)
- [[price-reversal-phenomenon-chen-2026]] — when cheaper reasoning models end up costing more (Chen et al. 2026)

**Architecture, model internals & tech reports**
- [[massive-activations-attention-sinks-sun-2026]] — anatomy of massive activations and attention sinks (Sun et al. 2026)
- [[attention-residuals-kimi-team-2026]] — attention residuals (Kimi team technical report 2026)
- [[attention-to-mamba-cross-architecture-distillation-moudgil-2026]] — recipe for distilling attention models into Mamba (Moudgil et al. 2026)
- [[scalable-moe-training-megatron-core-yan-2026]] — scalable MoE training with Megatron Core (Yan et al. 2026)
- [[nemotron-3-super-nvidia-2026]] — Nemotron 3 Super: MoE hybrid Mamba-Transformer for agentic reasoning (NVIDIA 2026)
- [[minimax-m2-series-minimax-2026]] — MiniMax-M2 series: mini-activations LLM tech report (MiniMax 2026)
- [[cola-continuous-latent-diffusion-language-model-guo-2026]] — Cola: continuous latent diffusion language model (Guo et al. 2026)
- [[elf-embedded-language-flows-hu-2026]] — ELF: embedded language flows (Hu et al. 2026)

**World modeling & surveys**
- [[agentic-world-modeling-foundations-capabilities-laws-chu-2026]] — agentic world modeling: foundations, capabilities, laws (Chu et al. 2026)
- [[survey-workflow-optimization-llm-agents-yue-2026]] — survey: static templates → dynamic runtime graphs for agent workflows (Yue et al. 2026)

**Evaluation, benchmarks & verification**
- [[arc-agi-3-arc-prize-foundation-2026]] — ARC-AGI-3: new challenge for frontier agentic intelligence (ARC Prize Foundation 2026)
- [[alphaeval-evaluating-agents-in-production-lu-2026]] — AlphaEval: evaluating agents in production (Lu et al. 2026)
- [[building-verifiers-for-computer-use-agents-rosset-2026]] — the art of building verifiers for computer-use agents (Rosset et al. 2026)
- [[llm-diagnosis-of-integration-test-failures-ziftci-2026]] — LLM-based automated diagnosis of integration-test failures at Google (Ziftci et al., Google 2026)
- [[screening-is-enough-nakanishi-2026]] — Screening Is Enough (Nakanishi et al. 2026)

**Safety, adversarial & commentary**
- [[claudini-autoresearch-adversarial-attacks-panfilov-2026]] — Claudini autoresearch discovers SOTA adversarial attacks on LLMs (Panfilov et al. 2026)
- [[agentic-ai-next-intelligence-explosion-evans-2026]] — arXiv version of the intelligence-explosion essay (Evans et al. 2026); cf. [[agentic-ai-and-the-next-intelligence-explosion]]

### Newsletter — NLP weekly (2026-06-07 "Top AI Papers")

Surfaced from the NLP-newsletter "Top AI Papers of the Week" digest (May 31 – Jun 7 2026; ingested 2026-06-08).

- [[self-revising-discovery-systems-wang-2026]] — categorical framework: an AI scientist that revises its own representational regime (Wang et al., MIT 2026)
- [[harness-updating-is-not-harness-benefit-lin-2026]] — self-evolution splits into harness-updating (flat across tiers) vs harness-benefit (peaks mid-tier) (Lin et al. 2026)
- [[leap-formal-mathematics-agentic-kung-2026]] — LEAP: agentic Lean scaffold solves all 12 Putnam-2025; Lean-IMO-Bench 10%→70% (Kung et al., Google 2026)
- [[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]] — Effective Feedback Compute: feedback quality (not raw budget) predicts agent success (R²≈0.99) (Zhang et al. 2026)
- [[autolab-long-horizon-auto-research-xu-2026]] — AutoLab: 36-task ultra-long-horizon optimization benchmark; persistence beats a strong start (Xu et al. 2026)
- [[learning-agent-compatible-context-management-yi-2026]] — AdaCoM: an external RL-trained context manager for a frozen agent (Yi et al. 2026)
- [[learn-from-your-own-latents-korchinski-2026]] — sample-complexity theory for JEPA/data2vec latent prediction: exponential data-efficiency gap (Korchinski et al. 2026)
- [[primer-in-post-training-reasoning-data-li-2026]] — first survey of 150+ post-training reasoning-data studies (Li et al. 2026)
- [[harness-1-state-externalizing-search-agents-jiang-2026]] — Harness-1: a 20B RL search agent with environment-side working memory; 0.730 curated recall (Jiang et al. 2026)
- [[scaling-behavior-single-llm-multi-agent-systems-li-2026]] — SIMAS: agent count gives diminishing returns then coordination-overhead degradation (Li et al. 2026)

### Newsletter — alphaXiv weekly (2026-06-10 Trending Papers)

Surfaced from the alphaXiv weekly digest (ingested 2026-06-19).

- [[cosmos-3-omnimodal-world-models-for-physical-ai-nvidia-2026]] — Cosmos 3: omnimodal world model (mixture-of-transformers) for Physical AI; SOTA T2I/I2V + robot policy (NVIDIA 2026)
- [[oprd-on-policy-representation-distillation-yang-2026]] — OPRD: distill by aligning intermediate hidden states on student trajectories; +2.7pt, −54% peak memory (Yang et al., ZJU/Ant 2026)
- [[agents-last-exam-sun-2026]] — ALE: long-horizon professional-task agent benchmark with verifiable outputs; frontier agents <1% full-pass (Sun et al., UC Berkeley 2026)
- [[latent-reasoning-with-normalizing-flows-tu-2026]] — NF-CoT: autoregressive normalizing flows for continuous latent reasoning; +13% pass@1, 1.92× faster than diffusion (Tu et al. 2026)
- [[mai-thinking-1-microsoft-2026]] — MAI-Thinking-1: Microsoft AI's 1T/35B-active MoE "hill-climbing machine," human-data-only (2026; no formal report)

### Newsletter — alphaXiv weekly (2026-06-17 Trending Papers)

Surfaced from the alphaXiv weekly digest (ingested 2026-06-18).

- [[from-agi-to-asi-genewein-2026]] — analytical framework mapping AGI→ASI: four pathways, frictions/bottlenecks (Genewein et al., Google DeepMind 2026)
- [[self-harness-harnesses-that-improve-themselves-zhang-2026]] — Self-Harness: a fixed LLM autonomously improves its own harness on Terminal-Bench-2.0 (Zhang et al., Shanghai AI Lab 2026)
- [[minimax-sparse-attention-lai-2026]] — MSA: blockwise sparse attention; 14.2× prefill / 7.6× decode at 1M context; powers MiniMax-M3 (Lai et al., MiniMax 2026)
- [[latent-spatial-memory-for-video-world-models-wang-2026]] — Mirage: video world model with latent spatial memory; 10.57× faster, 55× less GPU memory (Wang et al., ZJU 2026)
- [[glm-5-2-zai-2026]] — GLM-5.2 announcement: 744B/40B-active MoE, 1M context, IndexShare sparse attention (Z.ai 2026; no formal report)

## Comparisons

- [[cross-agent-comparison-2026]] — Claude Code / opencode / pi / Hermes / OpenClaw side-by-side
- [[agent-skills-cross-vendor]] — Capability-packaging compared across Anthropic / OpenAI / Google / Microsoft / AWS (SKILL.md vs GPTs vs Gems vs Declarative Agents vs Action Groups)

## Queries

- [[from-transformer-to-modern-agents]] — Nine evolutionary tracks from Transformer to 2026 agents
- [[common-features-of-modern-agents]] — The 8 universal features shared by Claude Code / opencode / pi / Hermes / OpenClaw
- [[hooks-in-modern-agents]] — ~30 named hooks across the 5 documented 2026 agents (PreToolUse, SessionStart, Stop, PreCompact, …)
- [[downsides-of-looped-transformers]] — Synthesis of training instability, expressivity gaps, overthinking, perplexity loss, and CoT-stochastic separation across the 10 LT papers
- [[policy-optimization-methods]] — Cross-cutting synthesis of PPO / GRPO / DPO / RLHF / RLAIF / agentic-RL — what they are, what entities and concepts belong, how they differ
- [[fundamental-components-of-harness]] — Three coordinated views of harness anatomy: 6 operational artifacts (Lin 2026), 5 mechanism categories (Ning 2026), and L2 architectural cut (3-layer model). Unified 8-component universal inventory.
- [[memory-systems-anatomy-2026]] — Three-part synthesis on modern agent memory: composition (6 layers + 7-op lifecycle + system inventory), long-term scoping (user-specific + domain-specific management), and the 10+ kinds beyond user/domain (cognitive role, function, substrate, generation, temporal binding).
