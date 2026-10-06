---
title: Deep Learning
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - ai-ml-dl-difference-nvidia-2016.md
  - ai-ml-dl-nn-ibm.md
status: draft
importance: high
---

# Deep Learning

The subset of [[machine-learning]] that uses **multi-layer artificial neural networks** and learns its features directly from raw data such as pixels, audio or text.

- IBM: "the 'deep' in deep learning refers to the depth of layers in a neural network". It gives a rule of thumb: "more than three layers, including the inputs and the output" ([[ai-vs-ml-vs-dl-vs-nn-ibm|IBM]]).
- Neural networks "came and mostly went over the decades" until scale made them work: huge networks with more layers, massive data, and GPUs ([[ai-ml-dl-difference-nvidia-2016|NVIDIA 2016]]).

## How it is trained

Data passes forward through layers of weighted connections. The error is **back-propagated** through the layers to adjust the weights. Training is compute-hungry, which is why GPUs mattered.

## Milestones relevant to this wiki

- **2012:** the deep-learning boom. NVIDIA credits Andrew Ng's YouTube-cat network; the field more usually credits AlexNet (general knowledge).
- **2016:** AlphaGo, which learned partly by self-play.
- **2017:** the [[transformer]] replaces recurrence with [[self-attention]] and becomes the universal substrate. See [[recurrent-neural-network]] for what it replaced.
- **2018 onward:** [[pre-training]] at scale ([[bert]], [[gpt-1]]) leads to [[large-language-model|large language models]].

## Related

[[machine-learning]] · [[large-language-model]] · [[transformer-architecture]] · [[recurrent-neural-network]] · [[pre-training]] · [[artificial-intelligence]]

## References

- [[ai-ml-dl-difference-nvidia-2016]]
- [[ai-vs-ml-vs-dl-vs-nn-ibm]]
