---
title: Overview
type: overview
created: 2026-05-12
updated: 2026-05-13
tags: []
---

# Overview

A synthesis of the wiki's current contents. The corpus spans 2017–2026 and traces the rise of [[transformer-architecture|Transformer-based]] language modeling from the original Vaswani et al. paper through the post-GPT-4 agentic and interpretability era.

## Summary

The wiki currently covers **16 ingested sources** and ~100 derived entity/concept pages. The corpus is dominated by foundational LLM research:

- The [[transformer|Transformer]] (2017) — the architectural foundation underneath everything else.
- The first wave of pre-trained Transformer LMs — [[bert]] (encoder, masked LM, 2018) and [[gpt-1]] (decoder-only, autoregressive, 2018) — establishing the [[pre-training|pre-train]] + [[fine-tuning|fine-tune]] recipe.
- The [[gpt-family|GPT scale-up]] — [[gpt-2|GPT-2]] (2019, zero-shot transfer), [[gpt-3|GPT-3]] (2020, [[in-context-learning|in-context learning]] at 175B), [[gpt-4|GPT-4]] (2023, multimodal + RLHF).
- The [[retrieval-augmented-generation|retrieval]] line — [[dpr|DPR]] (2020), [[rag|RAG]] (2020), [[retro|RETRO]] (2021).
- The [[llm-agent|agentic]] / reasoning line — [[chain-of-thought-prompting|CoT]] (2022), [[react|ReAct]] (2022), [[reflexion|Reflexion]] (2023).
- The [[multimodal-llm|multimodal]] extension — [[vit|ViT]] (2020).
- The current frontier of interpretability / alignment — [[emotion-concepts-anthropic-2026|Emotion Concepts]] (Anthropic 2026), [[agentic-ai-and-the-next-intelligence-explosion|Agentic AI essay]] (2026).

## Key Themes

### 1. The Transformer as universal substrate

Every model in this wiki — text, image, multimodal — is built on the [[transformer-architecture|Transformer]]. The arc 2017–2026 is a story of how broadly that one architecture stretches: [[bert|encoder-only]] for understanding, [[decoder-only-transformer|decoder-only]] for generation, [[transformer|encoder-decoder]] for translation/seq2seq, [[vit|ViT]] for vision, [[multimodal-llm|multimodal extensions]] for everything else. The core invariants — [[self-attention]], [[multi-head-attention]], [[residual-connection|residuals]] + [[layer-normalization]] — change very little across this range.

### 2. Scale as the dominant lever

[[scaling-laws|Scaling laws]] (Kaplan 2020, Chinchilla 2022) say loss is a smooth power-law function of parameters, data, and compute. Downstream capabilities follow ([[emergent-abilities|sometimes smoothly, sometimes with apparent emergence]]). The [[gpt-family|GPT family]] is the most legible scaling experiment in the corpus: GPT-1 (117M) → GPT-2 (1.5B) → GPT-3 (175B) → GPT-4 (undisclosed) traces a ~1000×+ expansion that produced [[in-context-learning]], [[chain-of-thought-prompting|CoT reasoning]], and (eventually) commercial-product viability.

### 3. The shift from fine-tuning to prompting to agents

Three regimes, in temporal order:

- **Fine-tune everything** (2018–2020): one pre-trained backbone, many fine-tuned task-specific heads. [[bert]] and [[gpt-1]] popularized this.
- **Prompt everything** (2020–2022): [[gpt-3]]'s [[in-context-learning|ICL]] meant a single model could be repurposed via prompts. [[chain-of-thought-prompting|CoT]] extended this to multi-step reasoning.
- **Run agents over everything** (2022–): [[react|ReAct]] introduced the prototypical LLM-agent loop with [[tool-use|tool use]]. [[reflexion|Reflexion]] added cross-episode self-reflection. The frontier in 2026 is robust, long-horizon agentic systems.

### 4. Retrieval as a complementary scale axis

The retrieval thread — [[dpr]] (2020), [[rag]] (2020), [[retro]] (2021) — established that **non-parametric memory** is a real, separate lever. [[retro-borgeaud-2021|RETRO showed 7.5B + retrieval ≈ 175B parametric-only]] — a 25× parameter-efficiency win. Retrieval also addresses [[hallucination]], permits knowledge updates without retraining, and provides inspectable provenance. Modern production LLM systems almost universally use some form of [[retrieval-augmented-generation|RAG]].

### 5. Alignment, interpretability, and the limits of behaviorism

The most recent sources point toward a methodological shift:

- [[rlhf|RLHF]] (and its DPO/RLAIF descendants) align models behaviorally via preferences. This works but doesn't open the black box.
- [[mechanistic-interpretability]] (Anthropic, Olah and colleagues) aims to reverse-engineer the algorithms inside frontier models. [[emotion-concepts-anthropic-2026|Sofroniew et al. (2026)]] is a striking concrete example: linear "[[functional-emotions|emotion vectors]]" in Claude Sonnet 4.5 causally drive specific alignment failures ([[reward-hacking]], blackmail, [[sycophancy]]).
- [[agentic-ai-and-the-next-intelligence-explosion|Evans, Bratton & Agüera y Arcas (2026)]] argue the alignment story has to scale to **billions of agents** in social configurations, which [[rlhf]]-style training cannot accomplish alone — proposing [[institutional-alignment|institutional alignment]] (roles, protocols, separation of powers) as a structural complement.

The pattern: from behavior management → to circuit-level understanding → to ecosystem-level governance.

### 6. The OpenAI → Anthropic diaspora

A sociological observation surfaced by the wiki: many co-authors of [[gpt-2-radford-2019|GPT-2]] and [[gpt-3-brown-2020|GPT-3]] — [[dario-amodei]], [[jared-kaplan]], [[tom-brown]], [[sam-mccandlish]], [[chris-olah]] — went on to co-found [[anthropic]] (2021). The two labs together set the frontier-LLM agenda from 2018 onward.

## Open Questions

- **What architecture replaces the Transformer**, if any? State-space models (Mamba, S4) and hybrids ([[retrieval-augmented-generation|retrieval-augmented]] variants) are active research; this wiki doesn't yet have papers on them.
- **What scales beyond pre-training?** Inference-time scaling via reasoning models (o1-style) suggests new scaling regimes — none in this corpus yet.
- **How faithful is [[chain-of-thought-prompting|CoT]] reasoning?** Lanham et al. 2023 and others suggest chains can be post-hoc rationalizations. Not yet in wiki.
- **What does scalable [[institutional-alignment]] actually look like in practice?**
- **Are [[functional-emotions]] a vestigial artifact of training data or actually useful for the Assistant role?** Both are plausible; the 2026 paper doesn't fully settle it.

## Suggested follow-up sources

To round out the corpus, future ingests could include: Chinchilla scaling laws (Hoffmann et al. 2022), InstructGPT/RLHF (Ouyang et al. 2022), LLaMA (Touvron et al. 2023), Constitutional AI (Bai et al. 2022), Toolformer (Schick et al. 2023), Mixtral / Mistral, Mamba / S4, OpenAI o1 / reasoning models, Anthropic SAE / monosemanticity work.
