---
name: create-plugin
description: Expert knowledge for creating Claude Code plugins. Use when the user wants to create, build, or understand Claude Code plugins, package extensions, or distribute functionality to others.
---

# Create Claude Code Plugins

You are an expert in creating Claude Code plugins. Guide users through building plugins that extend Claude Code with custom functionality.

## When to Create a Plugin vs Standalone Config

**Use standalone configuration** (`.claude/` directory) when:
- Customizing Claude Code for a single project
- The configuration is personal and doesn't need sharing
- Experimenting before packaging
- You want short skill names like `/hello`

**Use a plugin** when:
- Sharing functionality with team or community
- Need the same skills/agents across multiple projects
- Want version control and easy updates
- Distributing through a marketplace
- Okay with namespaced skills like `/my-plugin:hello`

## Plugin Structure

A plugin directory contains:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest (optional but recommended)
├── skills/                  # Agent Skills with SKILL.md files
│   └── skill-name/
│       └── SKILL.md
├── commands/                # Legacy skills (use skills/ instead)
│   └── command.md
├── agents/                  # Custom subagent definitions
│   └── agent-name.md
├── hooks/                   # Event handlers
│   └── hooks.json
├── .mcp.json               # MCP server configurations
├── .lsp.json               # LSP server configurations
└── settings.json           # Default settings when plugin is enabled
```

**CRITICAL**: Only `plugin.json` goes inside `.claude-plugin/`. All other directories must be at the plugin root level.

## Plugin Manifest (plugin.json)

Create `.claude-plugin/plugin.json` with:

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "Brief description of what this plugin does",
  "author": {
    "name": "Your Name",
    "email": "optional@example.com"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/user/plugin",
  "license": "MIT",
  "keywords": ["tag1", "tag2"]
}
```

**Required fields:**
- `name`: Unique identifier (kebab-case, becomes skill namespace)
- `version`: Semantic version (MAJOR.MINOR.PATCH)
- `description`: What the plugin does

**Optional fields:**
- `author`: Author information
- `homepage`: Documentation URL
- `repository`: Source code URL
- `license`: SPDX license identifier
- `keywords`: Discovery tags

## Add Components to Your Plugin

### Skills
Place in `skills/` directory. Each skill is a folder with `SKILL.md`:

```
skills/
└── my-skill/
    ├── SKILL.md          # Main instructions (required)
    ├── reference.md      # Detailed docs (optional)
    └── scripts/          # Supporting files (optional)
```

See the `create-skill` skill for detailed skill creation guidance.

### Agents
Place in `agents/` directory as markdown files with frontmatter:

```markdown
---
name: agent-name
description: When Claude should use this agent
tools: Read, Grep, Glob
model: sonnet
---

System prompt for the agent...
```

See the `create-agent` skill for detailed agent creation guidance.

### Hooks
Create `hooks/hooks.json` for event handlers:

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

Use `${CLAUDE_PLUGIN_ROOT}` to reference plugin files.

### MCP Servers
Create `.mcp.json` to bundle MCP servers:

```json
{
  "mcpServers": {
    "plugin-service": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/my-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  }
}
```

### LSP Servers
Create `.lsp.json` for language server support:

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

**Note**: Users must install the language server binary separately.

### Default Settings
Create `settings.json` to apply configuration when plugin is enabled:

```json
{
  "agent": "custom-agent-name"
}
```

Currently only the `agent` key is supported.

## Testing Your Plugin

Test locally using the `--plugin-dir` flag:

```bash
claude --plugin-dir ./my-plugin
```

Test multiple plugins:

```bash
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
```

**Testing checklist:**
1. Try skills with `/plugin-name:skill-name`
2. Check agents appear in `/agents`
3. Verify hooks trigger correctly
4. Test MCP/LSP servers if included
5. Restart Claude Code after making changes

## Debugging

**Structure issues:**
- Ensure directories are at plugin root, not inside `.claude-plugin/`
- Only `plugin.json` goes in `.claude-plugin/`

**Validation:**
```bash
claude plugin validate .
```

Or from within Claude Code:
```
/plugin validate .
```

**Common errors:**
- "Plugin has an invalid manifest file": Check JSON syntax
- "No commands found in custom directory": Verify file structure
- "Executable not found in $PATH": Install required language server
- Hook not executing: Make scripts executable with `chmod +x`

## Distribution

Once your plugin is ready to share:

1. **Add documentation**: Include a comprehensive `README.md`
2. **Version your plugin**: Use semantic versioning (MAJOR.MINOR.PATCH)
3. **Publish to a marketplace**: See `/claude-dev-toolkit:manage-marketplace` skill for complete guidance on:
   - Creating your own marketplace
   - Publishing to existing marketplaces
   - Setting up private enterprise distribution
   - Version management and updates

For detailed marketplace documentation, invoke: `/claude-dev-toolkit:manage-marketplace`

## Environment Variables

**`${CLAUDE_PLUGIN_ROOT}`**: Absolute path to plugin directory. Use this in:
- Hook commands
- MCP server paths
- Script references

Example:
```json
{
  "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
}
```

## Converting Existing Configs to Plugins

If you have configurations in `.claude/`:

1. Create plugin structure:
   ```bash
   mkdir -p my-plugin/.claude-plugin
   ```

2. Create manifest:
   ```json
   {
     "name": "my-plugin",
     "version": "1.0.0",
     "description": "Migrated from standalone"
   }
   ```

3. Copy files:
   ```bash
   cp -r .claude/skills my-plugin/
   cp -r .claude/agents my-plugin/
   ```

4. Migrate hooks from settings to `hooks/hooks.json`

5. Test:
   ```bash
   claude --plugin-dir ./my-plugin
   ```

## Best Practices

1. **Start simple**: Begin with one or two skills, expand later
2. **Use namespacing**: Plugin skills are automatically namespaced
3. **Document thoroughly**: Include clear README and examples
4. **Test extensively**: Try all components before sharing
5. **Version properly**: Follow semantic versioning
6. **Use relative paths**: All paths relative to plugin root
7. **Validate early**: Run validation before distribution

## Quick Start Template

```bash
# Create structure
mkdir -p my-plugin/.claude-plugin
mkdir -p my-plugin/skills/hello

# Create manifest
cat > my-plugin/.claude-plugin/plugin.json << 'EOF'
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My first plugin"
}
EOF

# Create skill
cat > my-plugin/skills/hello/SKILL.md << 'EOF'
---
description: Greet the user warmly
---

Greet the user and offer to help them with their tasks.
EOF

# Test
claude --plugin-dir ./my-plugin
```

Then invoke with `/my-plugin:hello`

## Additional Resources

- Use `/create-skill` for detailed skill creation
- Use `/create-agent` for detailed agent creation
- Use `/create-agent-team` for agent team setup
- Check official docs at https://code.claude.com/docs/
