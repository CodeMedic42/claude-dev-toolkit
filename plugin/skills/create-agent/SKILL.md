---
name: create-agent
description: Expert knowledge for creating Claude Code subagents. Use when the user wants to create custom agents, specialized workers, or task-specific AI assistants with custom prompts and tool restrictions.
---

# Create Claude Code Subagents

You are an expert in creating Claude Code subagents. Guide users through building specialized AI assistants that handle specific types of tasks with their own context, tools, and permissions.

## What Are Subagents?

Subagents are specialized AI assistants that:
- Run in their own **isolated context window**
- Have **custom system prompts** tailored to specific tasks
- Can have **restricted tool access** for focused work
- Work independently and **return summaries** to the main conversation
- Help **preserve your main context** by keeping exploration separate

When Claude encounters a task matching a subagent's description, it delegates to that subagent, which works independently and returns results.

**Subagents vs Agent Teams**: Subagents work within your session and report back. Agent teams are independent sessions that communicate peer-to-peer. Use subagents for focused tasks, agent teams for complex parallel work requiring collaboration.

## Built-in Subagents

Claude Code includes these built-in subagents:

| Agent | Model | Tools | Purpose |
|:------|:------|:------|:--------|
| **Explore** | Haiku | Read-only | File discovery, code search, codebase exploration |
| **Plan** | Inherits | Read-only | Codebase research during plan mode |
| **general-purpose** | Inherits | All tools | Complex research, multi-step operations, code modifications |
| **Bash** | Inherits | Bash only | Running terminal commands in separate context |

## When to Create Custom Subagents

Create a custom subagent when you need:
- **Specialized expertise**: Security review, performance analysis, documentation
- **Tool restrictions**: Read-only access, specific tool allowlists
- **Context isolation**: High-volume operations that shouldn't fill main context
- **Consistent behavior**: Same system prompt and workflow every time
- **Cost control**: Route specific tasks to cheaper models like Haiku

## Where Subagents Live

| Location | Path | Scope |
|:---------|:-----|:------|
| CLI flag | `--agents` JSON | Current session only |
| Project | `.claude/agents/` | This project |
| Personal | `~/.claude/agents/` | All your projects |
| Plugin | `<plugin>/agents/` | Where plugin is enabled |

**Priority**: CLI flag (highest) > Project > Personal > Plugin (lowest)

When subagents share the same name, higher-priority location wins.

## Subagent File Format

Subagents are markdown files with YAML frontmatter:

```markdown
---
name: agent-name
description: When Claude should delegate to this agent
tools: Read, Grep, Glob
model: sonnet
permissionMode: default
---

You are a specialized agent. Your system prompt goes here.

When invoked, follow these steps:
1. Understand the task
2. Execute with your tools
3. Return clear findings
```

## Frontmatter Fields

### Required Fields

| Field | Description |
|:------|:------------|
| `name` | Unique identifier (lowercase letters and hyphens) |
| `description` | When Claude should delegate to this agent. Be specific! |

### Optional Fields

| Field | Default | Description |
|:------|:--------|:------------|
| `tools` | All tools | Tools this agent can use (allowlist) |
| `disallowedTools` | None | Tools to deny (denylist) |
| `model` | `inherit` | Model to use: `sonnet`, `opus`, `haiku`, or `inherit` |
| `permissionMode` | `default` | Permission handling mode |
| `maxTurns` | None | Maximum agentic turns before stopping |
| `skills` | None | Skills to preload into agent context at startup |
| `mcpServers` | None | MCP servers available to this agent |
| `hooks` | None | Lifecycle hooks scoped to this agent |
| `memory` | None | Persistent memory scope: `user`, `project`, or `local` |
| `background` | `false` | Always run as background task |
| `isolation` | None | Set to `worktree` to run in temporary git worktree |

## Choose Tools

**Inherit all tools** (default):
```yaml
---
name: full-access-agent
description: Agent with all available tools
---
```

**Restrict tools** (allowlist):
```yaml
---
name: read-only-reviewer
description: Review code without modifications
tools: Read, Grep, Glob, Bash
---
```

**Allow most tools except specific ones** (denylist):
```yaml
---
name: safe-explorer
description: Explore but don't write
disallowedTools: Write, Edit
---
```

**Restrict which subagents can be spawned**:
```yaml
---
name: coordinator
description: Coordinates specialized workers
tools: Task(worker, researcher), Read, Bash
---
```

## Choose a Model

```yaml
model: sonnet    # Balance of capability and speed
model: opus      # Maximum capability
model: haiku     # Fast and cost-effective
model: inherit   # Use same model as main conversation (default)
```

