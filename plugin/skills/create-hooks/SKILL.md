---
name: create-hooks
description: Expert knowledge for creating Claude Code hooks. Use when the user wants to automate workflows, add validation, or trigger actions during Claude Code lifecycle events.
---

# Create Claude Code Hooks

You are an expert in creating Claude Code hooks. Guide users through automating workflows by triggering custom scripts and commands during Claude Code's lifecycle events.

## What Are Hooks?

Hooks are **automated triggers** that execute shell commands or inject prompts during specific Claude Code events:

- **Before/after tool calls**: Validate inputs, enforce standards, run linters
- **When subagents start/stop**: Setup environments, cleanup resources
- **User prompt submission**: Add context, enforce formats, run checks
- **Session start**: Initialize workspace, load configurations

Hooks enable workflow automation without modifying Claude Code itself.

## When to Use Hooks

Create hooks when you need to:
- **Validate operations**: Block unsafe commands, check syntax before execution
- **Enforce standards**: Auto-format code, run linters after edits
- **Add context**: Inject dynamic information into prompts
- **Setup/teardown**: Initialize environments, cleanup resources
- **Audit operations**: Log tool usage, track changes
- **Integrate tools**: Connect Claude Code to external systems

## Where Hooks Live

| Location | Path | Scope |
|:---------|:-----|:------|
| Project | `.claude/settings.json` | This project only |
| Personal | `~/.claude/settings.json` | All your projects |
| Plugin | `<plugin>/hooks/hooks.json` | Where plugin is enabled |

**Priority**: Project > Personal > Plugin

Multiple hooks can trigger for the same event and run in priority order.

## Hook Configuration

Define hooks in `settings.json` (or `hooks/hooks.json` for plugins):

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "pattern",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/my-hook.sh"
          }
        ]
      }
    ]
  }
}
```

## Hook Events

### Tool Lifecycle Hooks

**`PreToolUse`**: Before a tool executes
- **Use cases**: Validate inputs, block unsafe operations, check permissions
- **Can block**: Yes (exit code 2)

**`PostToolUse`**: After a tool executes
- **Use cases**: Run linters, format code, update indexes
- **Can block**: Yes (exit code 2 means revert/fail)

**`PreInvokedToolStream`**: Before tool starts streaming output
- **Use cases**: Setup logging, prepare monitoring
- **Can block**: No

**`PostInvokedToolStream`**: After tool finishes streaming
- **Use cases**: Cleanup, finalize logs
- **Can block**: No

### Agent Lifecycle Hooks

**`SubagentStart`**: When a subagent begins
- **Use cases**: Environment setup, load configurations
- **Can block**: No

**`SubagentStop`**: When a subagent ends
- **Use cases**: Cleanup resources, save state
- **Can block**: No

**`Stop`**: When main session ends
- **Use cases**: Final cleanup, save session data
- **Can block**: No

### Prompt Hooks

**`UserPromptSubmit`**: When user submits a message
- **Use cases**: Inject context, enforce formats, add instructions
- **Can block**: No (but can add to prompt)

## Hook Types

### Command Hooks

Execute shell commands:

```json
{
  "type": "command",
  "command": "./scripts/validate.sh",
  "timeout": 5000
}
```

**Input**: JSON on stdin with event data
**Output**:
- Exit code 0 = success
- Exit code 1 = warning (logged but continues)
- Exit code 2 = block operation
- Stderr = message to show user

**Timeout**: Default 5000ms, max 30000ms

### Prompt Hooks

Inject text into the prompt:

```json
{
  "type": "prompt",
  "prompt": "Current branch: !`git rev-parse --abbrev-ref HEAD`"
}
```

**Dynamic content**: Use `` !`command` `` to run shell commands
**Output**: Text is appended to user's message

### Agent Hooks (Subagent-scoped)

Run only when specific subagent is active:

```json
{
  "type": "agent",
  "agent": "code-reviewer",
  "hooks": {
    "PreToolUse": [...]
  }
}
```

## Matchers

Matchers control when hooks trigger based on tool names or patterns.

**Specific tool**:
```json
{
  "matcher": "Bash"
}
```

**Multiple tools** (regex OR):
```json
{
  "matcher": "Edit|Write"
}
```

**Tool with argument** (restrict to specific subagent):
```json
{
  "matcher": "Task\\(code-reviewer\\)"
}
```

**Match all tools**:
```json
{
  "matcher": ".*"
}
```

**Match specific files** (for Edit/Write):
```json
{
  "matcher": "Edit",
  "pathPattern": "\\.py$"
}
```

## JSON Input Schema

Hooks receive JSON on stdin with event context:

**PreToolUse/PostToolUse**:
```json
{
  "tool_name": "Bash",
  "tool_input": {
    "command": "npm test"
  },
  "tool_result": "...",  // Only in PostToolUse
  "cwd": "/path/to/project"
}
```

**SubagentStart/SubagentStop**:
```json
{
  "agent_name": "code-reviewer",
  "agent_id": "abc123",
  "cwd": "/path/to/project"
}
```

**UserPromptSubmit**:
```json
{
  "prompt": "User's message text",
  "cwd": "/path/to/project"
}
```

## Exit Codes

| Code | Meaning | Effect |
|:-----|:--------|:-------|
| 0 | Success | Continue normally |
| 1 | Warning | Log stderr but continue |
| 2 | Block | Stop operation, show stderr to user |
| Other | Error | Treated as failure |

## Example Hooks

### Validate Bash Commands

Block destructive git operations unless explicitly allowed:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate-git.sh"
          }
        ]
      }
    ]
  }
}
```

