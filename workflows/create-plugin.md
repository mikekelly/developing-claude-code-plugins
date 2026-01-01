# Workflow: Create New Plugin

<required_reading>
**Read these reference files NOW:**
1. references/plugin-structure.md
2. references/commands.md (if adding commands)
</required_reading>

<process>

## Step 1: Create Plugin Directory Structure

```bash
mkdir -p my-plugin/.claude-plugin
mkdir -p my-plugin/commands
```

## Step 2: Create Plugin Manifest

Create `my-plugin/.claude-plugin/plugin.json`:

```json
{
  "name": "my-plugin",
  "description": "Brief description of what this plugin does",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

**Required fields:**
- `name`: Unique identifier (kebab-case). Becomes the command namespace.

**Optional fields:**
- `version`: Semantic versioning (MAJOR.MINOR.PATCH)
- `description`: Shown in plugin manager
- `author`: Attribution
- `homepage`: Documentation URL
- `repository`: Source code URL
- `license`: SPDX identifier (MIT, Apache-2.0, etc.)

## Step 3: Add Your First Command

Create `my-plugin/commands/hello.md`:

```markdown
---
description: Greet the user with a friendly message
---

# Hello Command

Greet the user warmly and ask how you can help them today.
```

The filename (without .md) becomes the command name: `/my-plugin:hello`

## Step 4: Test the Plugin

```bash
claude --plugin-dir ./my-plugin
```

Then in Claude Code:
```
/my-plugin:hello
```

Verify the command appears in `/help` under the plugin namespace.

## Step 5: Add More Components (Optional)

Based on your needs, add:
- More commands: `commands/*.md`
- Agents: `agents/*.md`
- Skills: `skills/skill-name/SKILL.md`
- Hooks: `hooks/hooks.json`
- MCP servers: `.mcp.json`
- LSP servers: `.lsp.json`

See `workflows/add-component.md` for each component type.

</process>

<anti_patterns>
Avoid:
- Putting commands inside `.claude-plugin/` (they go at plugin root)
- Using spaces or uppercase in plugin name
- Hardcoding paths (use `${CLAUDE_PLUGIN_ROOT}`)
- Forgetting to restart Claude Code after changes
</anti_patterns>

<success_criteria>
Plugin is complete when:
- [ ] `.claude-plugin/plugin.json` exists with valid name
- [ ] At least one component (command, agent, skill, hook, MCP, or LSP)
- [ ] `claude plugin validate ./my-plugin` passes
- [ ] Plugin loads with `claude --plugin-dir ./my-plugin`
- [ ] Components appear in Claude Code (/help, /agents, etc.)
</success_criteria>
