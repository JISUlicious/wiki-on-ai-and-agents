---
title: "Latent Spatial Memory for Video World Models"
type: source
created: 2026-06-18
updated: 2026-06-18
sources:
  - latent-spatial-memory-for-video-world-models-wang-2026.md
arxiv_id: "2606.09828"
authors:
  - Weijie Wang
  - et al.
first_author: Weijie Wang
year: 2026
venue: arXiv preprint (Zhejiang University)
tags:
  - 2026
status: complete
importance: medium
---

# Latent Spatial Memory for Video World Models

Surfaced via the alphaXiv weekly digest (2026-06-17).

## Summary

This paper introduces **Mirage**, a video world model that maintains 3D spatial consistency across generated frames by caching scene information directly in the diffusion *latent* space rather than as an explicit RGB-colored point cloud. Prior video world models build point-cloud memory in pixel space, which is both computationally expensive (repeated rendering plus VAE encoding on every query) and lossy (the round trip through pixel space discards rich features of the learned latent representation). Mirage replaces this with a persistent 3D cache of latent tokens.

Mirage constructs its memory by lifting latent tokens into 3D via depth-guided back-projection, and queries it by synthesizing novel views through direct latent-space warping — a single latent-resolution projection that avoids the costly rasterize-and-encode round trip. This unified formulation removes both the information loss and the compute burden of pixel-space reconstruction, letting the model faithfully return to previously observed regions even after large camera detours. Given one input image and a user-specified camera trajectory, Mirage generates geometrically consistent video, reaching state-of-the-art results on WorldScore while being far cheaper to run.

## Key points

- Stores spatial memory as a 3D cache in the diffusion latent space, not an RGB point cloud — avoiding pixel-space reconstruction entirely.
- Memory construction: depth-guided back-projection lifts latent tokens into 3D; queries synthesize novel views via direct latent-space warping.
- Eliminates the repeated rasterize-and-encode (render + VAE) round trip that explicit RGB-memory baselines require.
- Up to 10.57x faster end-to-end video generation and ~55x lower GPU memory footprint vs. explicit 3D/RGB-cache baselines.
- Preserves 3D consistency across large camera detours; can faithfully return to previously observed regions.
- State-of-the-art on the WorldScore benchmark, leveraging the geometric prior of the diffusion model.

## Concepts & entities

- [[memory-management]] — Mirage is a memory mechanism (persistent latent 3D cache) for generative video.
- [[latent-reasoning]] — related to operating in latent space rather than decoding to pixels.
- Video world models, diffusion models, point-cloud memory, WorldScore, novel-view synthesis — plain text (no dedicated pages).

## References

- [arXiv:2606.09828](https://arxiv.org/abs/2606.09828) — "Latent Spatial Memory for Video World Models," Wang et al., 2026 (Zhejiang University; Microsoft Research).
- Local source text: `latent-spatial-memory-for-video-world-models-wang-2026.md`.
