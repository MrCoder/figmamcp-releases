# FigmaMCP

**Let Claude Code or Codex read and edit the Figma file you already have open.** A free Mac app with a local MCP server and a Figma plugin. No Figma API token, and no call quota to watch.

![FigmaMCP: an AI agent edits a live Figma file](https://figmamcp.dev/assets/figmamcp-agent-editing.webp)

**[Download for Mac](https://figmamcp.dev/download)** · [Website](https://figmamcp.dev) · [Setup guide](https://figmamcp.dev/docs/setup)

## Install in 3 steps

1. **Download and open the app** from [figmamcp.dev/download](https://figmamcp.dev/download). It is notarized for macOS and checks for Figma, Claude Code and Codex.
2. **Click through the checklist.** The app adds the plugin to the Figma desktop app and connects the MCP server to your AI. There is no manifest to import and no config file to edit.
3. **Ask your AI.** Open a Figma file and ask for a change. The AI reads the file first, then edits it in place while you watch.

FigmaMCP works through Figma's Plugin API on your own machine. It makes no calls to Figma's hosted MCP server.

## What it can do

- Read pages, layers, styles, variables, components, annotations and prototype reactions.
- Export a layer as an image so the AI can check its own work.
- Create frames, text, shapes, SVG, components and instances.
- Edit fills, strokes, effects, corner radius, auto layout, text and layer structure.
- Manage your design system: styles, variables, variable bindings, component properties and instance overrides.

## AI skills

Skills are short instruction files that teach an AI agent how to do one Figma task well with FigmaMCP. They are in [`skills/`](skills/):

| Skill | What it does |
|---|---|
| [`figma-spec-to-frames`](skills/figma-spec-to-frames/SKILL.md) | Builds a screen from a written spec as auto-layout frames with named layers, then exports a PNG to check it. |
| [`figma-rename-layers`](skills/figma-rename-layers/SKILL.md) | Renames a frame's layers clearly and consistently, after you approve the list. |
| [`figma-handoff-notes`](skills/figma-handoff-notes/SKILL.md) | Reads a frame's layout, type, colors, styles and tokens, and writes Figma annotations plus a handoff summary. |
| [`figma-design-qa`](skills/figma-design-qa/SKILL.md) | Scores a frame against a written rubric, fixes the weakest categories, and repeats until it reaches 9 out of 10 or the round limit. |
| [`figmamcp`](skills/figmamcp/SKILL.md) | The base skill: the plugin's safety rules, error codes and how to pick a tool. |

**Install:** copy a skill's folder into your agent's skills folder, then start a new agent session.

- Claude Code: `~/.claude/skills/<skill-name>/` (for one project only: `.claude/skills/<skill-name>/` in that project)
- Codex: `~/.codex/skills/<skill-name>/`

The FigmaMCP app already installs and updates the `figmamcp` skill for Claude Code and Codex, so you only need to copy the task skills. Each task skill needs FigmaMCP running with the plugin open in your file.

## Supported agents

| Agent | Status |
|---|---|
| Claude Code | Tested |
| Codex | Tested |
| Cursor | Works with MCP; not yet tested by us |

## Safety

- Every edit must name its exact target layer.
- A bulk edit with one invalid item changes nothing.
- A switch in the plugin makes the file read-only for the AI.
- Two AI sessions can work in one file; one cannot overwrite a layer the other is editing.

## Requirements

- macOS (Windows and Linux are not supported yet)
- The Figma desktop app
- Claude Code or Codex

## Privacy

FigmaMCP itself sends no design content anywhere. The plugin sends only a random install ID and anonymous tool-call counts; no file names or design content. Your AI agent does send what it reads from your file to its own model provider, under that provider's terms, so choose your agent with that in mind. Details: [figmamcp.dev/privacy](https://figmamcp.dev/privacy).

## Report a problem

[Open an issue](https://github.com/MrCoder/figmamcp-releases/issues/new/choose), or write to support@figmamcp.dev. Do not attach private design files or secrets.

## About this repository

This repository holds downloads, issue reports and the AI skills in `skills/`. It has no source code. Downloads are on the [Releases page](https://github.com/MrCoder/figmamcp-releases/releases).

FigmaMCP is a fork of [figma-edit-mcp](https://github.com/neozhehan/figma-edit-mcp) by Neo Product LLC. It includes third-party software; see [THIRD_PARTY_NOTICES.txt](THIRD_PARTY_NOTICES.txt). Not affiliated with or endorsed by Figma, Inc.
