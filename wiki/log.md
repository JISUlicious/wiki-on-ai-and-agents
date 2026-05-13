---
title: Activity Log
type: log
created: 2026-05-12
updated: 2026-05-13
---

# Activity Log

Chronological record of wiki operations. Newest entries first.

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
