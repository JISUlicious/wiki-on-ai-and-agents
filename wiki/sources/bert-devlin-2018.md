---
title: "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
arxiv_id: "1810.04805"
venue: NAACL 2019
authors:
  - "[[jacob-devlin]]"
  - Ming-Wei Chang
  - Kenton Lee
  - Kristina Toutanova
year: 2018
tags:
  - 2018
status: complete
importance: high
---

# BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding

**Source**: `sources/bert-devlin-2018.md` (extracted from `BERT Devlin 1810.04805.pdf`)
**arXiv**: [1810.04805](https://arxiv.org/abs/1810.04805)
**Authors**: [[jacob-devlin]], Ming-Wei Chang, Kenton Lee, Kristina Toutanova ([[google-research|Google AI Language]])

## Summary

BERT introduces a pre-training recipe for the [[transformer-architecture|Transformer encoder]] that produces **deep bidirectional** language representations — every token's representation is conditioned on both its left and right context, in every layer. Prior pre-training approaches were either left-to-right language models (e.g., [[gpt-1|OpenAI GPT]]) or shallow concatenations of independently-trained left-to-right and right-to-left models (ELMo). BERT removes this unidirectionality constraint by replacing the standard language-modeling objective with [[masked-language-modeling|masked language modeling (MLM)]] — randomly masking 15% of input tokens and training the model to predict them from bidirectional context.

The model is the [[transformer]] encoder stack used as-is, with two pre-training tasks: MLM and [[next-sentence-prediction|next-sentence prediction (NSP)]]. After pre-training on BookCorpus + English Wikipedia (~3.3B words), the same architecture is **fine-tuned end-to-end** on each downstream task with only a single additional output layer — minimal task-specific machinery.

BERT obtained state of the art on 11 NLP tasks at the time of release: GLUE 80.5 (+7.7 points over the prior best), SQuAD v1.1 F1 93.2, SQuAD v2.0 F1 83.1, MultiNLI 86.7. It became the dominant pre-trained encoder for NLP and triggered a wave of follow-up work (RoBERTa, ALBERT, DistilBERT, ELECTRA, etc.). The released checkpoints — BERT-Base (110M) and BERT-Large (340M) — were among the most-downloaded models of the 2018–2020 era.

## Key Points

- **Architecture**: standard [[transformer]] encoder. BERT-Base: L=12, H=768, A=12, 110M params (matched to [[gpt-1]] for comparison). BERT-Large: L=24, H=1024, A=16, 340M params.
- **MLM**: randomly select 15% of [[wordpiece|WordPiece]] tokens; replace 80% with `[MASK]`, 10% with a random token, 10% unchanged. The 80/10/10 split mitigates the train/serving mismatch since `[MASK]` never appears at fine-tune time.
- **NSP**: binary classification of whether sentence B follows sentence A in the corpus. Helps QA and NLI tasks.
- **Special tokens**: `[CLS]` at the start whose final hidden state aggregates the sequence for classification; `[SEP]` between sentence pairs. Segment embeddings (A/B) added to inputs.
- **[[fine-tuning|Fine-tuning]] vs. feature-based**: BERT is fine-tuned end-to-end on each task with just one new output layer. Feature-based use (frozen BERT embeddings into a task-specific model) also works but is slightly worse.
- **Result**: 11 task SOTAs; the result on [[glue|GLUE]] (+7.7 points) was particularly striking and signaled a paradigm shift in NLP.

## Entities Mentioned

- [[jacob-devlin]] — first author, designed BERT
- [[google-research]] (Google AI Language) — author affiliations
- [[bert]] — the model itself
- [[gpt-1|OpenAI GPT]] — primary contrast point throughout the paper (left-to-right vs. bidirectional)

## Concepts Discussed

- [[masked-language-modeling]] — the central new objective
- [[next-sentence-prediction]] — the auxiliary objective
- [[wordpiece]] — tokenization (very close cousin to [[byte-pair-encoding|BPE]])
- [[pre-training]] — general paradigm
- [[fine-tuning]] — vs. feature-based transfer
- [[transformer-architecture]] — uses the encoder-only variant
- [[self-attention]] — bidirectional in BERT, masked in GPT
- [[glue]] — benchmark suite used for evaluation
- [[squad]] — reading-comprehension benchmark

## Notable Quotes

> "BERT alleviates the previously mentioned unidirectionality constraint by using a 'masked language model' (MLM) pre-training objective, inspired by the Cloze task (Taylor, 1953)." — §1

> "BERTBASE was chosen to have the same model size as OpenAI GPT for comparison purposes. Critically, however, the BERT Transformer uses bidirectional self-attention, while the GPT Transformer uses constrained self-attention where every token can only attend to context to its left." — §3 BERT

## References

_Original source: `sources/bert-devlin-2018.md`_
_Code & checkpoints: https://github.com/google-research/bert_
