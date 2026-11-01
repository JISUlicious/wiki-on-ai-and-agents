---
title: Genie 3
type: entity
created: 2026-10-30
updated: 2026-10-30
sources:
  - genie-4-new-generation-of-world-simulation-deepmind-2026.md
status: draft
importance: medium
tags:
  - model
family: genie
successors:
  - [[genie-4]]
release_date: 2025-08-05
---

# Genie 3

[[deepmind|Google DeepMind]]'s first **real-time** interactive [[world-model|world model]], announced 5 Aug 2025. From a text prompt it generates a world you can navigate live at **24 fps and 720p**, which stays largely consistent **for a few minutes**, with visual memory reaching back about one minute. The consistency is emergent: there is no explicit 3D representation such as a NeRF or Gaussian splats; the world is generated frame by frame.

It introduced **promptable world events**: text edits to a running world, such as changing the weather or adding objects and characters. DeepMind tested it as an agent environment by running a version of its SIMA agent inside generated worlds. Stated limits were a narrow action space for agents and interactions of minutes rather than hours.

## Lineage

- **Predecessors:** Genie 1 and Genie 2 (2024), foundation world models that generated environments for agents but were not real-time.
- **Successor:** [[genie-4]] (Oct 2026). It adds native audio and dialogue, much stronger photorealism and human simulation, and a Gemini agent that steers generation in the loop.

> [!note] Source
> The details on this page come from DeepMind's [Genie 3 announcement](https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/), which was checked while ingesting [[genie-4-new-generation-of-world-simulation-deepmind-2026]]. The announcement has no source page of its own yet.

## References

- [[genie-4-new-generation-of-world-simulation-deepmind-2026]]
