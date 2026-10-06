---
title: "ChatGPT plugins"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - chatgpt-plugins-openai-2023.md
authors:
  - "[[openai]]"
introduces:
  - "[[chatgpt-plugins]]"
year: 2023
venue: OpenAI blog
url: https://openai.com/index/chatgpt-plugins/
tags:
  - 2023
  - blog
status: complete
importance: high
---

# ChatGPT plugins

**Source**: `sources/chatgpt-plugins-openai-2023.md`. The live page returned 403 to automated fetches, so it was captured from the Wayback Machine snapshot of 2023-03-30 at `openai.com/blog/chatgpt-plugins`.
**Author**: [[openai]] (byline OpenAI; long contributor list)
**Date**: **2023-03-23**

## Summary

OpenAI announces "initial support for plugins in ChatGPT": "tools designed specifically for language models with safety as a core principle". Plugins let [[chatgpt|ChatGPT]] reach up-to-date information, run computations and use third-party services.

The post calls plugins the model's "eyes and ears" for information that is "too recent, too personal, or too specific" to be in the training data.

## What shipped

- **Third-party plugins** from the first partners: Expedia, FiscalNote, Instacart, KAYAK, Klarna, Milo, OpenTable, Shopify, Slack, Speak, Wolfram and Zapier.
  - A developer builds an API, documents it with an **OpenAPI spec**, and writes a **manifest** file holding `name_for_model`, `description_for_model`, the API URL and the auth method.
  - The documentation of each enabled plugin is **inserted into the model's prompt**, and the model decides when to call it.
- **Two plugins hosted by OpenAI:**
  - **Browsing** (alpha): a text-only browser limited to GET requests, using the Bing API, with user-agent `ChatGPT-User` honouring robots.txt and citing its sources. The post traces it to [[webgpt|WebGPT]].
  - **Code interpreter** (alpha): Python in a sandboxed, firewalled environment with a persistent session and file upload and download.
- **An open-source retrieval plugin:** a self-hosted vector-database search over personal or organizational documents.
- **Rollout:** alpha access from a waitlist, starting with ChatGPT Plus users and a small set of developers.

## Safety section

- Red-teaming found that plugins released without safeguards could perform "sophisticated **[[prompt-injection|prompt injection]]**", send fraudulent or spam email, and bypass safety restrictions.
- OpenAI anticipated "open standards… to unify the ways in which applications expose an AI-facing interface" and called its plugin protocol "an early attempt at what such a standard might look like".

## Lineage

- **Built on:** [[webgpt|WebGPT]] (browsing), [[toolformer|Toolformer]]-style tool use, and the [[react|ReAct]] loop.
- **Fed into:**
  - OpenAI [[function-calling]] (June 2023), which moved tool definitions from a manifest into the API.
  - GPTs and Actions (November 2023, general knowledge), which replaced plugins. The plugin store was wound down in 2024 (general knowledge, not from this source).
  - The "AI-facing interface" standard the post predicted arrived from another lab as [[mcp|MCP]] (2024-11-25).
  - [[claude-code|Claude Code]] **plugins** (2025-10-09) reuse the name for a different idea: bundles of skills, hooks, commands and MCP servers for a coding agent. See [[claude-code-changelog-2026]].

## Concepts & entities

[[chatgpt-plugins]] · [[chatgpt]] · [[tool-use]] · [[function-calling]] · [[mcp]] · [[webgpt]] · [[prompt-injection]] · [[retrieval-augmented-generation]] · [[openai]]
