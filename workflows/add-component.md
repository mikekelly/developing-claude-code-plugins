# Workflow: Add Component to Plugin

<required_reading>
**Read the reference for the component type:**
- Commands: references/commands.md
- Agents: references/agents.md
- Skills: references/skills.md
- Hooks: references/hooks.md
- MCP/LSP: references/mcp-lsp.md
</required_reading>

<process>

## Step 1: Identify Component Type

Ask user which component to add:
1. **Command** - Slash command (`/plugin:command-name`)
2. **Agent** - Subagent for specialized tasks
3. **Skill** - Model-invoked capability
4. **Hook** - Event handler (PreToolUse, PostToolUse, etc.)
5. **MCP Server** - External tool integration
6. **LSP Server** - Code intelligence for a language

## Step 2: Add the Component

### Adding a Command

Create `commands/command-name.md`:

```markdown
---
description: Brief description shown in /help
allowed-tools: Read, Bash(git:*)
argument-hint: [arg1] [arg2]
---

# Command Name

Instructions for Claude when this command runs.
Use $ARGUMENTS for all args or $1, $2 for positional.
```

### Adding an Agent

Create `agents/agent-name.md`:

```markdown
---
name: agent-name
description: When to use this agent
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a specialist in [domain].

When invoked:
1. First action
2. Second action
3. Third action
```

### Adding a Skill

Create `skills/skill-name/SKILL.md`:

```yaml
---
name: skill-name
description: What this skill does. Use when [trigger conditions].
allowed-tools: Read, Grep, Glob
---

<objective>
What this skill accomplishes.
</objective>

<quick_start>
Immediate guidance for using this skill.
</quick_start>

<success_criteria>
How to know the skill worked.
</success_criteria>
```

### Adding Hooks

Create `hooks/hooks.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
          }
        ]
      }
    ]
  }
}
```

**Available events:** PreToolUse, PostToolUse, PostToolUseFailure, PermissionRequest, UserPromptSubmit, Notification, Stop, SubagentStop, SessionStart, SessionEnd, PreCompact

### Adding MCP Server

Create `.mcp.json` at plugin root:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/server-binary",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

### Adding LSP Server

Create `.lsp.json` at plugin root:

```json
{
  "language-name": {
    "command": "language-server-binary",
    "args": ["serve"],
    "extensionToLanguage": {
      ".ext": "language"
    }
  }
}
```

## Step 3: Test the Component

```bash
claude --plugin-dir ./my-plugin
```

Verify:
- Commands: Check `/help` for listing
- Agents: Check `/agents` for listing
- Skills: Ask "What skills are available?"
- Hooks: Trigger the event and check behavior
- MCP: Check `/mcp` for server status
- LSP: Open a file of that language type

</process>

<success_criteria>
Component is added when:
- [ ] File is in correct location (plugin root, not .claude-plugin/)
- [ ] Valid syntax (YAML frontmatter, JSON)
- [ ] Plugin still validates: `claude plugin validate ./my-plugin`
- [ ] Component appears in Claude Code
- [ ] Component functions correctly when triggered
</success_criteria>
