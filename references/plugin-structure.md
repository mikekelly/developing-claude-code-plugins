<overview>
Complete reference for Claude Code plugin directory structure, manifest schema, and file organization. Plugins bundle commands, agents, skills, hooks, MCP servers, and LSP servers into distributable packages.
</overview>

<directory_structure>
## Plugin Directory Layout

```
my-plugin/
├── .claude-plugin/           # Metadata directory (required)
│   └── plugin.json          # Plugin manifest (required)
├── commands/                 # Slash commands
│   ├── command-name.md
│   └── subdir/
│       └── nested-command.md
├── agents/                   # Subagent definitions
│   └── agent-name.md
├── skills/                   # Agent Skills
│   └── skill-name/
│       └── SKILL.md
├── hooks/                    # Hook configurations
│   └── hooks.json
├── .mcp.json                # MCP server definitions
├── .lsp.json                # LSP server configurations
├── scripts/                 # Utility scripts for hooks
│   └── helper.sh
├── README.md                # Documentation
└── CHANGELOG.md             # Version history
```

**CRITICAL**: Only `plugin.json` goes in `.claude-plugin/`. All other directories (commands/, agents/, skills/, hooks/) must be at the plugin root.
</directory_structure>

<manifest_schema>
## plugin.json Schema

### Required Fields

```json
{
  "name": "plugin-name"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Unique identifier (kebab-case, no spaces). Becomes slash command namespace. |

### Optional Metadata

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Brief plugin description",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://github.com/author"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/author/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"]
}
```

### Component Path Fields

Override default component locations:

```json
{
  "commands": ["./custom/commands/", "./extra-cmd.md"],
  "agents": ["./custom/agents/"],
  "skills": ["./custom/skills/"],
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "lspServers": "./.lsp.json"
}
```

**Note**: Custom paths supplement defaults - they don't replace them. If `commands/` exists, it's loaded in addition to custom paths.

### Inline Configurations

Hooks, MCP, and LSP can be defined inline in plugin.json:

```json
{
  "name": "my-plugin",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{"type": "command", "command": "npm run lint"}]
      }
    ]
  },
  "mcpServers": {
    "my-server": {
      "command": "${CLAUDE_PLUGIN_ROOT}/server",
      "args": []
    }
  },
  "lspServers": {
    "go": {
      "command": "gopls",
      "args": ["serve"],
      "extensionToLanguage": {".go": "go"}
    }
  }
}
```
</manifest_schema>

<environment_variables>
## Environment Variables

### ${CLAUDE_PLUGIN_ROOT}

Absolute path to the plugin's installation directory. **Required** for all paths in hooks, MCP servers, and scripts.

```json
{
  "hooks": {
    "PostToolUse": [{
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
      }]
    }]
  }
}
```

**Why needed**: Plugins are copied to a cache directory when installed. Relative paths like `./scripts/format.sh` won't work.

### ${CLAUDE_PROJECT_DIR}

Project root directory (where Claude Code was started). Available in hook scripts.
</environment_variables>

<installation_scopes>
## Installation Scopes

| Scope | Settings File | Use Case |
|-------|---------------|----------|
| `user` | `~/.claude/settings.json` | Personal plugins (default) |
| `project` | `.claude/settings.json` | Team plugins via version control |
| `local` | `.claude/settings.local.json` | Project-specific, gitignored |
| `managed` | `managed-settings.json` | Enterprise-managed (read-only) |

Install with scope:
```bash
claude plugin install plugin-name@marketplace --scope project
```
</installation_scopes>

<version_management>
## Version Management

Follow semantic versioning: `MAJOR.MINOR.PATCH`

- **MAJOR**: Breaking changes (incompatible API changes)
- **MINOR**: New features (backward-compatible additions)
- **PATCH**: Bug fixes (backward-compatible fixes)

Pre-release versions: `2.0.0-beta.1`

Update version in `plugin.json` before distributing changes. Document changes in `CHANGELOG.md`.
</version_management>

<caching_behavior>
## Plugin Caching

When installed, plugins are copied to `~/.claude/plugins/cache/`. This means:

- Files outside the plugin directory are NOT copied
- Paths like `../shared-utils` won't work
- Use symlinks if you need external files (symlinks are followed during copy)

To clear cache and reinstall:
```bash
rm -rf ~/.claude/plugins/cache
# Then reinstall plugins
```
</caching_behavior>
