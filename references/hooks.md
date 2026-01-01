<overview>
Reference for creating event hooks in Claude Code plugins. Hooks execute shell commands or LLM prompts in response to Claude Code events like tool usage, session start, or user prompts.
</overview>

<hook_basics>
## Hook Configuration

Location: `hooks/hooks.json` at plugin root (or inline in plugin.json)

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/handler.sh"
          }
        ]
      }
    ]
  }
}
```
</hook_basics>

<hook_events>
## Available Events

| Event | When Triggered | Uses Matcher |
|-------|----------------|--------------|
| `PreToolUse` | Before tool executes | Yes |
| `PostToolUse` | After tool succeeds | Yes |
| `PostToolUseFailure` | After tool fails | Yes |
| `PermissionRequest` | Permission dialog shown | Yes |
| `UserPromptSubmit` | User submits prompt | No |
| `Notification` | Notification sent | Yes (type) |
| `Stop` | Claude stops responding | No |
| `SubagentStop` | Subagent stops | No |
| `SessionStart` | Session begins | Yes (source) |
| `SessionEnd` | Session ends | No |
| `PreCompact` | Before context compaction | Yes (trigger) |

### Matcher Patterns

- Exact match: `Write` matches only Write tool
- Regex: `Write|Edit` matches both
- Wildcard: `*` or `""` matches all

### SessionStart Matchers

- `startup` - Initial startup
- `resume` - Resume from --resume, --continue, /resume
- `clear` - From /clear
- `compact` - From auto/manual compact

### Notification Matchers

- `permission_prompt` - Permission requests
- `idle_prompt` - Claude waiting for input
- `auth_success` - Authentication success
</hook_events>

<hook_types>
## Hook Types

### Command Hooks

Execute shell commands:

```json
{
  "type": "command",
  "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh",
  "timeout": 30
}
```

### Prompt Hooks

Use LLM to evaluate decisions (for Stop/SubagentStop):

```json
{
  "type": "prompt",
  "prompt": "Evaluate if Claude should stop. Context: $ARGUMENTS",
  "timeout": 30
}
```

Prompt hooks respond with:
```json
{
  "decision": "approve" | "block",
  "reason": "Explanation"
}
```
</hook_types>

<exit_codes>
## Exit Code Behavior

| Exit Code | Behavior |
|-----------|----------|
| 0 | Success. stdout shown in verbose mode |
| 2 | Blocking error. stderr fed to Claude |
| Other | Non-blocking error. stderr shown to user |

### Exit Code 2 by Event

| Event | Effect |
|-------|--------|
| PreToolUse | Blocks tool call |
| PermissionRequest | Denies permission |
| PostToolUse | Shows stderr to Claude |
| UserPromptSubmit | Blocks prompt, erases it |
| Stop | Blocks stoppage |
| SubagentStop | Blocks stoppage |
</exit_codes>

<json_output>
## JSON Output Control

Hooks can return structured JSON (exit code 0 only):

### PreToolUse Decision

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "allow" | "deny" | "ask",
    "permissionDecisionReason": "Explanation",
    "updatedInput": {
      "field": "modified value"
    }
  }
}
```

### PostToolUse Feedback

```json
{
  "decision": "block",
  "reason": "Linting errors found",
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse",
    "additionalContext": "Extra info for Claude"
  }
}
```

### Stop/SubagentStop Control

```json
{
  "decision": "block",
  "reason": "Tasks not complete - continue working"
}
```

### UserPromptSubmit Context

```json
{
  "hookSpecificOutput": {
    "hookEventName": "UserPromptSubmit",
    "additionalContext": "Current time: 2024-01-15"
  }
}
```

Or just print to stdout (exit code 0) - it's added as context.

### SessionStart Context

```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Project context loaded"
  }
}
```

### Common Fields

```json
{
  "continue": false,
  "stopReason": "Stopping Claude",
  "suppressOutput": true,
  "systemMessage": "Warning message"
}
```
</json_output>

<environment_variables>
## Environment Variables

- `${CLAUDE_PLUGIN_ROOT}` - Plugin installation directory
- `${CLAUDE_PROJECT_DIR}` - Project root
- `$FILE` - File path (for Write/Edit matchers)
- `CLAUDE_ENV_FILE` - Path to persist env vars (SessionStart only)

### Persisting Environment

In SessionStart hooks:

```bash
#!/bin/bash
if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
fi
exit 0
```
</environment_variables>

<examples>
## Complete Examples

### Auto-Format on File Write

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Validate Before Commit

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate-command.py"
          }
        ]
      }
    ]
  }
}
```

### Load Context on Session Start

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/load-context.sh"
          }
        ]
      }
    ]
  }
}
```

### Intelligent Stop Verification

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Evaluate if tasks are complete. Context: $ARGUMENTS",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```
</examples>

<hook_input>
## Hook Input (stdin)

Hooks receive JSON via stdin:

```json
{
  "session_id": "abc123",
  "transcript_path": "~/.claude/projects/.../session.jsonl",
  "cwd": "/path/to/project",
  "permission_mode": "default",
  "hook_event_name": "PostToolUse",
  "tool_name": "Write",
  "tool_input": {"file_path": "/path", "content": "..."},
  "tool_response": {"success": true}
}
```
</hook_input>

<script_requirements>
## Script Requirements

1. Make executable: `chmod +x scripts/*.sh`
2. Add shebang: `#!/bin/bash` or `#!/usr/bin/env python3`
3. Use `${CLAUDE_PLUGIN_ROOT}` for paths
4. Read JSON from stdin
5. Exit with appropriate code
</script_requirements>
