---
title: "How Transferable Are Features in Deep Neural Networks? (Yosinski et al.)"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - transferable-features-deep-nets-yosinski-2014.md
tags:
  - 2014
  - paper
status: complete
importance: high
authors:
  - Jason Yosinski
  - Jeff Clune
  - [[yoshua-bengio]]
  - Hod Lipson
introduces:
  - [[transfer-learning]]
year: 2014
venue: "NIPS 2014"
arxiv_id: "1411.1792"
---

# How Transferable Are Features in Deep Neural Networks? (Yosinski et al.)

**Source**: `sources/transferable-features-deep-nets-yosinski-2014.md`, text extracted from [arXiv:1411.1792](https://arxiv.org/abs/1411.1792).
- **Authors:** Jason Yosinski (Cornell), Jeff Clune (Wyoming), [[yoshua-bengio|Yoshua Bengio]] (Montréal) and Hod Lipson (Cornell).
- **Venue:** NIPS 2014.

## Summary

First-layer features of image networks are almost always **Gabor filters and colour blobs**, whatever the dataset or objective; a different result "causes suspicion of poorly chosen hyperparameters or a software bug". The paper calls these features **general**. Last-layer features must be **specific** to the task. Somewhere in between there is a transition, and the paper is the first to **measure it layer by layer**.

**Setup:**
- The 1,000 ImageNet classes are split into two 500-class halves, A and B, and an 8-layer AlexNet-style network (Caffe reference implementation) is trained on each.
- For each n = 1…7, the first n layers are copied and frozen, or fine-tuned, and the rest is retrained.
- **Self-copy control (BnB):** copy from B and retrain on B.
- **Transfer (AnB):** copy from A and retrain on B.
- Fine-tuned versions are written BnB⁺ and AnB⁺.

## Results

- **Layers 1–2 are general.** They transfer from A to B "almost perfectly". Layer 3 drops slightly, and layers 4–7 drop more.
- **Two separate causes make transfer fail:**
  1. **Fragile co-adaptation (an optimisation problem).** Even BnB, which copies a network back onto its *own* task, loses accuracy at layers 3–6, especially 4–5. Neighbouring layers had co-adapted in ways that retraining the upper layers alone cannot recover. The authors note: "optimization difficulties may be worse in the middle of a network than near the bottom or top".
  2. **Specificity.** Specialisation of the features themselves dominates at layers 6–7.
- **Task distance matters.** On a man-made vs natural split (551 and 449 classes), the frozen-feature gap at the last layer is **25%**, against **8%** for random similar splits. Transferring even from a distant task beats random weights. With random frozen weights, accuracy falls to near chance from layer 3 on.
- **Pretraining helps even after full fine-tuning.** Transferred-then-fine-tuned networks (AnB⁺) generalise *better* than training on the target task directly. The gain is **+1.6%** on average over layers 1–7, and +2.1% when at least five layers are kept. The longer total training does not explain it, since BnB⁺ trains just as long without the gain.

## Relevance to this wiki

- **The "general → specific" curve** is the transfer-side counterpart of [[visualizing-understanding-convnets-zeiler-2013|Zeiler & Fergus]]'s visual hierarchy. Early layers are shared across tasks; late layers belong to the task.
- **The middle of a CNN is its most entangled part.** The co-adaptation dip at layers 4–5 shows this. It echoes the mid-network findings in LLMs (see [[layer-depth-hierarchy-cnn-vs-llm]]).
- **It explains standard practice:** reuse early features, retrain or fine-tune the late ones ([[fine-tuning]]).

## References

- [[transfer-learning]] · [[feature-hierarchy]] · [[convolutional-neural-network]] · [[alexnet]] · [[fine-tuning]]
- [[visualizing-understanding-convnets-zeiler-2013]] · [[layer-depth-hierarchy-cnn-vs-llm]]