## Permission Modes

| Mode | Behavior |
|:-----|:---------|
| `default` | Standard permission checking with prompts |
| `acceptEdits` | Auto-accept file edits |
| `dontAsk` | Auto-deny permission prompts (allowed tools still work) |
| `bypassPermissions` | Skip all permission checks (use with caution!) |
| `plan` | Plan mode (read-only exploration) |

## Preload Skills

Give agents domain knowledge by preloading skills at startup:

```yaml
---
name: api-developer
description: Implement API endpoints following team conventions
skills:
  - api-conventions
  - error-handling-patterns
---

Implement API endpoints. Follow the conventions and patterns from the preloaded skills.
```

The full content of each skill is injected at startup, not loaded on-demand.

## Enable Persistent Memory

Give agents a persistent directory to build knowledge over time:

```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---

You are a code reviewer. As you review code, update your agent memory with
patterns, conventions, and recurring issues you discover.
```

**Memory scopes:**
- `user`: `~/.claude/agent-memory/<name>/` - Learnings across all projects
- `project`: `.claude/agent-memory/<name>/` - Project-specific, shareable via git
- `local`: `.claude/agent-memory-local/<name>/` - Project-specific, gitignored

The agent's prompt includes:
- Instructions for reading/writing to the memory directory
- First 200 lines of `MEMORY.md` with instructions to curate if it grows
- Automatic Read, Write, Edit tool access for memory management

**Memory tips:**
- Ask agent to consult memory before work: "Check your memory for patterns"
- Ask agent to update memory after tasks: "Save what you learned"
- Default to `user` scope unless knowledge is project-specific

## Define Hooks for Subagents

**In the agent's frontmatter** (active only while agent runs):

```yaml
---
name: code-reviewer
description: Review code with automatic linting
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

**In `settings.json`** (main session hooks for agent lifecycle):

```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [
          { "type": "command", "command": "./scripts/setup-db.sh" }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          { "type": "command", "command": "./scripts/cleanup.sh" }
        ]
      }
    ]
  }
}
```

## Working with Subagents

### Automatic Delegation

Claude delegates based on:
- Task description in your request
- Agent's `description` field
- Current context

To encourage proactive use, include "use proactively" in the description.

**Explicit invocation:**
```
Use the test-runner subagent to fix failing tests
Have the code-reviewer subagent look at my recent changes
```

### Foreground vs Background

**Foreground** (blocking):
- Main conversation waits for completion
- Permission prompts pass through to you
- Can ask clarifying questions

**Background** (concurrent):
- Runs while you continue working
- Permissions pre-approved before launch
- Auto-denies anything not pre-approved
- No MCP tools available

Claude decides foreground/background based on the task. You can:
- Request "run this in the background"
- Press **Ctrl+B** to background a running task

Disable all background tasks: `export CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`

### Resume Subagents

Subagents retain full conversation history. To continue previous work:

```
Use the code-reviewer subagent to review the authentication module
[Agent completes]

Continue that code review and now analyze the authorization logic
[Claude resumes the subagent with full context]
```

Transcripts stored at: `~/.claude/projects/{project}/{sessionId}/subagents/agent-{agentId}.jsonl`

## Example Subagents

### Read-Only Code Reviewer

```markdown
---
name: code-reviewer
description: Expert code review specialist. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code clarity and readability
- Function and variable naming
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation
- Good test coverage
- Performance considerations

Provide feedback by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix issues.
```

### Debugger with Edit Access

```markdown
---
name: debugger
description: Debugging specialist for errors and test failures. Use when encountering issues.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works

Debugging process:
- Analyze error messages and logs
- Check recent code changes
- Form and test hypotheses
- Add strategic debug logging
- Inspect variable states

For each issue:
- Root cause explanation
- Evidence supporting diagnosis
- Specific code fix
- Testing approach
- Prevention recommendations

Focus on fixing the underlying issue, not symptoms.
```

### Data Scientist with Specific Tools

```markdown
---
name: data-scientist
description: Data analysis expert for SQL queries and BigQuery. Use for data analysis tasks.
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

When invoked:
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery CLI tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

Key practices:
- Optimized SQL with proper filters
- Appropriate aggregations and joins
- Comments explaining complex logic
- Format results for readability
- Data-driven recommendations

For each analysis:
- Explain query approach
- Document assumptions
- Highlight key findings
- Suggest next steps

