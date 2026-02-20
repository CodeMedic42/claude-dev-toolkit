# Claude Dev Toolkit

A comprehensive toolkit for building Claude Code extensions including plugins, skills, agents, and agent teams.

## Overview

This plugin provides Claude with expert knowledge for creating various Claude Code extensions. Each skill contains detailed, actionable guidance extracted from the official Claude Code documentation.

## Installation

### Test Locally

```bash
claude --plugin-dir ./claude-dev-toolkit
```

### Install from Marketplace

Once added to a marketplace:

```bash
/plugin marketplace add <marketplace-source>
/plugin install claude-dev-toolkit@<marketplace-name>
```

## Skills Included

### 1. `/claude-dev-toolkit:create-plugin`

Expert knowledge for creating Claude Code plugins.

**Use when:**
- Creating new plugins
- Packaging extensions for distribution
- Understanding plugin structure and manifests
- Adding components (skills, agents, hooks, MCP servers, LSP servers)
- Converting standalone configs to plugins

**Key topics:**
- Plugin structure and manifest schema
- Component organization (skills, agents, hooks, MCP, LSP)
- Testing with `--plugin-dir`
- Debugging and validation
- Distribution strategies

### 2. `/claude-dev-toolkit:create-skill`

Expert knowledge for creating Claude Code skills.

**Use when:**
- Creating custom slash commands
- Adding reusable knowledge to Claude
- Building workflow automation
- Creating reference documentation
- Implementing task-specific instructions

**Key topics:**
- SKILL.md format and frontmatter
- Control fields (`disable-model-invocation`, `user-invocable`)
- Passing arguments with `$ARGUMENTS`
- Dynamic context injection with `` !`command` ``
- Running skills in subagents with `context: fork`
- Supporting files and progressive disclosure

### 3. `/claude-dev-toolkit:create-agent`

Expert knowledge for creating Claude Code subagents.

**Use when:**
- Creating specialized AI assistants
- Building task-specific workers
- Implementing tool restrictions
- Setting up context isolation
- Creating agents with persistent memory

**Key topics:**
- Subagent markdown format with frontmatter
- Tool allowlists and denylists
- Permission modes
- Model selection
- Preloading skills
- Persistent memory configuration
- Hooks for subagent lifecycle
- Foreground vs background execution

### 4. `/claude-dev-toolkit:create-agent-team`

Expert knowledge for orchestrating Claude Code agent teams.

**Use when:**
- Setting up parallel work across multiple sessions
- Creating teams with peer-to-peer communication
- Implementing shared task coordination
- Building complex collaborative workflows
- Managing multi-agent architectures

**Key topics:**
- Agent teams vs subagents
- Architecture patterns (specialized, research, review teams)
- Task management and coordination
- Communication patterns
- Team monitoring and intervention
- Cost considerations
- Limitations and best practices

### 5. `/claude-dev-toolkit:create-hooks`

Expert knowledge for creating Claude Code hooks.

**Use when:**
- Automating workflows around Claude Code events
- Adding validation before tool execution
- Running formatters or linters after code changes
- Setting up/tearing down environments
- Injecting dynamic context into prompts

**Key topics:**
- Hook events (PreToolUse, PostToolUse, SubagentStart, etc.)
- Hook types (command, prompt, agent)
- Matchers and patterns
- JSON input schemas
- Exit codes and blocking
- Environment variables
- Security and validation patterns

### 6. `/claude-dev-toolkit:create-mcp-server`

Expert knowledge for connecting Claude Code to MCP servers.

**Use when:**
- Integrating external tools and APIs
- Connecting to databases
- Adding custom functionality via MCP
- Using pre-built MCP servers
- Building custom MCP servers

**Key topics:**
- Model Context Protocol overview
- Configuration in `.mcp.json`
- Transport types (stdio, HTTP, SSE)
- Authentication methods
- Official MCP servers
- Building custom servers (TypeScript/Python)
- Security best practices

### 7. `/claude-dev-toolkit:create-claude-md`

Expert knowledge for managing Claude Code memory with CLAUDE.md files.

**Use when:**
- Setting up project memory
- Documenting conventions and patterns
- Creating persistent project context
- Managing auto memory
- Organizing hierarchical documentation

**Key topics:**
- CLAUDE.md file structure and location
- Content guidelines and best practices
- Importing external documentation
- Directory-specific context
- Dynamic content with shell commands
- Auto memory system
- Organizing large projects

### 8. `/claude-dev-toolkit:manage-marketplace`

Expert knowledge for creating, hosting, and using Claude Code plugin marketplaces.

**Use when:**
- Publishing plugins to marketplaces
- Creating your own marketplace
- Installing plugins from marketplaces
- Setting up private/enterprise distribution
- Managing plugin versions and updates

**Key topics:**
- Marketplace types (GitHub, npm, custom)
- Marketplace manifest format
- Publishing and version management
- Private enterprise marketplaces
- Plugin discovery and installation
- Security and curation
- Automation with CI/CD

## Usage Examples

### Create a New Plugin