**validate-git.sh**:
```bash
#!/bin/bash
set -e

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [ -z "$COMMAND" ]; then
  exit 0
fi

# Block dangerous git operations
if echo "$COMMAND" | grep -iE 'git.*(push --force|reset --hard|clean -fd)' > /dev/null; then
  echo "Blocked: Destructive git operation detected. Use explicit commands only." >&2
  exit 2
fi

exit 0
```

Make executable: `chmod +x scripts/validate-git.sh`

### Auto-format Code After Edits

Run Prettier after file writes:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "pathPattern": "\\.(js|ts|jsx|tsx)$",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

**format-code.sh**:
```bash
#!/bin/bash
set -e

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

if [ -n "$FILE_PATH" ] && [ -f "$FILE_PATH" ]; then
  npx prettier --write "$FILE_PATH" 2>&1 || {
    echo "Warning: Prettier formatting failed" >&2
    exit 1
  }
fi

exit 0
```

### Inject Current Branch Context

Add git branch to every user message:

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "\n\n[Current branch: !`git rev-parse --abbrev-ref HEAD 2>/dev/null || echo 'unknown'`]"
          }
        ]
      }
    ]
  }
}
```

### Setup Database Connection for Agent

Start database before db-agent runs:

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/start-db.sh"
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "db-agent",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/stop-db.sh"
          }
        ]
      }
    ]
  }
}
```

### Enforce Commit Message Format

Validate commits follow conventional format:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate-commit.sh"
          }
        ]
      }
    ]
  }
}
```

**validate-commit.sh**:
```bash
#!/bin/bash
set -e

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Check if this is a git commit
if echo "$COMMAND" | grep -q 'git commit.*-m'; then
  # Extract commit message
  MSG=$(echo "$COMMAND" | sed -n 's/.*-m[[:space:]]*"\([^"]*\)".*/\1/p')

  # Validate conventional format: type(scope): description
  if ! echo "$MSG" | grep -qE '^(feat|fix|docs|style|refactor|test|chore)(\([a-z-]+\))?: .+'; then
    echo "Blocked: Commit message must follow conventional format: type(scope): description" >&2
    echo "Examples: feat(auth): add login, fix(api): handle errors" >&2
    exit 2
  fi
fi

exit 0
```

### Read-Only Query Validator

Ensure database agent only runs SELECT queries:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/validate-readonly.sh"
          }
        ]
      }
    ]
  }
}
```

**validate-readonly.sh**:
```bash
#!/bin/bash
set -e

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Check for SQL write operations
if echo "$COMMAND" | grep -iE '\\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE)\\b' > /dev/null; then
  echo "Blocked: Only SELECT queries allowed" >&2
  exit 2
fi

exit 0
```

### Log All Tool Usage

Audit trail of all Claude Code operations:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/log-tool.sh"
          }
        ]
      }
    ]
  }
}
```

**log-tool.sh**:
```bash
#!/bin/bash
set -e

INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name // "unknown"')
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

echo "$TIMESTAMP - $TOOL_NAME" >> .claude/audit.log
echo "$INPUT" >> .claude/audit-details.log

exit 0
```

## Environment Variables

Available in hook commands:

| Variable | Description |
|:---------|:------------|
| `${CLAUDE_PROJECT_ROOT}` | Absolute path to project root |
| `${CLAUDE_PLUGIN_ROOT}` | Absolute path to plugin directory |
| `${CLAUDE_SESSION_ID}` | Current session identifier |

Example:
```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
}
```

## Plugin Hooks

Define hooks in `<plugin>/hooks/hooks.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh"
          }
        ]
      }
    ]
  }
}
```

Or in subagent frontmatter (scoped to that agent):

```yaml
---
name: secure-agent
description: Agent with security validation
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

## Best Practices

1. **Keep hooks fast**: Default timeout is 5s, avoid slow operations
2. **Exit codes matter**: Use 0/1/2 correctly to control flow
3. **Parse JSON carefully**: Use `jq` for reliable parsing
4. **Make scripts executable**: `chmod +x scripts/*.sh`
5. **Handle errors gracefully**: Check for missing fields, handle edge cases
6. **Test thoroughly**: Test success, warning, and block scenarios
7. **Stderr for messages**: User sees stderr output when hooks block
8. **Scope appropriately**: Use project hooks for project-specific rules
9. **Use matchers wisely**: Specific matchers perform better than catch-all
10. **Document your hooks**: Add comments explaining purpose and behavior

