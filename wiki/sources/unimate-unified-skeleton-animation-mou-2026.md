---
title: "UniMate: One Unified Model to Animate Diverse Skeletons"
type: source
created: 2026-10-24
updated: 2026-10-24
sources:
  - unimate-unified-skeleton-animation-mou-2026.md
  - repos/unimate-readme-b78c780.md
tags:
  - 2026
  - paper
status: complete
importance: medium
authors:
  - Linzhan Mou
  - Jiahui Lei
  - Zhiyang Dou
  - Chenyue Cai
  - Chaoyue Song
  - Adam Finkelstein
  - Szymon Rusinkiewicz
introduces:
  - [[unimate]]
year: 2026
venue: SIGGRAPH Asia 2026
arxiv_id: "2609.05415"
---

# UniMate: One Unified Model to Animate Diverse Skeletons

[arXiv:2609.05415](https://arxiv.org/abs/2609.05415), 2026-09-04. **SIGGRAPH Asia 2026**; the venue is stated in the repo README, not in the arXiv text.

| | |
|---|---|
| **Authors** | Mou, Lei, Dou, Cai, Song, Finkelstein, Rusinkiewicz |
| **Affiliations** | [[princeton-university]] (Mou, Cai, Finkelstein, Rusinkiewicz), [[uc-berkeley]], [[mit]], NTU Singapore |
| **Repo** | [Friedrich-M/UniMate](https://github.com/Friedrich-M/UniMate), commit `b78c780` (2026-10-07) |
| **Licences** | code MIT; checkpoints **CC BY-NC 4.0** |

## What it is

[[unimate|UniMate]] is a [[text-to-motion]] model for **any rigged skeleton**: humans, quadrupeds, birds, fish, insects, snakes, and articulated objects such as a lamp or a Gundam. Given a rigged asset and a text prompt, **one model** produces skeletal motion, with **no per-skeleton retraining and no test-time optimization**.

Earlier learned animators either assume a fixed template (SMPL, SMAL) or, like AnyTop, GANimator and SinMDM, need per-skeleton training or reference motions.

## How it works

**Skeleton encoding:**
- Joints are ordered breadth-first from the root, and each rig is scaled by its *topology diameter*.
- Each skeleton is described by:
  - rest positions;
  - parent/child/sibling/ancestor relations;
  - a graph-distance matrix;
  - joint depth;
  - **joint-name text embeddings**;
  - **spectral coordinates** (8 Laplacian eigenvectors).

**Motion features:** 12 per joint per frame: position, a 6D rotation relative to the rest pose, and velocity.

**TADiT (topology-aware diffusion transformer):**
- Skeleton tokens are prepended as an extra "frame".
- Each block runs **JointAttn → TempAttn → FFN**. The temporal branch uses 1D RoPE. The joint branch carries the topology through three mechanisms:
  1. a Graphormer-style **graph-distance and edge-type attention bias**;
  2. **Spec-RoPE**: rotary angles from Laplacian eigenvectors via SignNet. It reduces to 1D RoPE on a chain and is permutation-equivariant;
  3. a **global topology conditioner**: 4 learned queries pool the skeleton tokens.
- Conditioning (timestep + text + topology) enters through AdaLN-Zero. The text encoder is frozen FLAN-T5-Base ([[t5]], [[flan]]).
- Size: d = 512, 8 heads, SwiGLU, RMSNorm, QK-norm. The paper uses 8 blocks; the released config uses 10.

**Objective:** [[flow-matching|flow matching]] (rectified flow) that **predicts velocity**, plus a geodesic SO(3) rotation loss and a smoothness loss. There is no foot-contact loss.

**Inference:** 50 Euler steps with CFG s = 3. The paper reports 50 FPS; a 60-frame clip takes about 1.2 s end to end on an H100. The README calls this "real time".

**Training:** 8 × H100, batch 256, 100k steps, about one day. The released checkpoint trained for 150k steps.

## Data: UniML3D

| Source | Clips |
|---|---|
| Truebones | 1,094 animal clips on 74 skeletons |
| Mixamo | 2,425 human clips |
| Objaverse-XL | 9,487 clips from 6,965 assets |

> [!note] Two sizes for the same dataset
> The paper says **13,006 sequences** (2.14M frames) over "thousands of skeletons". The README at `b78c780` says **13,769 clips on 7,430 skeletons**. Both are recorded here. The released captions were re-processed and may not match the paper's prompts.

**Composition:** 84.1% of clips are bipeds; serpentine (0.3%) and marine (0.8%) are rare.

**Pipeline:**
- Filtering removes about 8.7% of clips as implausible.
- Joint names are standardized by an LLM ([[deepseek]]-V4-Flash, optionally refined by GPT-5).
- Clips are captioned from four rendered views by Qwen3.5-9B.

**Licensing:** Truebones is a paid pack and isn't redistributed; users rebuild those clips from scripts.

## Results

| Comparison | Baseline | UniMate |
|---|---|---|
| Held-out Truebones skeletons (FID ↓ / diversity) | AnyTop + MDM-style text: 2.711 / 8.139 | **0.757 / 9.200** |
| Mesh animation, VBench (dynamic degree / aesthetic) | AnimateAnyMesh 0.352 / 0.498, 15.5 s; V2M4 0.667 / 0.506, 1.64 h | **0.833 / 0.544, 1.2 s** |
| User study (32 people, 5-point scale) | AnimateAnyMesh 2.81, V2M4 2.41 | **4.62** |

- **Ablations (FID):** removing the graph bias gives 0.773; removing Spec-RoPE 0.798; removing the global conditioner 0.825 (jittery); removing rest-pose rotation rebasing 1.179.
- **Foot locking:** an IK foot-locking post-process cuts the skate ratio from 0.107 to 0.023.
- **Zero-shot edits:** cross-topology transfer, in-betweening and text editing, by fixing some tokens during sampling.

## Limitations

- **Physical artifacts:** foot sliding, drift, hovering and ground penetration.
- **Coverage:** fails on rare topologies and out-of-distribution motion (a lamp never tips over).
- **Control:** text and skeleton only, with no spatial constraints.
- **The README's own verdict:** "many motions and skeletons still fail".

## Related

[[unimate]] · [[text-to-motion]] · [[flow-matching]] · [[human-motion-diffusion-model-tevet-2022]] (MDM, the template for text cross-attention) · [[kimodo-kinematic-motion-diffusion-nvidia-2026]] (the human-only, constraint-rich counterpart) · [[rope-scaling]]
