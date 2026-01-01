# developing-claude-code-plugins

A Claude Code skill for building, testing, and distributing Claude Code plugins.

## What This Skill Does

This skill provides comprehensive guidance for the full plugin development lifecycle:

- **Create plugins** with slash commands, agents, skills, hooks, MCP servers, and LSP servers
- **Add components** to existing plugins
- **Test and debug** plugins during development
- **Create marketplaces** for distributing plugins to teams and communities

## Installation

This skill is installed as a personal skill in `~/.claude/skills/developing-claude-code-plugins/`.

## Usage

The skill is automatically invoked when you mention plugins, extensions, marketplaces, or want to add custom commands/agents to Claude Code.

Example prompts:
- "Create a new Claude Code plugin"
- "Add a slash command to my plugin"
- "How do I distribute my plugin?"
- "My plugin hooks aren't working"

## Structure

```
developing-claude-code-plugins/
├── SKILL.md              # Main skill file with router
├── workflows/            # Step-by-step procedures
│   ├── create-plugin.md
│   ├── add-component.md
│   ├── test-debug-plugin.md
│   └── create-marketplace.md
└── references/           # Domain knowledge
    ├── plugin-structure.md
    ├── commands.md
    ├── agents.md
    ├── skills.md
    ├── hooks.md
    ├── mcp-lsp.md
    ├── marketplace.md
    └── troubleshooting.md
```

## Source Documentation

Based on official Claude Code documentation:
- https://code.claude.com/docs/en/plugins.md
- https://code.claude.com/docs/en/plugins-reference.md
- https://code.claude.com/docs/en/skills.md
- https://code.claude.com/docs/en/hooks.md
- https://code.claude.com/docs/en/slash-commands.md
- https://code.claude.com/docs/en/sub-agents.md
- https://code.claude.com/docs/en/plugin-marketplaces.md
