<overview>
Reference for configuring MCP (Model Context Protocol) servers and LSP (Language Server Protocol) servers in Claude Code plugins. MCP provides external tool integration; LSP provides code intelligence.
</overview>

<mcp_basics>
## MCP Server Configuration

Location: `.mcp.json` at plugin root (or inline in plugin.json)

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

MCP servers provide external tools Claude can use (databases, APIs, etc.).
</mcp_basics>

<mcp_fields>
## MCP Server Fields

| Field | Required | Description |
|-------|----------|-------------|
| `command` | Yes | Server binary/script to execute |
| `args` | No | Command-line arguments |
| `env` | No | Environment variables |
| `cwd` | No | Working directory |

### Example with All Options

```json
{
  "mcpServers": {
    "database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--port", "5432", "--config", "${CLAUDE_PLUGIN_ROOT}/db-config.json"],
      "env": {
        "DB_HOST": "localhost",
        "DB_USER": "${DB_USER}",
        "DB_PASS": "${DB_PASS}"
      },
      "cwd": "${CLAUDE_PLUGIN_ROOT}"
    }
  }
}
```
</mcp_fields>

<mcp_tool_naming>
## MCP Tool Naming

MCP tools appear as: `mcp__<server-name>__<tool-name>`

Examples:
- `mcp__database__query`
- `mcp__github__search_repositories`
- `mcp__memory__create_entities`

### Permission Rules

Approve all tools from a server:
- `mcp__github` (server name alone)
- `mcp__github__*` (wildcard)

Approve specific tools:
- `mcp__github__get_issue`
- `mcp__github__list_issues`
</mcp_tool_naming>

<mcp_integration>
## Integration Behavior

- Plugin MCP servers start automatically when plugin is enabled
- Servers appear in `/mcp` command
- Tools integrate with Claude's existing toolkit
- Plugin servers run independently of user MCP servers
</mcp_integration>

<lsp_basics>
## LSP Server Configuration

Location: `.lsp.json` at plugin root (or inline in plugin.json)

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

LSP servers provide code intelligence (go to definition, find references, hover info).
</lsp_basics>

<lsp_fields>
## LSP Server Fields

### Required

| Field | Description |
|-------|-------------|
| `command` | LSP binary to execute (must be in PATH) |
| `extensionToLanguage` | Map file extensions to language IDs |

### Optional

| Field | Description |
|-------|-------------|
| `args` | Command-line arguments |
| `transport` | `stdio` (default) or `socket` |
| `env` | Environment variables |
| `initializationOptions` | Options for server initialization |
| `settings` | Settings via workspace/didChangeConfiguration |
| `workspaceFolder` | Workspace folder path |
| `startupTimeout` | Max startup wait (ms) |
| `shutdownTimeout` | Max shutdown wait (ms) |
| `restartOnCrash` | Auto-restart on crash |
| `maxRestarts` | Max restart attempts |
| `loggingConfig` | Debug logging config |
</lsp_fields>

<lsp_examples>
## LSP Examples

### Go (gopls)

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

### Python (Pyright)

```json
{
  "python": {
    "command": "pyright-langserver",
    "args": ["--stdio"],
    "extensionToLanguage": {
      ".py": "python"
    }
  }
}
```

### TypeScript

```json
{
  "typescript": {
    "command": "typescript-language-server",
    "args": ["--stdio"],
    "extensionToLanguage": {
      ".ts": "typescript",
      ".tsx": "typescriptreact",
      ".js": "javascript",
      ".jsx": "javascriptreact"
    }
  }
}
```

### Rust (rust-analyzer)

```json
{
  "rust": {
    "command": "rust-analyzer",
    "extensionToLanguage": {
      ".rs": "rust"
    }
  }
}
```
</lsp_examples>

<lsp_installation>
## LSP Server Installation

**You must install language server binaries separately.** LSP plugins configure Claude Code to use servers, they don't include the servers.

| Language | Install Command |
|----------|-----------------|
| Python (Pyright) | `pip install pyright` or `npm install -g pyright` |
| TypeScript | `npm install -g typescript-language-server typescript` |
| Rust | See rust-analyzer installation docs |
| Go | `go install golang.org/x/tools/gopls@latest` |

Check server is in PATH:
```bash
which gopls
which pyright-langserver
which typescript-language-server
```
</lsp_installation>

<lsp_debugging>
## LSP Debugging

Enable debug logging:

```json
{
  "typescript": {
    "command": "typescript-language-server",
    "args": ["--stdio"],
    "extensionToLanguage": {".ts": "typescript"},
    "loggingConfig": {
      "args": ["--log-level", "4"],
      "env": {
        "TSS_LOG": "-level verbose -file ${CLAUDE_PLUGIN_LSP_LOG_FILE}"
      }
    }
  }
}
```

Run with `--enable-lsp-logging` flag. Logs written to `~/.claude/debug/`.
</lsp_debugging>

<lsp_capabilities>
## LSP Capabilities

LSP integration provides:
- **Instant diagnostics**: Errors/warnings after edits
- **Code navigation**: Go to definition, find references
- **Hover information**: Type info and documentation
- **Language awareness**: Symbols, completions

Check `/plugin` Errors tab for LSP issues.
</lsp_capabilities>

<path_requirements>
## Path Requirements

**Always use `${CLAUDE_PLUGIN_ROOT}`** for plugin files:

```json
{
  "mcpServers": {
    "my-server": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  }
}
```

Plugins are copied to cache on install. Relative paths won't work.
</path_requirements>
