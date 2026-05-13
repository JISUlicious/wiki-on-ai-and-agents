---
title: Wiki Index
type: index
created: 2026-05-12
updated: 2026-05-13
---

# Wiki Index

A catalog of all pages in this wiki, organized by category. Within each category, the most central pages are linked first.

## Entities

### Models
- [[transformer]] — The original encoder-decoder Transformer (Vaswani 2017); origin of the [[transformer-architecture]]
- [[bert]] — Bidirectional encoder, masked LM pre-training (Devlin 2018)
- [[transformer-decoder]] — Decoder-only Transformer for Wikipedia summarization (Liu 2018), architectural ancestor of GPT
- [[gpt-1]] — First GPT, 117M params, generative pre-training + fine-tuning (Radford 2018)
- [[gpt-2]] — 1.5B params, zero-shot transfer thesis (Radford 2019)
- [[gpt-3]] — 175B params, in-context learning, few-shot prompting (Brown 2020)
- [[gpt-4]] — Multimodal, RLHF-aligned, undisclosed architecture (OpenAI 2023)
- [[claude]] — Anthropic's LLM family, including Claude Sonnet 4.5 studied in the emotion-concepts paper
- [[palm]] — Google's 540B model, used in the chain-of-thought paper
- [[bart]] — Denoising seq2seq Transformer, generator backbone for RAG
- [[vit]] — Vision Transformer, applies Transformer encoder to image patches (Dosovitskiy 2020)
- [[dpr]] — Dense Passage Retriever, dual-BERT retrieval (Karpukhin 2020)
- [[rag]] — Original RAG system, DPR + BART (Lewis 2020)
- [[retro]] — Retrieval-Enhanced Transformer, retrieval during pre-training at trillion-token scale (DeepMind 2021)

### People — researchers
- [[ashish-vaswani]] — First author of the Transformer paper
- [[noam-shazeer]] — Co-author of Transformer; proposed scaled dot-product + multi-head attention
- [[alec-radford]] — GPT-1 / GPT-2 lead, key OpenAI researcher
- [[ilya-sutskever]] — OpenAI co-founder and Chief Scientist
- [[jacob-devlin]] — First author of BERT
- [[tom-brown]] — First author of GPT-3, later at Anthropic
- [[jared-kaplan]] — Scaling laws first author, Anthropic co-founder
- [[dario-amodei]] — Anthropic co-founder and CEO; was at OpenAI for GPT-2 / GPT-3
- [[sam-mccandlish]] — Anthropic co-founder; was at OpenAI
- [[chris-olah]] — Founder of mechanistic interpretability research program; Anthropic co-founder
- [[patrick-lewis]] — First author of RAG; co-author of DPR
- [[danqi-chen]] — Princeton NLP faculty; co-author of DPR
- [[jason-wei]] — First author of chain-of-thought paper, emergent-abilities paper
- [[shunyu-yao]] — First author of ReAct, Tree-of-Thoughts; Princeton PhD
- [[blaise-aguera-y-arcas]] — Google research VP; co-author of agentic-AI essay
- [[james-evans]] — UChicago / Santa Fe Institute computational social scientist

### Organizations
- [[openai]] — Source of the GPT family, ChatGPT, the OpenAI API
- [[anthropic]] — Claude family; founded 2021 by ex-OpenAI researchers
- [[google-brain]] — Google's deep-learning research org until merging with DeepMind (2023)
- [[google-research]] — Google's broader research org including AI Language teams
- [[deepmind]] — UK AI lab; produced RETRO; merged with Google Brain into Google DeepMind (2023)
- [[fair]] — Facebook AI Research / Meta AI; produced DPR and RAG

### Universities
- [[university-of-toronto]] — Hinton's institutional home; Aidan Gomez on Transformer paper
- [[princeton-university]] — Princeton NLP; Danqi Chen, Shunyu Yao, Karthik Narasimhan
- [[university-of-washington]] — UW NLP; Sewon Min on DPR

## Concepts

### Core architectures and components
- [[transformer-architecture]] — The architectural pattern: attention-only sequence model
- [[decoder-only-transformer]] — The dominant variant for modern LLMs
- [[encoder-decoder]] — Two-stack architecture for seq2seq tasks
- [[attention-mechanism]] — Query/key/value weighted aggregation; central primitive
- [[self-attention]] — Intra-sequence attention; defining op of Transformers
- [[multi-head-attention]] — Parallel attention in projected subspaces
- [[scaled-dot-product-attention]] — The specific attention function in the Transformer
- [[positional-encoding]] — Injecting order without recurrence
- [[residual-connection]] — Skip connections for deep network training
- [[layer-normalization]] — Standard normalization in Transformers

### Tokenization
- [[byte-pair-encoding]] — Subword tokenization (BPE)
- [[wordpiece]] — Likelihood-based subword tokenization, used by BERT
- [[patch-tokenization]] — Treating image patches as tokens (ViT)

