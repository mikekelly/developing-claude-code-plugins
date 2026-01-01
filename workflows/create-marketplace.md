# Workflow: Create Plugin Marketplace

<required_reading>
**Read these reference files NOW:**
1. references/marketplace.md
2. references/plugin-structure.md
</required_reading>

<process>

## Step 1: Create Marketplace Directory

```bash
mkdir -p my-marketplace/.claude-plugin
mkdir -p my-marketplace/plugins
```

## Step 2: Create Marketplace Manifest

Create `my-marketplace/.claude-plugin/marketplace.json`:

```json
{
  "name": "my-marketplace",
  "owner": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "metadata": {
    "description": "Collection of plugins for [purpose]"
  },
  "plugins": []
}
```

**Required fields:**
- `name`: Marketplace identifier (kebab-case)
- `owner.name`: Maintainer name
- `plugins`: Array of plugin entries

**Reserved names (cannot use):**
- claude-code-marketplace, claude-code-plugins, claude-plugins-official
- anthropic-marketplace, anthropic-plugins, agent-skills, life-sciences

## Step 3: Add Plugins to Marketplace

### Option A: Local plugins (same repository)

Move/create plugins in `plugins/` directory:

```bash
mv my-plugin my-marketplace/plugins/
```

Add to marketplace.json:

```json
{
  "plugins": [
    {
      "name": "my-plugin",
      "source": "./plugins/my-plugin",
      "description": "What this plugin does"
    }
  ]
}
```

### Option B: GitHub repository plugins

```json
{
  "plugins": [
    {
      "name": "remote-plugin",
      "source": {
        "source": "github",
        "repo": "owner/plugin-repo"
      },
      "description": "Plugin from GitHub"
    }
  ]
}
```

### Option C: Git URL plugins

```json
{
  "plugins": [
    {
      "name": "git-plugin",
      "source": {
        "source": "url",
        "url": "https://gitlab.com/team/plugin.git"
      }
    }
  ]
}
```

## Step 4: Configure Plugin Entries

Full plugin entry with all options:

```json
{
  "name": "enterprise-tools",
  "source": "./plugins/enterprise-tools",
  "description": "Enterprise workflow automation",
  "version": "2.1.0",
  "author": {
    "name": "Enterprise Team"
  },
  "homepage": "https://docs.example.com/plugins",
  "repository": "https://github.com/company/plugin",
  "license": "MIT",
  "keywords": ["enterprise", "automation"],
  "category": "productivity",
  "strict": false
}
```

**Note on `strict`:**
- `true` (default): Plugin must have its own `plugin.json`
- `false`: Marketplace entry defines everything (no plugin.json needed)

## Step 5: Test Marketplace Locally

```bash
# Add marketplace
/plugin marketplace add ./my-marketplace

# Install a plugin
/plugin install plugin-name@my-marketplace

# Verify plugin works
/help
```

## Step 6: Host on GitHub

```bash
cd my-marketplace
git init
git add .
git commit -m "Initial marketplace"
git remote add origin https://github.com/you/my-marketplace.git
git push -u origin main
```

Users can then add with:
```
/plugin marketplace add you/my-marketplace
```

## Step 7: Configure Team Auto-Installation (Optional)

In your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "my-marketplace": {
      "source": {
        "source": "github",
        "repo": "you/my-marketplace"
      }
    }
  },
  "enabledPlugins": {
    "plugin-name@my-marketplace": true
  }
}
```

</process>

<anti_patterns>
Avoid:
- Using reserved marketplace names
- Relative paths that go outside plugin directory (`../`)
- Plugins that reference files outside their source path
- Missing `name` or `source` in plugin entries
</anti_patterns>

<success_criteria>
Marketplace is ready when:
- [ ] `marketplace.json` is valid JSON
- [ ] All plugin sources are accessible
- [ ] Local testing works: `claude plugin marketplace add ./my-marketplace`
- [ ] Plugins install correctly: `claude plugin install name@marketplace`
- [ ] Hosted on GitHub/GitLab for distribution
</success_criteria>