## Quick Start Templates

### Basic Validation Hook

```bash
mkdir -p .claude/scripts
cat > .claude/scripts/validate.sh << 'EOF'
#!/bin/bash
set -e

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Add your validation logic here
if [[ "$COMMAND" == *"dangerous"* ]]; then
  echo "Blocked: Dangerous command detected" >&2
  exit 2
fi

exit 0
EOF

chmod +x .claude/scripts/validate.sh
```

Add to `.claude/settings.json`:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/scripts/validate.sh"
          }
        ]
      }
    ]
  }
}
```

### Post-Edit Formatter

```bash
mkdir -p .claude/scripts
cat > .claude/scripts/format.sh << 'EOF'
#!/bin/bash
set -e

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

if [ -f "$FILE_PATH" ]; then
  # Add your formatting command
  # Examples: prettier, black, gofmt, rustfmt
  npx prettier --write "$FILE_PATH" 2>&1 || exit 1
fi

exit 0
EOF

chmod +x .claude/scripts/format.sh
```

Add to settings:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "pathPattern": "\\.(js|ts|jsx|tsx)$",
        "hooks": [
          {
            "type": "command",
            "command": "./.claude/scripts/format.sh"
          }
        ]
      }
    ]
  }
}
```

### Context Injection Hook

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "\n\n[Working on branch: !`git branch --show-current`]"
          }
        ]
      }
    ]
  }
}
```

## Debugging Hooks

**Check hook output**:
```bash
# Test your script directly
echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./scripts/validate.sh
echo $?  # Check exit code
```

**View hook logs**:
- Stderr output appears in Claude Code session
- Exit codes control behavior
- Use `set -x` in bash for debug output

**Common issues**:
- Script not executable: `chmod +x script.sh`
- JSON parsing errors: Test with `jq` directly
- Timeout: Increase in hook config or optimize script
- Wrong exit code: Verify 0 (success), 1 (warning), 2 (block)

## Troubleshooting

**Hook not triggering**:
- Check matcher pattern matches tool name exactly
- Verify hook file path is correct
- Ensure script is executable
- Check Claude Code logs

**Hook blocks everything**:
- Verify exit code logic (only exit 2 should block)
- Test script independently with sample input
- Add debug output to stderr

**Timeout errors**:
- Default is 5000ms, max 30000ms
- Optimize slow operations
- Move heavy work to background processes

**JSON parsing fails**:
- Use `jq` for reliable parsing
- Handle missing fields with `// empty` default
- Test with actual hook input format

## Advanced Patterns

### Conditional Hooks Based on Context

```bash
#!/bin/bash
set -e

INPUT=$(cat)
CWD=$(echo "$INPUT" | jq -r '.cwd // empty')

# Different behavior based on directory
if [[ "$CWD" == *"production"* ]]; then
  echo "Blocked: Direct edits not allowed in production" >&2
  exit 2
fi

exit 0
```

### Chained Validation

Run multiple checks in sequence:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {"type": "command", "command": "./scripts/check-auth.sh"},
          {"type": "command", "command": "./scripts/check-safety.sh"},
          {"type": "command", "command": "./scripts/check-policy.sh"}
        ]
      }
    ]
  }
}
```

All hooks must succeed (exit 0) for operation to proceed.

### Dynamic Context Based on Files

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "\n\nRecent changes:\n!`git diff --stat HEAD~1..HEAD`"
          }
        ]
      }
    ]
  }
}
```

## Disable Hooks Temporarily

**Environment variable**:
```bash
export CLAUDE_CODE_DISABLE_HOOKS=1
claude
```

**Per-session**:
```bash
claude --no-hooks
```

**In settings.json**:
```json
{
  "hooks": {
    "enabled": false
  }
}
```

## Related Features

- Use `/create-skill` to define workflows hooks can support
- Use `/create-agent` to create agents with scoped hooks
- Use `/create-plugin` to package hooks for distribution
- Check settings documentation for hook configuration

## Additional Resources

- Official docs: https://code.claude.com/docs/en/hooks
- Hook events reference: https://code.claude.com/docs/en/hooks-guide
- Example hooks repository: https://github.com/anthropics/claude-code-examples

## Quick Reference

**Event Timing**:
- PreToolUse → Tool executes → PostToolUse
- SubagentStart → Subagent works → SubagentStop
- User types → UserPromptSubmit → Claude responds

**Exit Codes**:
- 0 = Success, continue
- 1 = Warning, log and continue
- 2 = Block operation

**Input/Output**:
- Input: JSON on stdin with event data
- Output: Exit code + stderr for user messages

**Common Use Cases**:
- Validation: PreToolUse with exit code 2
- Formatting: PostToolUse with code formatters
- Context: UserPromptSubmit with prompt type
- Setup/cleanup: SubagentStart/Stop with commands
