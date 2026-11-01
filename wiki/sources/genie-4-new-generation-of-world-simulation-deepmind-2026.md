---
title: "Genie 4: A New Generation of World Simulation"
type: source
created: 2026-10-30
updated: 2026-10-30
sources:
  - genie-4-new-generation-of-world-simulation-deepmind-2026.md
authors:
  - Raia Hadsell
  - Diego Rivas
  - Hang Qi
introduces:
  - [[genie-4]]
year: 2026
venue: Google DeepMind blog (with a companion post on the Google Keyword blog)
status: complete
importance: high
tags:
  - 2026
  - blog-post
---

# Genie 4: A New Generation of World Simulation

**Raia Hadsell, Diego Rivas, Hang Qi** ([[deepmind|Google DeepMind]]), 27 Oct 2026. [DeepMind blog post](https://deepmind.google/blog/genie-4-a-new-generation-of-world-simulation/), plus a short [companion announcement](https://blog.google/innovation-and-ai/models-and-research/google-deepmind/introducing-genie-4/) on the Google blog. This is a product/research announcement with demo videos. There is **no paper, no technical report and no benchmark numbers**.

## Summary

[[genie-4|Genie 4]] is DeepMind's new **real-time interactive [[world-model|world model]]**: you describe a scene (or give a reference image), then walk around it with keyboard and mouse while the model generates each frame live. You can also type text prompts that change the scene as it runs. It succeeds [[genie-3|Genie 3]] (Aug 2025).

The post claims four advances:

1. **Steering in real time with language.** Change the weather, move the camera, add a person or object. A Gemini model rewrites your prompt to fit the scene and, *between* your inputs, keeps steering the world so it continues coherently.
2. **Realism in physics and people.** Aerodynamics, collisions and, as "the most dramatic leap", humans that move naturally and can be steered.
3. **Native audio.** Genie 4 is called DeepMind's **first real-time world model that generates audio natively**: ambient sound, music and **interactive dialogue** that reacts to user input, produced together with the video.
4. **Gemini as an agent in the loop.** **Gemini 3.8 Flash** watches the generated frames, breaks a goal into steps, checks from incoming frames whether each step succeeded, and re-plans when the world or the user changes something. The aim is reliable causality and multi-step events.

**Built on Gemini.** "Gemini Omni" anchors the simulation in real-world appearance and behaviour, and other Gemini models supply the reasoning that keeps it coherent. The post does not describe the architecture, training data, resolution, frame rate or model size.

## Key points

- **Robotics demo with Gemini 4 Argon.** [[gemini-4-argon-agents-beat-minecraft-faldor-2026|Gemini 4 Argon]] splits a user task into steps and feeds them to Genie 4 as **text captions interleaved while the video renders**. The example shows Apptronik's Apollo humanoid playing checkers. The post links this to Gemini Robotics, as a way to simulate many scenarios before running them on real robots.
- **Trusted-tester uses** (described in prose only, no numbers):
  - filmmakers using it as a "draft mode" for previsualisation, re-running takes with different camera, lighting and dialogue;
  - photojournalists turning one archival photo into an explorable scene;
  - game developers going from idea to prototype before writing engine code;
  - enterprises generating simulation scenarios.
- **Future direction: "hybrid world models".** Code and language models handle structure and narrative planning, and a pixel model renders what the world looks, sounds and acts like. Genie + Gemini is presented as an early step.
- **Safety.** Internal safety evaluations, a trusted-tester-first rollout, and **SynthID** watermarks on all generated video and audio.
- **Availability.** A research preview for trusted testers only. "Coming soon" to Google AI Ultra subscribers in Project Genie and to early-access partners in the Gemini API and AI Studio. There is a waitlist.
- **Stated limitations.** Visual details and object permanence **drift over time**, especially in longer sessions, after several complex prompt edits, or while moving around the scene. Complex physics, audio accuracy, legible text, spatial accuracy and long-term consistency are all called open problems.

> [!warning] Secondary coverage vs the post
> - **"Up to two minutes, double Genie 3's limit"** (Bloomberg via Quartz) does not appear in either Google post. It also does not match the [Genie 3 announcement](https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/), which says Genie 3 stays consistent "for a few minutes" (visual memory of about one minute). Treat session length as unconfirmed.
> - Some coverage says it is already available to Ultra subscribers. The post says only trusted testers have it; Ultra access is "coming soon".

## What it does not show

- **No quantitative evaluation.** There are no metrics, user studies or comparisons with Genie 3 or other world models (e.g. [[infinite-worlds-versatile-interactions-gao-2026|LingBot-World 2.0]], [[cosmos-3-omnimodal-world-models-for-physical-ai-nvidia-2026|Cosmos 3]]). "Unprecedented realism" is a claim made with demo videos.
- **Demos are curated.** They are recordings of real interactions, but chosen by Google, and many prompts are long, carefully written cinematic descriptions.
- **No evidence about agent training.** Training robots and agents is the stated motivation, but no agent or policy is trained inside Genie 4 in the post. The Argon/Apollo demo *renders* a planned task; it does not train or test a policy. (Genie 3's post at least ran a SIMA agent in generated worlds.)
- **How much is Genie and how much is Gemini is unclear.** Much of the coherence and causality is credited to Gemini models steering from outside the generator. The post does not separate what the world model does alone.

## Concepts & entities

- [[genie-4]] · [[genie-3]]: the model and its predecessor.
- [[world-model]]: a pixel-space, real-time *interactive* world model, unlike the latent world models that dominate the wiki's 2026 sources.
- [[deepmind]] · [[google]]
- [[gemini-4-argon-agents-beat-minecraft-faldor-2026]]: Gemini 4 Argon, used here as the task planner for the robot demo.
- [[code-world-model-coding-agent-as-world-brain-chen-2026]]: a similar "hybrid" split, where code or a language model holds the state and logic and a video model renders it.
- [[infinite-worlds-versatile-interactions-gao-2026]]: an open competitor in real-time interactive world models, with published duration and frame-rate numbers.

## References

- [DeepMind: Genie 4: A new generation of world simulation](https://deepmind.google/blog/genie-4-a-new-generation-of-world-simulation/) (27 Oct 2026)
- [Google blog: We're announcing Genie 4](https://blog.google/innovation-and-ai/models-and-research/google-deepmind/introducing-genie-4/) (27 Oct 2026)
- [DeepMind: Genie 3: A new frontier for world models](https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/) (5 Aug 2025), for comparison
- [Quartz: Google DeepMind is unveiling Genie 4](https://qz.com/google-deepmind-genie-4-world-model-audio-102726), secondary
