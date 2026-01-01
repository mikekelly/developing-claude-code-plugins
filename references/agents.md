<overview>
Reference for creating subagent definitions in Claude Code plugins. Agents are specialized AI assistants that Claude can delegate tasks to, each with their own context window, tools, and system prompt.
</overview>

<agent_basics>
## Agent File Structure

Location: `agents/` directory at plugin root

```markdown
---
name: agent-name
description: When to use this agent
---

System prompt for the agent goes here.
```
</agent_basics>

<frontmatter_fields>
## Configuration Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier (lowercase, hyphens) |
| `description` | Yes | When Claude should use this agent |
| `tools` | No | Comma-separated list of allowed tools |
| `model` | No | Model alias: `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | No | `default`, `acceptEdits`, `bypassPermissions`, `plan` |
| `skills` | No | Comma-separated skill names to preload |

### Complete Example

```markdown
---
name: code-reviewer
description: Expert code review specialist. Use proactively after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
permissionMode: default
skills: code-quality, security-review
---

You are a senior code reviewer...
```
</frontmatter_fields>

<tool_configuration>
## Tool Configuration

### Inherit All Tools (Default)

Omit `tools` field to inherit all tools from main thread:

```markdown
---
name: general-helper
description: General assistant
---
```

### Specify Individual Tools

```markdown
---
tools: Read, Grep, Glob, Bash
---
```

### Available Tools

- File operations: `Read`, `Edit`, `Write`, `Glob`, `Grep`
- Shell: `Bash`
- Web: `WebFetch`, `WebSearch`
- Task delegation: `Task`
- MCP tools: `mcp__server__tool`

### MCP Tool Access

Agents can access MCP tools when `tools` is omitted (inherits all) or when explicitly listed:

```markdown
---
tools: Read, mcp__github__search_repositories
---
```
</tool_configuration>

<model_selection>
## Model Selection

| Value | Behavior |
|-------|----------|
| `sonnet` | Use Sonnet model (default for subagents) |
| `opus` | Use Opus model |
| `haiku` | Use Haiku model (fast, lightweight) |
| `inherit` | Use same model as main conversation |

```markdown
---
model: haiku
---
# Fast agent for simple tasks
```

```markdown
---
model: inherit
---
# Adapts to user's model choice
```
</model_selection>

<permission_modes>
## Permission Modes

| Mode | Behavior |
|------|----------|
| `default` | Normal permission prompts |
| `acceptEdits` | Auto-accept file edits |
| `bypassPermissions` | Skip all permission prompts |
| `plan` | Read-only planning mode |

```markdown
---
permissionMode: acceptEdits
---
# Agent can edit files without prompting
```
</permission_modes>

<skills_preloading>
## Skills Preloading

Subagents do NOT inherit skills from main conversation. Explicitly list skills to preload:

```markdown
---
skills: pr-review, security-check
---
```

Listed skills are loaded into agent's context when it starts.
</skills_preloading>

<proactive_invocation>
## Proactive Invocation

To encourage Claude to use your agent automatically, include trigger phrases in description:

```markdown
---
description: Use PROACTIVELY after code changes. MUST be used before committing.
---
```

Effective trigger phrases:
- "Use PROACTIVELY"
- "MUST BE USED when..."
- "Use immediately after..."
</proactive_invocation>

<examples>
## Complete Examples

### Code Reviewer

```markdown
---
name: code-reviewer
description: Expert code review specialist. Use PROACTIVELY after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code clarity and readability
- Proper error handling
- Security vulnerabilities
- Test coverage
- Performance considerations

Provide feedback organized by priority:
- Critical (must fix)
- Warnings (should fix)
- Suggestions (consider)
```

### Debugger

```markdown
---
name: debugger
description: Debugging specialist. Use when encountering errors, test failures, or unexpected behavior.
tools: Read, Edit, Bash, Grep, Glob
model: sonnet
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate failure location
4. Implement minimal fix
5. Verify solution

Focus on fixing the underlying issue, not symptoms.
```

### Read-Only Explorer

```markdown
---
name: codebase-explorer
description: Fast codebase exploration. Use for finding files, understanding structure.
tools: Read, Grep, Glob, Bash(ls:*), Bash(git log:*), Bash(git diff:*)
model: haiku
---

You explore codebases quickly without making changes.

Use Glob for file patterns, Grep for content search, Read for file contents.
Return findings with absolute file paths.
```
</examples>

<resumable_agents>
## Resumable Agents

Agents can be resumed to continue previous conversations:

```
# Initial invocation returns agentId: "abc123"
> Use code-analyzer to review authentication

# Resume later
> Resume agent abc123 and now analyze authorization
```

Agent continues with full context from previous session.
</resumable_agents>
