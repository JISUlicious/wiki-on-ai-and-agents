---
title: BERT
type: entity
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
family: bert
predecessor: "[[transformer]]"
successors: []
parameters: "110M (Base) / 340M (Large)"
release_date: 2018-10-11
status: complete
importance: high
tags:
  - model
---

# BERT

BERT (**B**idirectional **E**ncoder **R**epresentations from **T**ransformers) is the pre-trained Transformer encoder introduced in "[[bert-devlin-2018|BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding]]" (Devlin et al., 2018) at [[google-research|Google AI Language]].

It became the dominant pre-trained encoder for NLP between 2018 and 2020, supplanting feature-based models like ELMo and unidirectional pre-training like [[gpt-1]] on understanding tasks (classification, span extraction, retrieval).

## Architecture

The **encoder-only** variant of the [[transformer-architecture|Transformer]] (no decoder, no causal mask). Self-attention is fully bidirectional in every layer.

| Variant | Layers (L) | Hidden (H) | Heads (A) | Params |
|---|---|---|---|---|
| BERT-Base | 12 | 768 | 12 | 110M |
| BERT-Large | 24 | 1024 | 16 | 340M |

BERT-Base was sized to match [[gpt-1]] for a controlled comparison; BERT-Base outperformed it on GLUE despite identical parameter count.

## Training

- **Corpus**: BookCorpus (~800M words) + English Wikipedia (~2.5B words).
- **Objective 1**: [[masked-language-modeling|MLM]] — 15% of [[wordpiece|WordPiece]] tokens are selected; 80% replaced with `[MASK]`, 10% with a random token, 10% kept. Cross-entropy on predicting originals.
- **Objective 2**: [[next-sentence-prediction|NSP]] — binary classification of whether sentence B follows sentence A.
- **Input format**: `[CLS] tok... [SEP] tok... [SEP]`, with segment embeddings A/B and learned positional embeddings.
- **Tokenizer**: WordPiece, 30k vocab.
- **Training compute**: BERT-Large took 4 days on 16 Cloud TPU chips (2018-era hardware).

## Lineage

BERT is encoder-only and trained for representation, not generation. Its successors are improved encoder pre-training schemes (RoBERTa removed NSP and trained longer; ALBERT used parameter sharing; ELECTRA used replaced-token-detection) and various distilled / efficient variants (DistilBERT, TinyBERT). It is **not** an ancestor of the GPT family — those descend from [[transformer-decoder]] (Liu 2018) and [[gpt-1]].

BERT is later used as a building block for retrieval: [[dpr|DPR (Karpukhin et al. 2020)]] uses two BERT encoders as a question encoder and a passage encoder for dense retrieval, which in turn enables [[rag]] and [[retro]].

## Results (at release)

- GLUE: 80.5 (+7.7 over prior SOTA)
- SQuAD v1.1: 93.2 F1
- SQuAD v2.0: 83.1 F1
- MultiNLI: 86.7 accuracy
- 11 task SOTAs total

## References

- [[bert-devlin-2018]]
