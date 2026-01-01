<overview>
Troubleshooting guide for common Claude Code plugin issues. Covers manifest errors, component loading failures, hooks, MCP/LSP problems, and marketplace issues.
</overview>

<validation_errors>
## Manifest Validation Errors

Run validation:
```bash
claude plugin validate ./my-plugin
# or
/plugin validate ./my-plugin
```

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `Invalid JSON syntax: Unexpected token` | Missing/extra comma, unquoted string | Validate JSON: `cat plugin.json \| python -m json.tool` |
| `name: Required` | Missing name field | Add `"name": "plugin-name"` |
| `Plugin has a corrupt manifest` | JSON parse error | Check for syntax errors |

### JSON Validation

```bash
# Check JSON syntax
cat .claude-plugin/plugin.json | python -m json.tool

# Or use jq
jq . .claude-plugin/plugin.json
```
</validation_errors>

<plugin_not_loading>
## Plugin Not Loading

### Debug Mode

```bash
claude --debug --plugin-dir ./my-plugin
```

Look for:
- "loading plugin" messages
- Manifest errors
- Component registration status

### Check Structure

```bash
# Verify plugin.json location
ls -la my-plugin/.claude-plugin/
# Should show: plugin.json

# Verify components at root
ls -la my-plugin/
# Should show: commands/, agents/, etc.
```

### Common Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Plugin not found | Wrong path | Check path exists |
| No components | Wrong location | Move from .claude-plugin/ to root |
| Still not loading | Cache issue | Restart Claude Code |
</plugin_not_loading>

<commands_not_appearing>
## Commands Not Appearing

### Check Location

Commands must be at plugin root, NOT in .claude-plugin/:

```
✅ Correct:
my-plugin/
├── .claude-plugin/plugin.json
└── commands/hello.md

❌ Wrong:
my-plugin/
└── .claude-plugin/
    ├── plugin.json
    └── commands/hello.md
```

### Verify Files

```bash
# List command files
ls my-plugin/commands/*.md

# Check file has frontmatter
head -10 my-plugin/commands/hello.md
```

### Namespace Check

Commands appear as `/plugin-name:command-name`. Check `/help` for the plugin section.
</commands_not_appearing>

<hooks_not_firing>
## Hooks Not Firing

### Script Permissions

```bash
# Make executable
chmod +x my-plugin/scripts/*.sh
chmod +x my-plugin/scripts/*.py
```

### Shebang Line

Check first line of script:
```bash
head -1 my-plugin/scripts/format.sh
# Should be: #!/bin/bash or #!/usr/bin/env bash
```

### Path Check

Use `${CLAUDE_PLUGIN_ROOT}`:
```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
}
```

NOT:
```json
{
  "command": "./scripts/format.sh"
}
```

### Event/Matcher Check

- Event names are case-sensitive: `PostToolUse` not `postToolUse`
- Matcher must match tool: `Write|Edit` for file operations

### Test Script Manually

```bash
cd my-plugin
echo '{"tool_name": "Write"}' | ./scripts/format.sh
echo $?  # Check exit code
```
</hooks_not_firing>

<mcp_issues>
## MCP Server Issues

### Server Not Starting

1. Check command exists:
```bash
which server-binary
# or
ls -la ${CLAUDE_PLUGIN_ROOT}/servers/
```

2. Check paths use `${CLAUDE_PLUGIN_ROOT}`

3. Test server manually:
```bash
./servers/server-binary --help
```

### Tools Not Appearing

1. Check `/mcp` for server status
2. Verify MCP protocol implementation
3. Check debug output for connection errors

### Connection Issues

Run with debug:
```bash
claude --debug
```

Look for MCP initialization errors.
</mcp_issues>

<lsp_issues>
## LSP Server Issues

### "Executable not found in $PATH"

Install the language server first:

```bash
# Python
pip install pyright
# or
npm install -g pyright

# TypeScript
npm install -g typescript-language-server typescript

# Go
go install golang.org/x/tools/gopls@latest
```

Verify in PATH:
```bash
which pyright-langserver
which typescript-language-server
which gopls
```

### Not Providing Intelligence

1. Open a file of that language type
2. Check `/plugin` Errors tab
3. Verify `extensionToLanguage` mapping

### Debug Logging

Add logging config:
```json
{
  "loggingConfig": {
    "args": ["--log-level", "4"]
  }
}
```

Run with `--enable-lsp-logging`. Check `~/.claude/debug/`.
</lsp_issues>

<marketplace_issues>
## Marketplace Issues

### Can't Add Marketplace

- Verify URL is accessible
- Check `.claude-plugin/marketplace.json` exists
- Validate JSON syntax

### Plugins Not Appearing

- Check plugin `source` paths exist
- Verify each plugin has valid structure
- Look for validation errors

### Installation Failures

- Check plugin source is accessible
- For GitHub: verify repo is public or you have access
- For local: check paths don't use `../`
</marketplace_issues>

<path_issues>
## Path-Related Issues

### Files Not Found After Installation

Plugins are copied to cache. External files aren't copied.

```
✅ Works:
"command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"

❌ Doesn't work:
"command": "../shared/format.sh"
```

Solutions:
1. Keep all files in plugin directory
2. Use symlinks (followed during copy)
3. Restructure to avoid external references

### Clear Cache

```bash
rm -rf ~/.claude/plugins/cache
# Then reinstall plugins
```
</path_issues>

<debugging_checklist>
## Quick Debugging Checklist

- [ ] `plugin.json` is valid JSON
- [ ] `name` field present and kebab-case
- [ ] Components at plugin root (not in .claude-plugin/)
- [ ] Scripts executable (`chmod +x`)
- [ ] Paths use `${CLAUDE_PLUGIN_ROOT}`
- [ ] LSP binaries installed and in PATH
- [ ] Restarted Claude Code after changes
- [ ] Ran `claude plugin validate ./my-plugin`

### Full Debug Session

```bash
# 1. Validate
claude plugin validate ./my-plugin

# 2. Check structure
ls -laR my-plugin/

# 3. Test with debug
claude --debug --plugin-dir ./my-plugin

# 4. Look for errors in output
# 5. Check /help, /agents, /mcp for components
```
</debugging_checklist>

<common_mistakes>
## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Components in .claude-plugin/ | Not loading | Move to plugin root |
| Relative paths | File not found | Use `${CLAUDE_PLUGIN_ROOT}` |
| Non-executable scripts | Hook doesn't run | `chmod +x` |
| Missing shebang | Script errors | Add `#!/bin/bash` |
| Case-sensitive event names | Hook doesn't fire | Use exact case: `PostToolUse` |
| Uppercase in plugin name | Validation fails | Use lowercase-kebab-case |
| Forgot to restart | Changes not applied | Restart Claude Code |
| LSP binary not installed | No code intelligence | Install language server |
</common_mistakes>
