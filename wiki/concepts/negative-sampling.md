---
title: Negative Sampling
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - mikolov-skipgram-2013.md
status: complete
importance: high
---

# Negative Sampling

Negative sampling is a training objective for learning representations via contrast. Given a positive `(query, key)` pair (e.g., a word and one of its real context words), the model is also presented with `k` randomly-sampled "negative" pairs (the same query with unrelated keys), and trained to score the positive higher than the negatives under a binary logistic loss.

Introduced in [[distributed-representations-of-words-and-phrases-mikolov-2013|Mikolov et al. (2013b)]] as a faster alternative to hierarchical softmax for Skip-gram. The Skip-gram loss with `k` negatives is:

```
L = log σ(v_O · v_I) + Σ_{n=1..k} E_{w_n ~ P_n(w)} log σ(−v_{w_n} · v_I)
```

where `v_I` is the input-word vector, `v_O` is the true context-word vector, and `w_n` are sampled from a noise distribution (typically `unigram(w)^(3/4)`).

## Where it shows up everywhere

Negative sampling is conceptually identical to the **InfoNCE** / contrastive objective used in:

- [[dpr|DPR]] (Karpukhin 2020): in-batch negatives + 1 BM25-hard negative per question — same loss family.
- **CLIP** (Radford et al. 2021): paired (image, caption) positives, all other image-caption pairs in the batch as negatives.
- **SimCLR** (Chen et al. 2020): contrastive self-supervised vision.
- **Sentence-Transformers** (Reimers & Gurevych 2019): the dominant sentence-embedding training recipe.
- **DPO** ([[dpo-rafailov-2023|Rafailov 2023]]): the binary preference loss is a structural cousin — given a preferred and dispreferred response, push the model toward the preferred.

The pattern — turn a dense matching problem into a binary "is this the right partner?" classification with cheap-to-sample negatives — is one of the most reused ideas in modern ML.

## Sampling distribution

A subtle but important detail: in word2vec, negatives are drawn from `P(w)^(3/4)`, not from `P(w)` directly. The 3/4 power flattens the distribution slightly, oversampling rare words and undersampling extremely frequent ones (which are also subsampled separately). This is one of those engineering tricks that's never theoretically derived but consistently outperforms the "principled" choice.

## References

- [[distributed-representations-of-words-and-phrases-mikolov-2013]]
