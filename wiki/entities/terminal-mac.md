---
title: terminal-mac
type: entity
created: 2026-11-04
updated: 2026-11-04
sources:
  - terminal-mac-govijr-github.md
release_date: 2026-11-03
status: draft
importance: low
tags:
  - product
---

# terminal-mac

An open-source (MIT) Swift tool by Govijr that runs **a separate macOS virtual machine for [[computer-use-agent|computer-use agents]]** on [[apple-silicon|Apple silicon]], and shows its screen inside a terminal. First released 2026-11-03 as an early release candidate (v0.2.0). See [[terminal-mac-govijr-github]].

- **Separate desktop.** Apple's Virtualization framework runs a real macOS guest. A helper inside the guest takes screenshots and posts input, so the agent never controls the user's own pointer, focus or clipboard.
- **Agent interface.** An [[mcp|MCP]] server with screenshot, click, move, drag, scroll, type, key and status tools, plus a [[skill-md-format|SKILL.md]] skill, also packaged as a [[claude-code]] plugin.
- **Terminal viewer.** Streams changed frames to Ghostty, Kitty or iTerm2.
- **Limits.** Not a hardened sandbox: the guest has internet access by default. First-time setup needs the user (macOS Setup Assistant and two guest permissions). As of 2026-11-04 no live guest run has been recorded in its validation notes.

Sibling project: terminal-phone, for iOS simulators and iPhones.

## Related

- [[computer-use-agent]] · [[mcp]] · [[agent-skills]] · [[claude-code]] · [[apple-silicon]] · [[prompt-injection]]

## References

- [[terminal-mac-govijr-github]]
