---
title: Wiki Index
type: index
created: 2026-05-12
updated: 2026-05-14
---

# Wiki Index

A catalog of all pages in this wiki, organized by category. The wiki currently covers 45 sources, 72 entities, 79 concepts, 1 comparison.

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
- [[faiss]] — Facebook AI Similarity Search library

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

### People — Other
- [[blaise-aguera-y-arcas]] — Google research VP; Paradigms of Intelligence team
- [[james-evans]] — Computational social scientist; UChicago + Santa Fe Institute
- [[mario-zechner]] — pi-mono creator; "badlogic"

### Organizations
- [[anthropic]] — Founded 2021 by Amodei et al.; Claude family
- [[openai]] — Source of GPT family, ChatGPT, the OpenAI API
- [[deepmind]] — UK AI lab; merged with Google Brain in 2023
- [[google-brain]] — Google's deep-learning research org until merger
- [[google-research]] — Google's broader research org
- [[fair]] — Facebook AI Research / Meta AI
- [[nous-research]] — Decentralized AI research collective; Hermes models / agent

### Universities
- [[university-of-toronto]] — Hinton's institutional home
- [[stanford-university]] — Stanford NLP; many papers in this wiki
- [[princeton-university]] — Princeton NLP
- [[university-of-washington]] — UW NLP

## Concepts

### Core architectures and components
- [[transformer-architecture]] — The architectural pattern: attention-only sequence model
- [[decoder-only-transformer]] — The dominant variant for modern LLMs
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
- [[tool-use]] — LLMs invoking external tools / functions
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

### Multimodal
- [[multimodal-llm]] — LLMs accepting / producing multiple modalities

### Benchmarks
- [[glue]] — General NLP understanding suite
- [[squad]] — Reading-comprehension QA
- [[mmlu]] — Multi-subject multiple-choice
- [[humaneval]] — Python function-completion code benchmark

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

### Frontier-era (2026)
- [[emotion-concepts-anthropic-2026]] — Functional emotions in Claude Sonnet 4.5 (Anthropic)
- [[agentic-ai-and-the-next-intelligence-explosion]] — Evans, Bratton, Agüera y Arcas (2026)
- [[claude-code-agent-doc-2026]] — Claude Code architecture writeup
- [[opencode-agent-doc-2026]] — opencode architecture writeup
- [[pi-mono-agent-doc-2026]] — pi-mono architecture writeup
- [[hermes-agent-doc-2026]] — Hermes Agent architecture writeup
- [[openclaw-agent-doc-2026]] — OpenClaw architecture writeup
- [[cross-agent-comparison-doc-2026]] — Five-agent comparison source

## Comparisons

- [[cross-agent-comparison-2026]] — Claude Code / opencode / pi / Hermes / OpenClaw side-by-side

## Queries

_No filed queries yet._
