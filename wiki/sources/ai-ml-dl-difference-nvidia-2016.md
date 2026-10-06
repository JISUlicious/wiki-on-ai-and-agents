---
title: "What's the Difference Between Artificial Intelligence, Machine Learning, and Deep Learning?"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - ai-ml-dl-difference-nvidia-2016.md
authors:
  - Michael Copeland
year: 2016
venue: NVIDIA Blog
url: https://blogs.nvidia.com/blog/2016/07/29/whats-difference-artificial-intelligence-machine-learning-deep-learning-ai/
tags:
  - 2016
  - blog
status: complete
importance: medium
---

# What's the Difference Between AI, Machine Learning, and Deep Learning? (NVIDIA, 2016)

**Source**: `sources/ai-ml-dl-difference-nvidia-2016.md`, captured from the Wayback Machine snapshot of 2016-08-01. The original URL now redirects to a different NVIDIA article on training vs inference.
**Author**: Michael Copeland ([[nvidia]] blog), per the page's author tag
**Date**: **2016-07-29**, from the URL path

## Summary

The popular **concentric-circles** picture of the field, written months after AlphaGo beat Lee Se-dol:
- AI ("the idea that came first") is the largest circle;
- machine learning ("which blossomed later") sits inside it;
- deep learning ("which is driving today's AI explosion") sits inside both.

## The three circles

- **[[artificial-intelligence|Artificial intelligence]]:** coined when "a handful of computer scientists rallied around the term at the Dartmouth Conferences in 1956". It went through boom and bust "until 2012".
  - **General AI** (human-like) is distinguished from **narrow AI**: specific tasks such as image classification or face recognition.
- **[[machine-learning|Machine learning]]:** "using algorithms to parse data, learn from it, and then make a determination or prediction". The machine is *trained* on data rather than hand-coded.
  - Approaches named: decision trees, inductive logic programming, clustering, reinforcement learning, Bayesian networks.
  - Computer vision was long the best application area but still needed hand-coded features such as edge detectors.
- **[[deep-learning|Deep learning]]:** artificial neural networks "came and mostly went over the decades". They became practical when they were made huge, with more layers and neurons, and trained on massive data using GPUs.
  - The post credits Andrew Ng's Google work learning to recognize cats from 10 million YouTube videos: "Ng put the 'deep' in deep learning."
  - It also cites AlphaGo's self-play training.

## Why it boomed (per the post)

Since about 2015: widely available **GPUs** for parallel processing, near-unlimited storage, and a flood of data ("Big Data").

## Caveats

- This is a vendor blog. NVIDIA sells the GPUs that it credits for the boom.
- The "Ng put the deep in deep learning" framing is a simplification. The deep-learning revival is more usually traced to Hinton's group, e.g. AlexNet in 2012 (general knowledge).
- The post predates Transformers (2017) and LLMs. The fourth, innermost circle is a later addition: see [[large-language-model]].

## Concepts & entities

[[artificial-intelligence]] · [[machine-learning]] · [[deep-learning]] · [[large-language-model]] · [[reinforcement-learning]] · [[nvidia]]