```
/claude-dev-toolkit:create-plugin

Help me create a plugin that adds git workflow commands
```

### Design a Custom Skill

```
/claude-dev-toolkit:create-skill

I want to create a skill that helps with API documentation
```

### Build a Specialized Agent

```
/claude-dev-toolkit:create-agent

Create a security reviewer agent that only has read access
```

### Set Up an Agent Team

```
/claude-dev-toolkit:create-agent-team

I need a team to implement a new authentication system
```

### Create Workflow Hooks

```
/claude-dev-toolkit:create-hooks

Add a hook that runs prettier after I edit JavaScript files
```

### Connect to External Services

```
/claude-dev-toolkit:create-mcp-server

Help me connect Claude to my PostgreSQL database
```

### Set Up Project Memory

```
/claude-dev-toolkit:create-claude-md

Create a CLAUDE.md file for my React project
```

### Publish to a Marketplace

```
/claude-dev-toolkit:manage-marketplace

How do I publish my plugin to a marketplace?
```

## How Skills Work Together

These skills are designed to work in combination:

1. **Start with skills** to add knowledge and workflows
2. **Package as plugins** when ready to share
3. **Create agents** for specialized task execution
4. **Orchestrate teams** for complex parallel work

Example workflow:
```
1. /create-skill → Design your workflows
2. /create-agent → Build specialized workers
3. /create-plugin → Package everything together
4. /create-agent-team → Coordinate complex tasks
```

## Skill Invocation

All skills can be invoked in two ways:

**Automatic (model-invoked):** Claude will load these skills automatically when you ask questions about creating plugins, skills, agents, agent teams, hooks, MCP servers, CLAUDE.md files, or plugin marketplaces.

**Manual invocation:** Use the skill name directly:
- `/claude-dev-toolkit:create-plugin`
- `/claude-dev-toolkit:create-skill`
- `/claude-dev-toolkit:create-agent`
- `/claude-dev-toolkit:create-agent-team`
- `/claude-dev-toolkit:create-hooks`
- `/claude-dev-toolkit:create-mcp-server`
- `/claude-dev-toolkit:create-claude-md`
- `/claude-dev-toolkit:manage-marketplace`

## Development

### Plugin Structure

```
claude-dev-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── create-plugin/
│   │   └── SKILL.md
│   ├── create-skill/
│   │   └── SKILL.md
│   ├── create-agent/
│   │   └── SKILL.md
│   ├── create-agent-team/
│   │   └── SKILL.md
│   ├── create-hooks/
│   │   └── SKILL.md
│   ├── create-mcp-server/
│   │   └── SKILL.md
│   ├── create-claude-md/
│   │   └── SKILL.md
│   └── manage-marketplace/
│       └── SKILL.md
└── README.md
```

### Testing Changes

After modifying skills:

1. Restart Claude Code if testing with `--plugin-dir`
2. Verify skills appear in `/` menu
3. Test both automatic and manual invocation
4. Validate with `claude plugin validate .`

## Contributing

To add new skills to this toolkit:

1. Create a new directory under `skills/`
2. Add `SKILL.md` with proper frontmatter
3. Update this README
4. Test thoroughly
5. Increment version in `plugin.json`

## Version History

### 1.0.2 (2026-02-20)
- **Complete rewrite**: Verified manage-marketplace skill against official documentation
- **Corrected source format**: GitHub sources use object format, not tarball URLs
- Added all 5 source types: relative path, github, url, npm, pip
- Documented all optional fields with complete schema
- Added strict mode, version management, and private repo authentication
- Removed all incorrect examples and replaced with verified formats

### 1.0.1 (2026-02-20)
- **Critical fix**: Corrected marketplace.json schema in manage-marketplace skill
- Fixed: `plugins` must be array (not object), `owner` field required
- Updated all examples, scripts, and automation code with correct schema

### 1.0.0 (2026-02-20)
- Added manage-marketplace skill for plugin distribution and discovery
- Complete plugin lifecycle coverage: creation → distribution → installation
- Enhanced marketplace documentation (GitHub, npm, private/enterprise)

### 1.1.0 (2026-02-20)
- Added three new skills: create-hooks, create-mcp-server, create-claude-md
- Complete coverage of Claude Code extension ecosystem
- Enhanced documentation for workflow automation and external integrations

### 1.0.0 (2026-02-20)
- Initial release
- Four core skills: create-plugin, create-skill, create-agent, create-agent-team
- Comprehensive documentation extracted from official Claude Code docs

## License

MIT

## Resources

- [Claude Code Documentation](https://code.claude.com/docs/)
- [Agent Skills Standard](https://agentskills.io)
- [Plugin Development Guide](https://code.claude.com/docs/en/plugins)
- [Skills Guide](https://code.claude.com/docs/en/skills)
- [Subagents Guide](https://code.claude.com/docs/en/sub-agents)
- [Agent Teams Guide](https://code.claude.com/docs/en/agent-teams)

## Support

For issues or questions:
- Open an issue in the repository
- Check the official Claude Code documentation
- Review the skill content for detailed guidance