### Training paradigms
- [[pre-training]] — Self-supervised training on large corpora
- [[fine-tuning]] — Task-specific adaptation of a pre-trained model
- [[autoregressive-language-modeling]] — Next-token prediction; the canonical LLM objective
- [[masked-language-modeling]] — BERT's bidirectional cloze-style objective
- [[next-sentence-prediction]] — BERT's auxiliary sentence-pair objective (later dropped)
- [[rlhf]] — Reinforcement learning from human feedback; alignment post-training

### Capabilities and prompting
- [[in-context-learning]] — Learning from prompt examples without weight updates (GPT-3)
- [[zero-shot-learning]] — Task from instruction only
- [[few-shot-learning]] — Task from instruction + a handful of examples
- [[chain-of-thought-prompting]] — Prompting with intermediate reasoning steps
- [[emergent-abilities]] — Capabilities that appear with scale
- [[scaling-laws]] — Power-law relationships between loss, params, data, compute

### Agents and tools
- [[llm-agent]] — LLM in a perception-action loop with tools
- [[tool-use]] — LLMs invoking external tools / functions
- [[react]] — Interleaving reasoning and acting (Yao 2022)
- [[reflexion]] — Verbal-RL agent improvement via self-reflection
- [[self-reflection]] — LLM critiquing/revising its own output
- [[centaur-actor]] — Human-AI hybrid actor

### Retrieval
- [[retrieval-augmented-generation]] — RAG paradigm: condition generation on retrieved text
- [[dense-retrieval]] — Learned vector retrieval; dominant modern approach
- [[dual-encoder]] — Two-tower architecture enabling efficient retrieval
- [[bm25]] — Classical sparse retrieval baseline
- [[maximum-inner-product-search]] — MIPS; serving-time retrieval algorithm
- [[chunked-cross-attention]] — RETRO's mechanism for integrating retrieved chunks
- [[open-domain-qa]] — Retriever + reader QA from a large corpus

### Multimodal
- [[multimodal-llm]] — LLMs accepting / producing multiple modalities

### Benchmarks
- [[glue]] — General NLP understanding suite (BERT era)
- [[squad]] — Reading-comprehension QA
- [[mmlu]] — Multi-subject multiple-choice knowledge benchmark (GPT-4 era)
- [[humaneval]] — Python function completion benchmark for code LLMs

### Behavior, safety, interpretability
- [[hallucination]] — Confident fabrication; central reliability problem
- [[mechanistic-interpretability]] — Reverse-engineering trained networks
- [[functional-emotions]] — Behaviorally-effective internal emotion representations (Anthropic 2026)
- [[sycophancy]] — LLM tendency to flatter / agree with the user
- [[reward-hacking]] — Optimizing the proxy reward at the expense of the goal
- [[agentic-misalignment]] — Misaligned behavior in agentic settings
- [[institutional-alignment]] — Aligning agent ecosystems via roles/norms, not just weights

### Model families and lineage
- [[gpt-family]] — The OpenAI GPT line: GPT-1 → GPT-4 and beyond

## Sources

- [[attention-is-all-you-need-vaswani-2017]] — Transformer paper (NeurIPS 2017)
- [[bert-devlin-2018]] — BERT paper (NAACL 2019)
- [[gpt-1-radford-2018]] — GPT-1 technical report (OpenAI 2018)
- [[transformer-decoder-liu-2018]] — Decoder-only Transformer for Wikipedia (ICLR 2018)
- [[gpt-2-radford-2019]] — GPT-2 technical report (OpenAI 2019)
- [[dpr-karpukhin-2020]] — Dense Passage Retrieval (EMNLP 2020)
- [[rag-lewis-2020]] — Retrieval-Augmented Generation (NeurIPS 2020)
- [[gpt-3-brown-2020]] — GPT-3, Few-Shot Learners (NeurIPS 2020 Best Paper)
- [[vit-dosovitskiy-2020]] — Vision Transformer (ICLR 2021)
- [[retro-borgeaud-2021]] — RETRO retrieval-augmented LM (ICML 2022)
- [[chain-of-thought-wei-2022]] — Chain-of-Thought prompting (NeurIPS 2022)
- [[react-yao-2022]] — ReAct (ICLR 2023)
- [[gpt-4-openai-2023]] — GPT-4 Technical Report (OpenAI 2023)
- [[reflexion-shinn-2023]] — Reflexion (NeurIPS 2023)
- [[emotion-concepts-anthropic-2026]] — Emotion concepts and functional emotions in Claude Sonnet 4.5 (Anthropic, Transformer Circuits, April 2026)
- [[agentic-ai-and-the-next-intelligence-explosion]] — Plural / social view of the intelligence explosion (Evans, Bratton, Agüera y Arcas, 2026)

## Comparisons

_No comparison pages yet. Promising future comparisons: GPT vs. BERT (decoder-only vs. encoder-only), RAG vs. RETRO (inference-time vs. training-time retrieval), CoT vs. ReAct (closed reasoning vs. grounded acting), pre-norm vs. post-norm Transformers, scaling laws (Kaplan vs. Chinchilla)._

## Queries

_No filed queries yet._
