<overview>
Reference for creating and hosting Claude Code plugin marketplaces. Marketplaces are catalogs that enable plugin distribution to teams and communities.
</overview>

<marketplace_structure>
## Marketplace Directory Structure

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json    # Marketplace manifest (required)
└── plugins/                # Local plugin directories
    ├── plugin-one/
    │   ├── .claude-plugin/
    │   │   └── plugin.json
    │   └── commands/
    └── plugin-two/
        └── ...
```
</marketplace_structure>

<marketplace_schema>
## marketplace.json Schema

### Required Fields

```json
{
  "name": "marketplace-name",
  "owner": {
    "name": "Maintainer Name"
  },
  "plugins": []
}
```

| Field | Description |
|-------|-------------|
| `name` | Unique identifier (kebab-case). Users see this when installing. |
| `owner.name` | Maintainer name |
| `plugins` | Array of plugin entries |

### Reserved Names (Cannot Use)

- `claude-code-marketplace`
- `claude-code-plugins`
- `claude-plugins-official`
- `anthropic-marketplace`
- `anthropic-plugins`
- `agent-skills`
- `life-sciences`

### Optional Metadata

```json
{
  "name": "my-marketplace",
  "owner": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "metadata": {
    "description": "Collection of plugins for X",
    "version": "1.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [...]
}
```

`pluginRoot` is prepended to relative source paths.
</marketplace_schema>

<plugin_entries>
## Plugin Entry Schema

### Required Fields

```json
{
  "name": "plugin-name",
  "source": "./plugins/plugin-name"
}
```

### Optional Fields

```json
{
  "name": "enterprise-tools",
  "source": "./plugins/enterprise-tools",
  "description": "Enterprise workflow automation",
  "version": "2.1.0",
  "author": {
    "name": "Enterprise Team",
    "email": "team@example.com"
  },
  "homepage": "https://docs.example.com/plugins",
  "repository": "https://github.com/company/plugin",
  "license": "MIT",
  "keywords": ["enterprise", "automation"],
  "category": "productivity",
  "tags": ["workflow", "automation"],
  "strict": false
}
```

### strict Field

| Value | Behavior |
|-------|----------|
| `true` (default) | Plugin must have its own plugin.json |
| `false` | Marketplace entry defines everything |
</plugin_entries>

<plugin_sources>
## Plugin Sources

### Local Path (Same Repository)

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin"
}
```

### GitHub Repository

```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo"
  }
}
```

### Git URL

```json
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git"
  }
}
```

### GitHub with Specific Path/Ref

```json
{
  "name": "nested-plugin",
  "source": {
    "source": "github",
    "repo": "owner/monorepo",
    "path": "packages/plugin",
    "ref": "v2.0"
  }
}
```
</plugin_sources>

<inline_components>
## Inline Component Configuration

Define commands, agents, hooks directly in marketplace entry:

```json
{
  "name": "enterprise-tools",
  "source": "./plugins/enterprise-tools",
  "commands": ["./commands/core/", "./commands/extra.md"],
  "agents": ["./agents/reviewer.md"],
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh"
      }]
    }]
  },
  "mcpServers": {
    "db": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db",
      "args": []
    }
  },
  "strict": false
}
```
</inline_components>

<hosting>
## Hosting Marketplaces

### GitHub (Recommended)

```bash
cd my-marketplace
git init
git add .
git commit -m "Initial marketplace"
git remote add origin https://github.com/you/my-marketplace.git
git push -u origin main
```

Users add with:
```
/plugin marketplace add you/my-marketplace
```

### Other Git Services

```
/plugin marketplace add https://gitlab.com/company/plugins.git
```

### Local Testing

```
/plugin marketplace add ./my-marketplace
```
</hosting>

<team_configuration>
## Team Auto-Installation

In project `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "company/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "formatter@company-tools": true,
    "linter@company-tools": true
  }
}
```

Team members are prompted to install marketplace when they trust the project.
</team_configuration>

<enterprise_restrictions>
## Enterprise Marketplace Restrictions

In managed settings, use `strictKnownMarketplaces`:

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    }
  ]
}
```

| Value | Effect |
|-------|--------|
| Undefined | No restrictions |
| `[]` | Complete lockdown |
| List | Only listed marketplaces allowed |
</enterprise_restrictions>

<validation>
## Validation

```bash
# Validate marketplace
claude plugin validate ./my-marketplace

# Or from Claude Code
/plugin validate ./my-marketplace
```

Common errors:
- `Duplicate plugin name` - Each plugin needs unique name
- `Path traversal not allowed` - No `..` in source paths
- `Invalid JSON syntax` - Check commas, quotes
</validation>

<caching_notes>
## Caching Behavior

Plugins are copied to cache on install. This means:
- Files outside plugin source are NOT copied
- Use symlinks if you need shared files
- Symlinks are followed during copy

To force refresh:
```bash
rm -rf ~/.claude/plugins/cache
# Then reinstall
```
</caching_notes>

<cli_commands>
## CLI Commands

```bash
# Add marketplace
claude plugin marketplace add owner/repo
claude plugin marketplace add ./local-path

# Install plugin
claude plugin install plugin-name@marketplace-name

# Update plugin
claude plugin update plugin-name@marketplace-name

# Uninstall
claude plugin uninstall plugin-name@marketplace-name

# Enable/disable
claude plugin enable plugin-name@marketplace-name
claude plugin disable plugin-name@marketplace-name
```
</cli_commands>
