# Shotforge Skill

Agent skill for operating Shotforge through MCP to create, revise, localize, and export App Store screenshot decks.

## Install

Install from the public GitHub repo with the Skills CLI:

```bash
npx skills add N1k1tung/shotforge --skill shotforge
```

`npx skills` handles popular agent tools directly. Agents that read shared project skills can also use `.agents/skills/shotforge`.

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
- The live Shotforge HTTP MCP server enabled at `http://127.0.0.1:32100/mcp`.

This repository contains the agent skill only. It does not bundle the Shotforge app or configure MCP for the user.

## MCP Setup

After installing the skill, open Shotforge and enable the embedded MCP server in `Settings > MCP`. The default address is:

```text
http://127.0.0.1:32100/mcp
```

Skill installation and MCP server registration are separate. Use the setup command or config for your agent.

### Codex

```bash
codex mcp add shotforge --url http://127.0.0.1:32100/mcp
```

### Claude Code

```bash
claude mcp add --transport http shotforge http://127.0.0.1:32100/mcp
```

### OpenCode

```bash
opencode mcp add
```

Choose a remote MCP server, name it `shotforge`, and use `http://127.0.0.1:32100/mcp`.

Or add this to `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "shotforge": {
      "type": "remote",
      "url": "http://127.0.0.1:32100/mcp",
      "enabled": true
    }
  }
}
```

### Cursor

```bash
mkdir -p ~/.cursor
$EDITOR ~/.cursor/mcp.json
```

Add or merge:

```json
{
  "mcpServers": {
    "shotforge": {
      "url": "http://127.0.0.1:32100/mcp"
    }
  }
}
```

### Google Antigravity

```bash
mkdir -p ~/.gemini/antigravity
$EDITOR ~/.gemini/antigravity/mcp_config.json
```

Add or merge:

```json
{
  "mcpServers": {
    "shotforge": {
      "serverUrl": "http://127.0.0.1:32100/mcp"
    }
  }
}
```

### GitHub Copilot CLI

In Copilot CLI interactive mode:

```text
/mcp add
```

Use:

```text
Server Name: shotforge
Server Type: HTTP
URL: http://127.0.0.1:32100/mcp
Tools: *
```

Or add this to `~/.copilot/mcp-config.json`:

```json
{
  "mcpServers": {
    "shotforge": {
      "type": "http",
      "url": "http://127.0.0.1:32100/mcp",
      "headers": {},
      "tools": ["*"]
    }
  }
}
```

### Pi

```bash
mkdir -p ~/.pi/agent
$EDITOR ~/.pi/agent/mcp.json
```

Add or merge:

```json
{
  "mcpServers": {
    "shotforge": {
      "type": "http",
      "url": "http://127.0.0.1:32100/mcp"
    }
  }
}
```

## Repository Layout

```text
shotforge/
  SKILL.md
  agents/openai.yaml
```

`shotforge/SKILL.md` is the skill definition. `shotforge/agents/openai.yaml` provides optional UI metadata for OpenAI-compatible skill surfaces.