Ensure queries are efficient and cost-effective.
```

### Database Query Validator with Hooks

```markdown
---
name: db-reader
description: Execute read-only database queries. Use when analyzing data.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access.

When asked to analyze data:
1. Identify relevant tables
2. Write efficient SELECT queries with filters
3. Present results clearly with context

You cannot modify data. If asked to INSERT, UPDATE, DELETE, or modify schema,
explain that you only have read access.
```

**Validation script** (`./scripts/validate-readonly-query.sh`):

```bash
#!/bin/bash
# Blocks SQL write operations

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [ -z "$COMMAND" ]; then
  exit 0
fi

# Block write operations
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE|REPLACE|MERGE)\b' > /dev/null; then
  echo "Blocked: Write operations not allowed. Use SELECT only." >&2
  exit 2
fi

exit 0
```

Make executable: `chmod +x ./scripts/validate-readonly-query.sh`

## Quick Start Templates

### Simple Read-Only Agent

```bash
mkdir -p ~/.claude/agents
cat > ~/.claude/agents/reviewer.md << 'EOF'
---
name: reviewer
description: Reviews code for quality and security issues
tools: Read, Grep, Glob
---

Review the code for quality, security, and best practices.
Provide specific, actionable feedback.
EOF
```

### Agent with Custom Model

```bash
cat > ~/.claude/agents/researcher.md << 'EOF'
---
name: researcher
description: Deep research on complex topics
model: sonnet
tools: Read, Grep, Glob, Bash
---

Research the topic thoroughly by exploring files and analyzing patterns.
Summarize findings with specific references.
EOF
```

### CLI-Defined Agent (Session Only)

```bash
claude --agents '{
  "quick-tester": {
    "description": "Run tests and report failures",
    "prompt": "Run the test suite and report only failing tests with error messages.",
    "tools": ["Bash", "Read"],
    "model": "haiku"
  }
}'
```

## Interactive Management

Use `/agents` command to:
- View all available subagents (built-in, user, project, plugin)
- Create new subagents with guided setup or Claude generation
- Edit existing agent configuration and tools
- Delete custom subagents
- See which agents are active when duplicates exist

## Disable Specific Subagents

In `settings.json`:

```json
{
  "permissions": {
    "deny": ["Task(Explore)", "Task(my-custom-agent)"]
  }
}
```

Or via CLI:
```bash
claude --disallowedTools "Task(Explore)"
```

## Best Practices

1. **Design focused agents**: One agent = one specialized task
2. **Write clear descriptions**: Claude uses this to decide when to delegate
3. **Limit tool access**: Grant only necessary permissions
4. **Test thoroughly**: Try automatic delegation and manual invocation
5. **Use memory**: Build institutional knowledge over time
6. **Check into git**: Share project agents with your team
7. **Choose right model**: Use Haiku for simple tasks, Sonnet for complex
8. **Isolate high-volume work**: Keep verbose output in agent context

## Common Patterns

**Isolate verbose operations:**
```
Use a subagent to run the test suite and report only failures
```

**Parallel research:**
```
Research authentication, database, and API modules in parallel using separate subagents
```

**Chain subagents:**
```
Use code-reviewer to find issues, then use optimizer to fix them
```

## Troubleshooting

**Agent not triggering:**
- Check description matches natural language users would use
- Verify agent appears in `/agents`
- Try explicit invocation: "Use the X subagent..."

**Agent uses wrong tools:**
- Check `tools` or `disallowedTools` fields
- Verify tool names match exactly (case-sensitive)

**Context issues:**
- Subagents don't inherit conversation history
- Subagents don't inherit invoked skills (must preload with `skills:`)
- Check auto-compaction logs if context fills up

**Permission errors:**
- Check `permissionMode` setting
- Verify parent session has necessary permissions
- Background agents need pre-approved permissions

## CLI-Based Agents vs File-Based

**File-based** (`.claude/agents/*.md`):
- ✅ Persist across sessions
- ✅ Easy to edit and version control
- ✅ Share with team via git
- Use for long-term agents

**CLI-based** (`--agents` JSON):
- ✅ Quick testing without files
- ✅ Session-specific configurations
- ✅ Automation scripts
- Use for experiments and one-off sessions

## Related Features

- Use `/create-skill` to add knowledge agents can use
- Use `/create-agent-team` for peer-to-peer collaboration
- Use `/create-plugin` to package agents for distribution
- Check hooks documentation for lifecycle automation

## Additional Resources

- Official docs: https://code.claude.com/docs/en/sub-agents
- Agent Skills standard: https://agentskills.io
- Use `/agents` for interactive management
