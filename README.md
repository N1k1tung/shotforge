# Shotforge Skill

Agent skill for operating Shotforge through MCP to create, revise, localize, and export App Store screenshot decks.

## Install

Install from the public GitHub repo with the Skills CLI:

```bash
npx skills add N1k1tung/shotforge --skill shotforge
```

For a global Codex install:

```bash
npx skills add N1k1tung/shotforge --skill shotforge -g -a codex -y
```

You can also install directly from the skill folder URL:

```bash
npx skills add https://github.com/N1k1tung/shotforge/tree/main/shotforge
```

From a local clone, verify discovery before publishing:

```bash
npx skills add . --list
```

## What It Covers

- Generate App Store screenshot decks from a brief and app screenshots.
- Import screenshots, icons, and supporting assets into Shotforge.
- Add or revise device families, localizations, and floating callouts.
- Render final PNG exports through Shotforge MCP.

## Requirements

- Shotforge for macOS.
- A working Shotforge MCP transport:
  - live GUI HTTP MCP at `http://127.0.0.1:32100/mcp`, or
  - headless stdio MCP launched from `Shotforge.app`.

This repository contains the agent skill only. It does not bundle the Shotforge app or configure MCP for the user.

## Repository Layout

```text
shotforge/
  SKILL.md
  agents/openai.yaml
```

`shotforge/SKILL.md` is the skill definition. `shotforge/agents/openai.yaml` provides optional UI metadata for OpenAI-compatible skill surfaces.
