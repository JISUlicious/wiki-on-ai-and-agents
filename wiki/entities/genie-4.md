---
title: Genie 4
type: entity
created: 2026-10-30
updated: 2026-10-30
sources:
  - genie-4-new-generation-of-world-simulation-deepmind-2026.md
status: draft
importance: high
tags:
  - model
family: genie
predecessor: [[genie-3]]
successors: []
release_date: 2026-10-27
---

# Genie 4

[[deepmind|Google DeepMind]]'s real-time interactive [[world-model|world model]], announced on **27 Oct 2026** as a research preview for trusted testers. From a text prompt or reference image, it generates a photorealistic world that you move through with keyboard and mouse. You can also steer it with text prompts while it runs. It is the first Genie model to **generate audio natively** (ambient sound, music and dialogue that reacts to you).

## What it does

- **Real-time steering.** Text prompts change the running world (weather, camera, new people or objects). A Gemini model adapts each prompt to the scene and keeps the world moving forward between your inputs.
- **Physics and people.** Better realism for physical events and especially for human motion and reactions.
- **Native audio and dialogue.** Generated together with the video.
- **Gemini in the loop.** Gemini 3.8 Flash observes frames, splits goals into steps, checks each step and re-plans. "Gemini Omni" grounds appearance and behaviour. Gemini 4 Argon has been used to drive robot-task simulations, such as an Apptronik Apollo humanoid playing checkers.

No architecture, size, resolution, frame rate, session length or benchmark numbers have been published.

## Single model or system?

What users call "Genie 4" is a **system**: a core Genie 4 generator plus Gemini models that steer it. The evidence, from [[genie-4-new-generation-of-world-simulation-deepmind-2026|the announcement]]:

| Component | Relation to the core generator | Evidence strength |
|---|---|---|
| **Genie 4 (core)** | Generates frames and audio in real time from the history, user actions and text captions. Both posts call it "the Genie 4 model" (singular). | Moderate (implied by wording) |
| **Gemini 3.8 Flash** | **Separate, at run time.** A "guiding agent in the loop": it observes generated frames, splits the goal into steps, verifies each step from new frames and re-plans. | Strong (stated explicitly) |
| **Gemini (prompt steering)** | **Separate, at run time.** Adapts each user text edit to the scene and keeps steering the world between user inputs. | Strong (stated explicitly) |
| **Gemini 4 Argon** | **Separate, demo only.** Plans a robot task and feeds the steps to Genie 4 as text captions interleaved while it renders. | Strong, but limited to an experiment |
| **Gemini Omni** | **Unknown.** "Anchors the simulations in real-world appearance and behavior"; Genie 4 "incorporates and builds on" Gemini capabilities. It could be shared weights, initialisation, training help or a run-time component. | Undisclosed |

DeepMind frames this split (Gemini plans the structure and timeline, Genie renders pixels and sound) as an early step toward **hybrid world models**. No architecture, model size or training details have been published, so the inside of the core generator, and how Omni connects to it, cannot be determined from public sources.

> [!note] Implication for evaluating claims
> Claims of "reliable causality" and "multi-step reasoning" are made for the *system*. No result separates the generator's own consistency from the Gemini steering.

## Lineage

- **Inherited from [[genie-3]]:** real-time autoregressive frame-by-frame generation from a text description, navigation controls, and "promptable world events" (text edits to a running world).
- **What changed:**
  - native audio, including interactive speech;
  - much stronger photorealism and human simulation;
  - an explicit **Gemini agent steering the generator in the loop**. DeepMind frames this as a first step toward "hybrid world models" where language or code models plan the structure and a pixel model renders it.
- **Still the same:** drift and loss of object permanence in longer sessions. Both posts list long-horizon consistency as an open problem.
- Earlier members of the family were Genie 1 and Genie 2 (2024), which generated environments for agents but were not real-time.

## Availability

Trusted testers only at launch. "Coming soon" to Google AI Ultra subscribers through Project Genie and to Gemini API / AI Studio early-access partners. All outputs carry SynthID watermarks.

## References

- [[genie-4-new-generation-of-world-simulation-deepmind-2026]]
