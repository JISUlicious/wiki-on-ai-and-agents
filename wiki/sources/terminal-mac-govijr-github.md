---
title: "terminal-mac (GitHub repository, Govijr, 2026)"
type: source
created: 2026-11-04
updated: 2026-11-04
sources:
  - terminal-mac-govijr-github.md
  - terminal-mac-govijr-github-docs.md
authors:
  - Govijr
introduces:
  - [[terminal-mac]]
year: 2026
venue: "GitHub repository (MIT license)"
status: complete
importance: low
tags:
  - 2026
  - github
---

# terminal-mac (Govijr, 2026)

**[github.com/govijr/terminal-mac](https://github.com/govijr/terminal-mac)** by the GitHub user Govijr (profile company: Detach). Swift, MIT license. The repository was created on 2026-11-03 and had three releases that day (v0.2.0 builds 4–6); it had 452 stars when fetched on 2026-11-04. Ingested from the README plus `docs/AGENT.md`, `docs/VALIDATION.md` and `skills/terminal-mac/SKILL.md`. A sibling project, terminal-phone, does the same for iOS simulators and iPhones.

## Summary

[[terminal-mac]] gives a [[computer-use-agent]] **its own virtual Mac** instead of the user's desktop. It runs a real macOS guest under Apple's Virtualization framework on [[apple-silicon|Apple silicon]] (macOS 14+). A helper app **inside the guest** captures the guest screen with ScreenCaptureKit and posts input events there, so agent actions never move the host's pointer or touch its clipboard. The guest has its own apps, files, accounts and permissions; it does not inherit the host's signed-in sessions.

## How it works

| Part | What it does |
|---|---|
| Host CLI (`terminal-mac up`) | Downloads Apple's restore image, installs a VM with a sparse 64 GiB disk, boots it and checks the helper. Later runs reconnect. |
| Guest helper (`TerminalMacGuest.app`) | Opened once by the user from a read-only shared folder; installs itself, registers for login and needs **Screen Recording** and **Accessibility** granted *in the guest*. Refuses to run unless `hw.model` starts with `VirtualMac`. |
| Control channel | A file "mailbox" on a shared folder (request/reply JSON and screenshots), not a network listener. Rejects symlinks and hardlinks, bounds sizes, checks request IDs, expires pending actions. |
| Terminal viewer | Streams the guest screen into Ghostty, Kitty or iTerm2 using their image protocols: changed frames only, up to 30 fps, lz4-compressed, via shared memory for Kitty and Ghostty. Ctrl+G opens a palette of Mac shortcuts because terminals keep ⌘. |
| Agent interface | An [[mcp\|MCP]] server (`terminal-mac mcp`) with `screenshot`, `click`, `move`, `drag`, `scroll`, `type`, `key` and `status`; the same actions as a JSON CLI. Coordinates are guest screen points matching the latest screenshot. |

## Agent packaging

The project ships its agent instructions as a [[skill-md-format|SKILL.md]] skill that teaches an observe → act → verify loop: screenshot, act, screenshot again. It is distributed three ways:

- a [[claude-code]] plugin with the MCP server and skill together (`/plugin install terminal-mac --marketplace govijr/terminal-mac`);
- `terminal-mac skills install` or `--dir PATH` for any agent with a skills folder, or `npx skills add`;
- `terminal-mac mcp --config` output for other MCP clients (Cursor, VS Code, Codex, Windsurf).

The skill tells the agent when to stop and ask: a guest login screen, a permission prompt, a password, a two-factor code or a payment. It also forbids falling back to host screenshots or host input. There is no shell-execution action.

## Isolation model, as the README states it

- The guest sees only two shares: `control` (read/write) and `tools` (read-only). Host home, project folders, clipboard, microphone and camera are not attached.
- **Not a hardened sandbox.** Default NAT networking gives the guest internet access and may reach services on the host or local network. The README says it is not a network-isolated environment for hostile code; `--offline` removes the network device.
- The `VirtualMac` check prevents accidental host execution; it is "not hardware attestation".
- TCC permission grants are interactive guest setup, not bypassed.

## Caveats

- **No live guest test yet.** `VALIDATION.md` lists only build, signing and 14 automated protocol tests. No restore image was downloaded and no VM installed or booted; it says screen viewing, privacy grants and guest input should not be described as live-verified. The README also says "live guest acceptance on a clean Mac is still pending".
- **Very new and single-author.** All commits are from one author on 2026-11-03.
- **Missing pieces:** no VM delete, clone, rollback, accessibility-tree snapshot (`snapshot` is a screenshot alias), unattended guest setup or tmux transport.

> [!warning] Contradiction
> `docs/VALIDATION.md` (checks dated 2026-11-01, build 2) says both bundled apps have **Developer ID signatures** and Hardened Runtime, with notarization not run. The README and the 2026-11-03 commit "Sign releases with a self-signed identity" say releases are signed with a **self-signed** "Govijr Code Signing" certificate and not notarized. The later README reflects the current release; the validation file appears not to have been updated.

## How it relates to other wiki pages

- **[[computer-use-agent]]**: Anthropic's and OpenAI's computer-use releases ([[computer-use-claude-3-5-anthropic-2024]], [[operator]]) drive a screen supplied by the developer. terminal-mac is one way to supply a separate macOS screen locally, so the agent's clicks can't hit the user's own apps.
- **[[prompt-injection]]**: running the agent in a VM is a form of capability containment. It limits what an injected instruction can reach on the host, but the guest's internet access remains.
- **[[agent-skills]]**: an example of one tool shipping the same skill as a plugin, a CLI-installed folder and an `npx skills` package.

## References

- [Repository](https://github.com/govijr/terminal-mac) · [AGENT.md](https://github.com/govijr/terminal-mac/blob/main/docs/AGENT.md) · [VALIDATION.md](https://github.com/govijr/terminal-mac/blob/main/docs/VALIDATION.md)
- [Apple: running macOS in a VM on Apple silicon](https://developer.apple.com/documentation/virtualization/running-macos-in-a-virtual-machine-on-apple-silicon)
