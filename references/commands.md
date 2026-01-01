<overview>
Reference for creating slash commands in Claude Code plugins. Commands are Markdown files with YAML frontmatter that define prompts Claude executes when users invoke them.
</overview>

<command_basics>
## Command File Structure

Location: `commands/` directory at plugin root

Filename becomes command name: `review.md` → `/plugin-name:review`

```markdown
---
description: Brief description shown in /help
---

# Command Title

Instructions for Claude when this command is invoked.
```
</command_basics>

<frontmatter_options>
## Frontmatter Fields

| Field | Purpose | Example |
|-------|---------|---------|
| `description` | Shown in /help | `"Review code for quality issues"` |
| `allowed-tools` | Tools command can use | `Bash(git:*), Read, Edit` |
| `argument-hint` | Shown during autocomplete | `"[file] [--strict]"` |
| `model` | Specific model to use | `"claude-3-5-haiku-20241022"` |
| `disable-model-invocation` | Prevent SlashCommand tool from calling | `true` |

### Example with All Options

```markdown
---
description: Create a git commit with smart message
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
argument-hint: [message]
model: claude-3-5-haiku-20241022
---

Analyze staged changes and create a commit.
If message provided: $ARGUMENTS
```
</frontmatter_options>

<arguments>
## Handling Arguments

### All Arguments: $ARGUMENTS

Captures everything after the command:

```markdown
Fix issue #$ARGUMENTS following our coding standards

# Usage: /plugin:fix 123 high-priority
# $ARGUMENTS = "123 high-priority"
```

### Positional Arguments: $1, $2, $3...

Access individual arguments:

```markdown
Review PR #$1 with priority $2 and assign to $3

# Usage: /plugin:review 456 high alice
# $1 = "456", $2 = "high", $3 = "alice"
```
</arguments>

<bash_execution>
## Bash Command Execution

Execute bash commands before the prompt runs using `!` prefix. Output is included in context.

**Requires** `allowed-tools` with `Bash`:

```markdown
---
allowed-tools: Bash(git:*)
description: Smart commit with context
---

## Context

- Current status: !`git status`
- Current diff: !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Your Task

Based on the above, create a commit.
```
</bash_execution>

<file_references>
## File References

Include file contents using `@` prefix:

```markdown
Review the implementation in @src/utils/helpers.js

Compare @src/old-version.js with @src/new-version.js
```
</file_references>

<namespacing>
## Command Namespacing

Plugin commands use format: `/plugin-name:command-name`

**Subdirectories for organization:**
- `commands/frontend/component.md` → `/component` with description "(project:frontend)"
- Subdirectory appears in description, not command name

**Conflict resolution:**
- Project commands override user commands with same name
- Commands in different subdirectories can share names
</namespacing>

<allowed_tools>
## allowed-tools Patterns

Restrict which tools the command can use:

```markdown
---
allowed-tools: Read, Grep, Glob
---
# Read-only command - no file modifications

---
allowed-tools: Bash(npm:*), Bash(npx:*)
---
# Only npm/npx commands

---
allowed-tools: Bash(git add:*), Bash(git commit:*), Bash(git push:*)
---
# Specific git operations
```

If omitted, command inherits conversation's tool permissions.
</allowed_tools>

<thinking_mode>
## Extended Thinking

Commands can trigger extended thinking by including thinking keywords in the prompt:

```markdown
Think step by step about this problem...

Let's reason through this carefully...
```
</thinking_mode>

<examples>
## Complete Examples

### Simple Review Command

```markdown
---
description: Quick code review
---

Review this code for:
- Potential bugs or edge cases
- Security concerns
- Performance issues
- Readability improvements

Be concise and actionable.
```

### Git Commit Helper

```markdown
---
description: Smart commit message generator
allowed-tools: Bash(git:*)
---

## Context
- Status: !`git status`
- Diff: !`git diff --staged`
- Recent commits: !`git log --oneline -5`

## Task
Generate a clear, conventional commit message based on staged changes.
```

### PR Review with Arguments

```markdown
---
description: Review pull request
argument-hint: [pr-number]
allowed-tools: Bash(gh:*)
---

Fetch and review PR #$1:

!`gh pr view $1`
!`gh pr diff $1`

Provide feedback on code quality, tests, and documentation.
```
</examples>
