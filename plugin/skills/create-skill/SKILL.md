---
name: create-skill
description: Expert knowledge for creating Claude Code skills. Use when the user wants to create custom skills, slash commands, or add reusable knowledge and workflows to Claude Code.
---

# Create Claude Code Skills

You are an expert in creating Claude Code skills. Guide users through building skills that extend Claude's capabilities with custom knowledge and workflows.

## What Are Skills?

Skills extend what Claude can do. A skill is a markdown file (`SKILL.md`) with instructions that Claude either:
- **Loads automatically** when relevant to the conversation
- **Invokes with `/skill-name`** when you or Claude triggers it

Skills follow the [Agent Skills](https://agentskills.io) open standard and work across multiple AI tools.

## When to Create a Skill

Create a skill when you want to:
- **Add domain knowledge**: API conventions, style guides, architecture patterns
- **Create workflows**: Deployment steps, code review checklists, testing procedures
- **Build shortcuts**: Repeatable tasks triggered with `/skill-name`
- **Share expertise**: Package knowledge for team or community use

## Where Skills Live

| Location   | Path                                       | Scope                  |
|:-----------|:-------------------------------------------|:-----------------------|
| Personal   | `~/.claude/skills/<skill-name>/SKILL.md`   | All your projects      |
| Project    | `.claude/skills/<skill-name>/SKILL.md`     | This project only      |
| Plugin     | `<plugin>/skills/<skill-name>/SKILL.md`    | Where plugin is enabled|
| Enterprise | See managed settings                        | Organization-wide      |

**Priority**: Enterprise > Personal > Project. Plugin skills are namespaced and don't conflict.

## Skill Structure

Each skill is a directory containing `SKILL.md` and optional supporting files:

```
my-skill/
├── SKILL.md           # Main instructions (required)
├── reference.md       # Detailed docs (optional)
├── examples.md        # Usage examples (optional)
└── scripts/
    └── helper.py      # Utility scripts (optional)
```

## SKILL.md Format

Every `SKILL.md` has two parts:

1. **YAML frontmatter** (between `---` markers): Metadata that tells Claude when to use the skill
2. **Markdown content**: Instructions Claude follows when the skill is invoked

```yaml
---
name: my-skill
description: What this skill does and when to use it
---

Your skill instructions here in markdown...
```

## Frontmatter Fields

### Required/Recommended

| Field | Required | Description |
|:------|:---------|:------------|
| `name` | No | Display name (defaults to directory name). Kebab-case, max 64 chars. |
| `description` | Recommended | What the skill does and when to use it. Claude uses this to decide when to load the skill automatically. |

### Optional Control Fields

| Field | Default | Description |
|:------|:--------|:------------|
| `disable-model-invocation` | `false` | Set `true` to prevent Claude from loading automatically. User must invoke with `/name`. |
| `user-invocable` | `true` | Set `false` to hide from `/` menu. Use for background knowledge. |

### Optional Configuration Fields

| Field | Description |
|:------|:------------|
| `argument-hint` | Hint shown during autocomplete (e.g., `[filename]` or `[issue-number]`) |
| `allowed-tools` | Tools Claude can use without permission when this skill is active |
| `model` | Model to use when skill is active (`sonnet`, `opus`, `haiku`) |
| `context` | Set to `fork` to run in isolated subagent context |
| `agent` | Which subagent type to use when `context: fork` is set |
| `hooks` | Hooks scoped to this skill's lifecycle |

## Control Who Invokes Skills

**Default behavior**: Both you and Claude can invoke any skill.

**User-only skills** (`disable-model-invocation: true`):
- Only you can invoke with `/skill-name`
- Use for workflows with side effects: `/commit`, `/deploy`, `/send-slack-message`
- Claude won't trigger these automatically

**Claude-only skills** (`user-invocable: false`):
- Only Claude can invoke automatically
- Use for background knowledge that isn't actionable as a command
- Hidden from `/` menu

```yaml
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true
---

Deploy $ARGUMENTS to production:
1. Run tests
2. Build application
3. Push to deployment target
4. Verify deployment
```

## Types of Skill Content

**Reference Content**: Knowledge Claude applies throughout your session
```yaml
---
name: api-conventions
description: API design patterns for this codebase
---

When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

**Task Content**: Step-by-step instructions for specific actions
```yaml
---
name: deploy
description: Deploy the application to production
context: fork
disable-model-invocation: true
---

Deploy the application:
1. Run the test suite
2. Build the application
3. Push to the deployment target
4. Verify deployment succeeded
```

## Pass Arguments to Skills

Use `$ARGUMENTS` placeholder to capture user input:

```yaml
---
name: fix-issue
description: Fix a GitHub issue by number
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Understand requirements
3. Implement the fix
4. Write tests
5. Create a commit
```

Invoke with: `/fix-issue 123`

**Individual arguments** using `$ARGUMENTS[N]` or `$N`:

```yaml
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

Invoke with: `/migrate-component SearchBar React Vue`

## String Substitutions

Available substitutions in skill content:

| Variable | Description |
|:---------|:------------|
| `$ARGUMENTS` | All arguments passed when invoking |
| `$ARGUMENTS[N]` or `$N` | Specific argument by index (0-based) |
| `${CLAUDE_SESSION_ID}` | Current session ID |

## Advanced Patterns

### Inject Dynamic Context

Use `` !`command` `` syntax to run shell commands before Claude sees the skill:

```yaml
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request focusing on the main changes and their impact.
```

Commands run **before** Claude sees anything. Claude receives the output, not the command.

### Run Skills in Subagents

Add `context: fork` to run in isolation:

```yaml
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

The skill content becomes the prompt for the subagent. Choose an agent type:
- `Explore`: Read-only, optimized for codebase research
- `Plan`: For planning mode research
- `general-purpose`: Can explore and modify

### Add Supporting Files

Keep `SKILL.md` under 500 lines. Move detailed content to separate files:

```
my-skill/
├── SKILL.md          # Overview (required)
├── reference.md      # Detailed API docs
├── examples.md       # Usage examples
└── scripts/
    └── helper.py     # Utility scripts
```

Reference them from `SKILL.md`:

```markdown
## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

Claude loads supporting files only when needed.

### Restrict Tool Access

Limit which tools Claude can use when the skill is active:

```yaml
---
name: safe-reader
description: Read files without making changes
allowed-tools: Read, Grep, Glob
---

Analyze the codebase for patterns but do not modify any files.
```

### Enable Extended Thinking

Include the word "ultrathink" anywhere in your skill content to enable extended thinking mode.

## Quick Start Templates

### Simple Reference Skill

```bash
mkdir -p ~/.claude/skills/api-guide
cat > ~/.claude/skills/api-guide/SKILL.md << 'EOF'
---
description: API design guidelines for this project
---

When creating API endpoints:
- Use RESTful conventions
- Return consistent error formats
- Include proper validation
- Document with OpenAPI specs
EOF
```

### Action Skill with Arguments

```bash
mkdir -p ~/.claude/skills/commit-with-msg
cat > ~/.claude/skills/commit-with-msg/SKILL.md << 'EOF'
---
description: Create a git commit with a custom message
disable-model-invocation: true
---

Create a git commit with the message: "$ARGUMENTS"

1. Stage all changes with `git add -A`
2. Create commit with the provided message
3. Show the commit details with `git show HEAD`
EOF
```

### Research Skill in Subagent

```bash
mkdir -p ~/.claude/skills/analyze-security
cat > ~/.claude/skills/analyze-security/SKILL.md << 'EOF'
---
description: Analyze code for security vulnerabilities
context: fork
agent: Explore
---

Analyze the codebase for security issues:

1. Search for common vulnerability patterns
2. Check authentication and authorization
3. Review input validation
4. Look for SQL injection risks
5. Check for XSS vulnerabilities
6. Summarize findings with specific file references
EOF
```

## Troubleshooting

**Skill not triggering:**
- Check description includes keywords users would naturally say
- Verify skill appears in "What skills are available?"
- Try invoking directly with `/skill-name`
- Make description more specific

**Skill triggers too often:**
- Make description more specific about when to use it
- Add `disable-model-invocation: true` for manual-only invocation

**Claude doesn't see all skills:**
- Skills consume context budget (2% of context window or 16,000 chars)
- Run `/context` to check for excluded skills
- Set `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable to increase limit

**Skill content not loading:**
- Restart Claude Code after creating new skills
- Check YAML frontmatter syntax is valid
- Ensure `SKILL.md` filename is correct

## Best Practices

1. **Clear descriptions**: Use natural language that matches how users ask
2. **Keep focused**: One skill = one clear purpose
3. **Use arguments**: Make skills dynamic with `$ARGUMENTS`
4. **Add examples**: Show how to invoke the skill
5. **Progressive disclosure**: Keep `SKILL.md` concise, use supporting files for details
6. **Test thoroughly**: Try automatic and manual invocation
7. **Version control**: Share project skills via git
8. **Document usage**: Include examples and argument hints

## Share Skills

**Project skills**: Commit `.claude/skills/` to version control

**Plugins**: Create a `skills/` directory in your plugin:
```
my-plugin/
└── skills/
    └── my-skill/
        └── SKILL.md
```

**Managed**: Deploy organization-wide through managed settings

## Generate Visual Output

Skills can bundle scripts in any language. Example pattern for generating interactive HTML:

```yaml
---
name: codebase-visualizer
description: Generate interactive visualization of project structure
allowed-tools: Bash(python *)
---

Run the visualization script:

```bash
python ~/.claude/skills/codebase-visualizer/scripts/visualize.py .
```

This creates `codebase-map.html` with an interactive tree view.
```

The script generates a self-contained HTML file that opens in the browser.

## Related Features

- **Subagents**: Use skills in isolated context with `context: fork`
- **Plugins**: Package skills for distribution
- **Hooks**: Automate workflows around tool events
- **CLAUDE.md**: Add always-on context (use skills for on-demand content)

## Additional Resources

- Use `/create-plugin` to package skills as plugins
- Use `/create-agent` to create specialized subagents
- Check https://agentskills.io for the open standard
- Official docs at https://code.claude.com/docs/en/skills
