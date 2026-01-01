# Workflow: Test and Debug Plugin

<required_reading>
**Read these reference files NOW:**
1. references/troubleshooting.md
2. references/plugin-structure.md
</required_reading>

<process>

## Step 1: Validate Plugin Structure

```bash
claude plugin validate ./my-plugin
```

Common validation errors:
- `Invalid JSON syntax` → Check for missing/extra commas, unquoted strings
- `name: Required` → Add `name` field to plugin.json
- `No commands found` → Commands in wrong directory

## Step 2: Test Plugin Loading

```bash
claude --debug --plugin-dir ./my-plugin
```

Debug output shows:
- Which plugins are loading
- Any manifest errors
- Component registration status
- MCP server initialization

## Step 3: Check Component Registration

Once Claude Code starts:

**Commands:**
```
/help
```
Look for `/my-plugin:command-name` in the list.

**Agents:**
```
/agents
```
Check plugin agents appear in the list.

**Skills:**
```
What skills are available?
```
Plugin skills should be listed.

**MCP Servers:**
```
/mcp
```
Check server connection status.

**Hooks:**
Trigger the event (e.g., write a file for PostToolUse) and observe behavior.

## Step 4: Diagnose Common Issues

### Plugin Not Loading

1. Check structure:
```bash
ls -la my-plugin/.claude-plugin/
# Should show: plugin.json
```

2. Validate manifest:
```bash
cat my-plugin/.claude-plugin/plugin.json | python -m json.tool
```

### Commands Not Appearing

1. Verify location (must be at plugin root):
```bash
ls my-plugin/commands/
# Should show: *.md files
```

2. Check filename matches expected command name

### Hooks Not Firing

1. Check script is executable:
```bash
chmod +x my-plugin/scripts/*.sh
```

2. Verify shebang line:
```bash
head -1 my-plugin/scripts/script.sh
# Should be: #!/bin/bash or #!/usr/bin/env bash
```

3. Check path uses `${CLAUDE_PLUGIN_ROOT}`:
```json
"command": "${CLAUDE_PLUGIN_ROOT}/scripts/script.sh"
```

### MCP Server Not Starting

1. Check command exists and is executable
2. Verify all paths use `${CLAUDE_PLUGIN_ROOT}`
3. Test server manually outside Claude Code

### LSP Not Working

1. Install language server binary first
2. Check it's in PATH:
```bash
which language-server-binary
```

## Step 5: Test Multiple Plugins

Load multiple plugins to check for conflicts:

```bash
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
```

</process>

<debugging_checklist>
- [ ] `plugin.json` is valid JSON
- [ ] `name` field present and kebab-case
- [ ] Components at plugin root (not in .claude-plugin/)
- [ ] Scripts are executable (`chmod +x`)
- [ ] Paths use `${CLAUDE_PLUGIN_ROOT}`
- [ ] Restart Claude Code after changes
</debugging_checklist>

<success_criteria>
Plugin is working when:
- [ ] `claude plugin validate` passes
- [ ] Plugin loads without errors in debug mode
- [ ] All components appear in their interfaces
- [ ] Components function correctly when triggered
- [ ] No conflicts with other plugins
</success_criteria>
