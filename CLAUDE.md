# LLM Wiki Schema

## Domain

This wiki is a personal knowledge base on **Large Language Models and the surrounding ML/AI landscape**. The corpus skews toward foundational LLM research papers (transformer family, training methods, retrieval, agentic patterns) but the scope is intentionally broader: machine learning techniques, AI products and companies, infrastructure, deployment, evaluation, and adjacent ideas are all in-scope.

The wiki is read and written by an LLM. It is also designed to be browsed in Obsidian by a human.

## Source Types

Expected source types and how to handle each:

- **arXiv papers (PDFs)** — Foundational. Most filenames already encode the arXiv ID (e.g., `Transformar Vaswani 1706.03762.pdf`). Source-summary pages MUST record the arXiv ID, year, and a link of the form `https://arxiv.org/abs/{ID}`. Naming for the source-summary page: `{kebab-title}-{first-author-last}-{year}.md` or `{kebab-title}-{arxiv-id}.md`. Prefer the title-based form when title is distinctive (e.g., `attention-is-all-you-need-vaswani-2017.md`).
- **Articles / blog posts (markdown, .mhtml)** — Common for opinion, agentic AI commentary, lab posts. Source-summary page names the article and author.
- **Conference / venue papers without arXiv** — Same as arXiv but skip arXiv ID; record venue + year.
- **Books, talks, lecture notes** — Treat as long-form sources; consider chapter-level sub-pages if substantive.

Source files themselves live in `sources/` and are **immutable**. Wiki pages summarize and interlink them; do not edit source files.

## Page Types

### Entities
Distinct things: people, organizations, products, places, events, **and models**.
- Directory: `wiki/entities/`
- Naming: `kebab-case-name.md`
- Every entity page MUST carry **exactly one classification tag** from the vocabulary below (see SKILL.md → Tag Policy → "Entity classification tags").

**Entity classification vocabulary** (this is the controlled set for this wiki):

| Tag | Use for |
|---|---|
| `model` | Named ML models: GPT-3, BERT, Transformer, Claude 3.5 Sonnet, T5, LLaMA-2, ViT, DPR. Each gets its own page; mixed-architecture or general terms (e.g., "decoder-only transformer") are *concepts*, not entities. |
| `person` | Researchers, founders, public figures (one page per person; affiliation history in the body). |
| `org` | For-profit labs and companies (OpenAI, Anthropic, FAIR, DeepMind, Google Brain/Research). |
| `university` | Academic institutions (Stanford, Princeton, UofT, UW). Kept distinct from `org` for cleaner filtering of academic vs. industrial affiliations. |
| `product` | Consumer or developer products distinct from the underlying model (ChatGPT, Claude.ai, GitHub Copilot, Cursor). Also software libraries (FAISS, PyTorch). |
| `event` | Workshops, conferences, releases, incidents that warrant their own page. Rare. |

Lint enforces that every entity page has exactly one of these tags. To extend the vocabulary, update this table and `lint.md`.

### Concepts
Ideas, theories, techniques, patterns, methodologies, architectures, benchmarks.
- Directory: `wiki/concepts/`
- Naming: `kebab-case-name.md`
- Architectures (`transformer-architecture`, `mixture-of-experts`, `state-space-model`) are concepts.
- Training/inference techniques (`rlhf`, `dpo`, `chain-of-thought`, `speculative-decoding`) are concepts.
- Evaluation benchmarks (`mmlu`, `humaneval`, `arc-agi`) are concepts.
- Open research questions/agendas (`scaling-laws`, `mechanistic-interpretability`, `alignment`) are concepts.

### Source Summaries
One page per ingested source document. Captures key claims, methods, results, and takeaways.
- Directory: `wiki/sources/`
- Naming for arXiv papers: `{kebab-title}-{first-author-last}-{year}.md` (preferred) or fall back to `{arxiv-id}.md`.
- Must link out to the canonical entities and concepts the source discusses.
- Must record arXiv ID, venue (if known), publication year, and author list in frontmatter or body.

### Comparisons
Side-by-side analyses, tradeoff tables, technique-vs-technique discussions.
- Directory: `wiki/comparisons/`
- Naming: `kebab-case-comparison-title.md` (e.g., `rag-vs-fine-tuning.md`, `gpt-3-vs-gpt-4.md`).

### Queries
Filed answers to questions asked against the wiki.
- Directory: `wiki/queries/`
- Naming: `kebab-case-question-slug.md`.

## Conventions

### General
- All wiki pages use YAML frontmatter with: `title`, `type`, `created`, `updated`, `sources`, `tags`.
- Use Obsidian-style `[[wikilinks]]` for internal links. Filename without extension or path prefix.
- File names are kebab-case, no spaces, no underscores.
- Dates use `YYYY-MM-DD` format.
- When new information contradicts existing content, use `> [!warning] Contradiction` callouts and note both claims with their sources.

### arXiv IDs
- Whenever a source has an arXiv ID, record it in the source-summary page's frontmatter under `arxiv_id` and in the body as a link: `[arXiv:1706.03762](https://arxiv.org/abs/1706.03762)`.
- arXiv IDs are also acceptable as source-summary filenames as a fallback (`1706.03762.md`), but title-based names are preferred for readability.

### Model lineage / family trees
- Model entity pages MUST record family lineage when applicable, via dedicated frontmatter fields:
  ```yaml
  family: gpt        # gpt, llama, claude, gemini, bert, t5, etc.
  predecessor: [[gpt-2]]
  successors:
    - [[gpt-4]]
  ```
- The body of each model entity page should include a **Lineage** section with a short prose description of what was inherited, what changed, and what was scaled.
- When a comparison page exists for adjacent models in a lineage, link it from each model's page.

### Year tags on sources
- Tag source-summary pages with their publication year (e.g., `tags: [2017]`, `tags: [2020, paper]`).
- Years are organizational (not a concept page), so they belong in the tag layer per the tag policy.
- This enables Dataview queries like "all 2023 sources" without bloating the concept namespace.

### Tag policy reminder
- Tags are a thin organizational layer; max 3 per page; kebab-case; MUST NOT collide with an existing page slug.
- Promote a tag to a page once ≥3 pages use it and it names a real concept/entity.

## Frontmatter Schema

```yaml
title: string                          # required
type: entity | concept | source | comparison | query  # required
created: YYYY-MM-DD                    # required
updated: YYYY-MM-DD                    # required
sources:                               # required for non-index pages
  - source-filename.md
tags:                                  # optional, encouraged; max 3
  - tag1                               # for type: entity, MUST include one of:
                                       # model, person, org, university, product, event

# Source-specific (when type: source)
arxiv_id: "1706.03762"                 # if applicable
venue: NeurIPS 2017                    # if known
authors:
  - Ashish Vaswani
  - Noam Shazeer
year: 2017

# Model-entity-specific (when type: entity and the entity is a model)
family: gpt
predecessor: [[gpt-2]]
successors:
  - [[gpt-4]]
parameters: 175B                       # if disclosed
release_date: 2020-06-11               # if known
```

## Maintenance Notes

This file co-evolves with the wiki. Update it when:
- A new recurring source type appears (e.g., podcast transcripts).
- A new entity sub-type emerges that needs naming conventions (e.g., evaluation suites).
- A new tag becomes a stable part of the vocabulary.

The wiki was initialized on 2026-05-12 with a corpus of foundational LLM papers (Transformer, BERT, GPT-1/2/3/4, RAG, RETRO, DPR, ViT, Chain-of-Thought, ReAct, Reflexion) plus articles on agentic AI and emotion concepts. The first ingests will exercise and refine these conventions.
