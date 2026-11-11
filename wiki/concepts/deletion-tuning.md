---
title: Deletion Tuning
type: concept
created: 2026-11-11
updated: 2026-11-11
sources:
  - the-surprising-effectiveness-of-deleting-weights-lu-2026.md
builds-on:
  - "[[fine-tuning]]"
  - "[[lora]]"
status: draft
importance: medium
---

# Deletion Tuning

A post-training method that **adapts a pretrained LLM only by choosing which weights to set to zero**. It writes no new values. Introduced by Lu, Yang et al. (NYU and University of Calgary, NeurIPS 2026) in [[the-surprising-effectiveness-of-deleting-weights-lu-2026]].

## How it works

- **What is trained:** one trainable logit per weight of the gate projection in each transformer block's SwiGLU MLP. The base model is frozen.
- **Training:** the weight is multiplied by a sigmoid of the logit (a soft mask). Its sharpness rises over training until it becomes a step.
- **Deployment:** a weight is kept if its logit is positive and zeroed if not. The adapter is a bitmask, about 0.03 bits per masked weight after compression.
- **Initialization:** logits start from Wanda pruning scores (|W| × input-activation norm). Loss: the post-training objective itself (SFT, GRPO or self-distillation) plus a small weight-magnitude regularizer from MaskLLM.
- **Sparsity:** no target is set. Training ends up deleting fewer than 0.05% of all weights.

## What it shows

- **Matches full fine-tuning at 7B and larger:**
  - GRPO on Llama3-8B, Qwen2.5-7B, Qwen2.5-14B and the Qwen3-30B-A3B MoE;
  - SFT on Llama3B;
  - self-distillation on Qwen7B.

  Smaller backbones recover 90–99%.
- **Adapters thousands of times smaller than FFT** and smaller than rank-1 [[lora|LoRA]] (8.4 MB for an 8B model). Switching tasks means scattering zeros into the base weights.
- **Forgets less.** Forward KL to the base model stays ≤ 0.05, and prior-task accuracy stays within one point of base. FFT, LoRA and random sparse updates lose up to 5.4 points. This is relevant to [[continual-learning]].
- **Real new capability:** pass@128 beats the base model, so the mask solves problems the base model never solves.
- **FFT-like update geometry:** low spectrum drift, and changes that avoid the base model's top weights, the signature of RL with verifiable rewards.

## Where it sits

Pruning (Wanda, SparseGPT) deletes weights to *keep* abilities while making inference cheaper. The Lottery Ticket Hypothesis finds sparse subnetworks to *train*. Earlier mask learning (Piggyback, "Masking as an efficient alternative to fine-tuning") adapted vision models and encoder-only language models. Deletion Tuning makes the mask the entire post-training update of a decoder LLM across RL, SFT and distillation, and lets training choose how many weights to delete.

Compared with sparse-update methods (MTH, SHiRA, diff pruning) and additive adapters ([[lora]]), it is the only one that writes no floating-point values.

**Open questions** (the paper proposes them but does not test them):
- Training sequentially across many tasks.
- Merging skills by combining masks.
- Locating what post-training changes by comparing task masks.
- Structured masks that also compress the model.

## References

- [[the-surprising-effectiveness-of-deleting-weights-lu-2026]]
