---
name: create-mcp-server
description: Expert knowledge for connecting Claude Code to MCP servers. Use when the user wants to integrate external tools, databases, APIs, or services through the Model Context Protocol.
---

# Connect Claude Code to MCP Servers

You are an expert in connecting Claude Code to Model Context Protocol (MCP) servers. Guide users through integrating external tools and data sources that extend Claude's capabilities.

## What is MCP?

**Model Context Protocol (MCP)** is an open standard that connects Claude Code to external tools, databases, and services through a simple protocol.

MCP servers provide:
- **Tools**: Functions Claude can call (e.g., search databases, call APIs)
- **Resources**: Data Claude can read (e.g., files, database records)
- **Prompts**: Pre-built prompt templates with arguments

## Why Use MCP?

MCP enables Claude to:
- **Access external data**: Query databases, read APIs, fetch live information
- **Execute operations**: Create tickets, send messages, update records
- **Integrate services**: Connect to GitHub, Slack, databases, cloud services
- **Extend capabilities**: Add domain-specific tools without modifying Claude Code

**Key benefits**:
- Open standard supported by multiple AI tools
- Growing ecosystem of pre-built servers
- Build once, use across different AI platforms
- Secure, controlled access to external systems

## MCP Server Types

### Pre-built Servers

Community and official servers available via npm:

```bash
# Official servers
npx -y @modelcontextprotocol/server-github
npx -y @modelcontextprotocol/server-postgres
npx -y @modelcontextprotocol/server-slack

# Community servers
npx -y @your-org/custom-mcp-server
```

**Popular official servers**:
- `server-github`: GitHub API integration (repos, issues, PRs)
- `server-postgres`: PostgreSQL database queries
- `server-sqlite`: SQLite database access
- `server-slack`: Slack workspace integration
- `server-filesystem`: File system operations
- `server-puppeteer`: Browser automation

### Custom Servers

Build your own MCP server when you need:
- Custom business logic
- Internal API integration
- Specialized data access
- Domain-specific operations

Languages supported:
- **TypeScript/JavaScript**: Official SDK with full support
- **Python**: Official SDK with full support
- **Other languages**: Implement the protocol directly

## Configure MCP Servers

MCP servers are configured in `.mcp.json` files:

| Location | Path | Scope |
|:---------|:-----|:------|
| Personal | `~/.claude/.mcp.json` | All your projects |
| Project | `.mcp.json` | This project only |
| Plugin | `<plugin>/.mcp.json` | Where plugin is enabled |

**Priority**: Project > Personal > Plugin

## Configuration Format

Basic `.mcp.json` structure:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token-here"
      }
    }
  }
}
```

## Transport Types

MCP supports three communication methods:

### 1. Stdio (Standard Input/Output)

**Default and recommended**. Server communicates via stdin/stdout.

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["-y", "@my-org/mcp-server"]
    }
  }
}
```

**Best for**: Local tools, CLI applications, most use cases

### 2. HTTP with SSE (Server-Sent Events)

Server runs as HTTP endpoint, Claude connects as client.

```json
{
  "mcpServers": {
    "remote-api": {
      "url": "https://api.example.com/mcp",
      "transport": "sse"
    }
  }
}
```

**Best for**: Remote services, multi-client scenarios, cloud deployments

### 3. HTTP (Direct)

Simple HTTP request/response (no streaming).

```json
{
  "mcpServers": {
    "simple-api": {
      "url": "https://api.example.com/mcp",
      "transport": "http"
    }
  }
}
```

**Best for**: Simple REST-like integrations, legacy systems

## Configuration Options

### Required Fields

| Field | Description |
|:------|:------------|
| `command` | Executable to run (stdio only) |
| `url` | HTTP endpoint (HTTP/SSE only) |

### Optional Fields

| Field | Default | Description |
|:------|:--------|:------------|
| `args` | `[]` | Command-line arguments |
| `env` | `{}` | Environment variables for the server |
| `transport` | `stdio` | Transport type: `stdio`, `sse`, or `http` |
| `disabled` | `false` | Temporarily disable without removing config |
| `timeout` | 5000 | Connection timeout in milliseconds |

## Authentication Methods

### API Tokens

Pass tokens via environment variables:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Security best practice**: Reference environment variables, don't hardcode tokens.

### OAuth 2.0

For HTTP/SSE servers requiring OAuth:

```json
{
  "mcpServers": {
    "oauth-api": {
      "url": "https://api.example.com/mcp",
      "transport": "sse",
      "auth": {
        "type": "oauth2",
        "authorizationUrl": "https://api.example.com/oauth/authorize",
        "tokenUrl": "https://api.example.com/oauth/token",
        "clientId": "${CLIENT_ID}",
        "clientSecret": "${CLIENT_SECRET}",
        "scopes": ["read", "write"]
      }
    }
  }
}
```

Claude Code handles the OAuth flow automatically.

### No Authentication

For internal services or public APIs:

```json
{
  "mcpServers": {
    "internal-tool": {
      "command": "/usr/local/bin/internal-mcp-server"
    }
  }
}
```

## Example Configurations

### GitHub Integration

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Setup**:
1. Generate GitHub personal access token: https://github.com/settings/tokens
2. Export token: `export GITHUB_TOKEN="ghp_..."`
3. Claude can now create issues, search repos, manage PRs

### PostgreSQL Database

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${DATABASE_URL}"
      }
    }
  }
}
```

**Setup**:
1. Export connection string: `export DATABASE_URL="postgresql://user:pass@localhost/db"`
2. Claude can now query your database

### Slack Workspace

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
      }
    }
  }
}
```

**Setup**:
1. Create Slack app: https://api.slack.com/apps
2. Add bot token scopes: `channels:read`, `chat:write`, `users:read`
3. Install app to workspace
4. Export tokens: `export SLACK_BOT_TOKEN="xoxb-..."` and `export SLACK_TEAM_ID="T..."`
5. Claude can now send messages, read channels

### File System Access

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/allowed/path",
        "/another/allowed/path"
      ]
    }
  }
}
```

**Setup**:
- Specify allowed paths as arguments
- Claude can only access listed directories

### Browser Automation

```json
{
  "mcpServers": {
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

**Setup**:
- Requires Chromium installed
- Claude can navigate web pages, take screenshots, extract data

### Remote MCP Server

```json
{
  "mcpServers": {
    "remote-tools": {
      "url": "https://mcp.example.com/v1",
      "transport": "sse",
      "timeout": 10000
    }
  }
}
```

## Plugin Integration

Bundle MCP servers in plugins by creating `<plugin>/.mcp.json`:

```json
{
  "mcpServers": {
    "plugin-service": {
      "command": "${CLAUDE_PLUGIN_ROOT}/bin/mcp-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DATA_DIR": "${CLAUDE_PLUGIN_ROOT}/data"
      }
    }
  }
}
```

**Important**:
- Use `${CLAUDE_PLUGIN_ROOT}` for plugin-relative paths
- Server binaries must be included in plugin distribution
- Document required environment variables in plugin README

## Build a Custom MCP Server

### TypeScript/JavaScript

**1. Install SDK**:
```bash
npm install @modelcontextprotocol/sdk
```

**2. Create server**:
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-custom-server",
  version: "1.0.0"
}, {
  capabilities: {
    tools: {}
  }
});

// Define tools
server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "my_tool",
    description: "What this tool does",
    inputSchema: {
      type: "object",
      properties: {
        param: { type: "string", description: "Parameter description" }
      },
      required: ["param"]
    }
  }]
}));

// Handle tool calls
server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "my_tool") {
    const result = await doSomething(request.params.arguments);
    return {
      content: [{ type: "text", text: JSON.stringify(result) }]
    };
  }
  throw new Error("Unknown tool");
});

// Start server
const transport = new StdioServerTransport();
await server.connect(transport);
```

**3. Package and use**:
```bash
# Build
npm run build

# Test locally
node dist/index.js

# Or publish to npm
npm publish
```

### Python

**1. Install SDK**:
```bash
pip install mcp
```

**2. Create server**:
```python
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

app = Server("my-custom-server")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="my_tool",
            description="What this tool does",
            inputSchema={
                "type": "object",
                "properties": {
                    "param": {"type": "string", "description": "Parameter"}
                },
                "required": ["param"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "my_tool":
        result = do_something(arguments["param"])
        return [TextContent(type="text", text=str(result))]
    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream)

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

**3. Use it**:
```json
{
  "mcpServers": {
    "my-tool": {
      "command": "python",
      "args": ["/path/to/server.py"]
    }
  }
}
```

## Server Capabilities

MCP servers can provide:

### Tools

Functions Claude can call:

```typescript
{
  name: "search_database",
  description: "Search records by query",
  inputSchema: {
    type: "object",
    properties: {
      query: { type: "string" },
      limit: { type: "number", default: 10 }
    },
    required: ["query"]
  }
}
```

### Resources

Data Claude can read:

```typescript
{
  uri: "database://customers/12345",
  name: "Customer Record",
  mimeType: "application/json",
  description: "Customer details and history"
}
```

### Prompts

Pre-built prompt templates:

```typescript
{
  name: "analyze_code",
  description: "Code analysis prompt",
  arguments: [
    { name: "file_path", description: "Path to file", required: true }
  ]
}
```

## Testing MCP Servers

### Test with MCP Inspector

Official debugging tool:

```bash
npx @modelcontextprotocol/inspector npx -y @my-org/my-server
```

Opens web UI to:
- Test tool calls
- Inspect resources
- Debug protocol messages
- Verify schemas

### Test in Claude Code

```bash
# Add to .mcp.json (project)
# Restart Claude Code
# Test tool: Ask Claude to use the tool
```

**Debugging tips**:
- Check server logs (stderr)
- Verify environment variables are set
- Test server binary directly
- Use inspector for protocol-level debugging

## Manage MCP Servers

### List configured servers

```
What MCP servers are configured?
```

Claude shows all servers from personal, project, and plugin configs.

### Temporarily disable

```json
{
  "mcpServers": {
    "my-server": {
      "disabled": true,
      "command": "..."
    }
  }
}
```

### Remove server

Delete from `.mcp.json` or remove the file.

## Security Considerations

**Best practices**:
1. **Never hardcode secrets**: Use environment variables
2. **Limit scope**: Only grant necessary permissions
3. **Validate inputs**: MCP servers should validate all inputs
4. **Use allow lists**: Restrict file system access to specific paths
5. **Audit logs**: Log MCP tool usage for security review
6. **Review server code**: Understand what third-party servers do
7. **Use HTTPS**: For remote servers, always use TLS
8. **Rotate tokens**: Regularly update API tokens and keys

## Troubleshooting

**Server not connecting**:
- Verify command path is correct
- Check server binary is installed
- Ensure environment variables are exported
- Test server directly: `npx -y @package/server`
- Check Claude Code logs

**Tools not appearing**:
- Restart Claude Code after config changes
- Verify `.mcp.json` syntax is valid
- Check server implements `tools/list` handler
- Test with MCP Inspector

**Authentication failures**:
- Verify tokens are exported: `echo $GITHUB_TOKEN`
- Check token has required scopes
- Ensure token hasn't expired
- Test API access directly with curl

**Timeout errors**:
- Increase timeout in config
- Check network connectivity for remote servers
- Verify server responds within timeout
- Test server performance independently

**Permission denied**:
- Check file permissions on server binary
- Verify environment variables are accessible
- Ensure paths are correct and readable
- Test with explicit absolute paths

## Advanced Patterns

### Multiple Environments

Different configs per environment:

```json
{
  "mcpServers": {
    "api-dev": {
      "url": "https://dev.example.com/mcp",
      "transport": "sse"
    },
    "api-prod": {
      "disabled": true,
      "url": "https://prod.example.com/mcp",
      "transport": "sse"
    }
  }
}
```

Enable/disable based on context.

### Chained Servers

Use multiple servers together:

```json
{
  "mcpServers": {
    "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"] },
    "slack": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-slack"] },
    "database": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-postgres"] }
  }
}
```

Claude can use tools from all servers in the same conversation.

### Conditional Server Loading

Use environment variables to control loading:

```json
{
  "mcpServers": {
    "prod-db": {
      "command": "bash",
      "args": ["-c", "[ \"$ENVIRONMENT\" = \"production\" ] && npx -y @my/prod-db-server || exit 1"]
    }
  }
}
```

## Official MCP Servers

**Available servers** (install with npx):

| Server | Package | Purpose |
|:-------|:--------|:--------|
| GitHub | `@modelcontextprotocol/server-github` | Issues, PRs, repos |
| PostgreSQL | `@modelcontextprotocol/server-postgres` | Database queries |
| SQLite | `@modelcontextprotocol/server-sqlite` | SQLite databases |
| Slack | `@modelcontextprotocol/server-slack` | Workspace integration |
| File System | `@modelcontextprotocol/server-filesystem` | File operations |
| Puppeteer | `@modelcontextprotocol/server-puppeteer` | Browser automation |
| Memory | `@modelcontextprotocol/server-memory` | Persistent KV store |
| Brave Search | `@modelcontextprotocol/server-brave-search` | Web search |

Find more at: https://github.com/modelcontextprotocol/servers

## Best Practices

1. **Start with official servers**: Use pre-built servers when available
2. **Test thoroughly**: Verify all tools work as expected
3. **Document requirements**: List required environment variables
4. **Use env vars for secrets**: Never commit tokens
5. **Version your servers**: Pin versions for stability
6. **Monitor usage**: Track tool calls and performance
7. **Implement retries**: Handle transient failures gracefully
8. **Validate schemas**: Ensure input schemas are clear and complete
9. **Provide good descriptions**: Help Claude understand when to use tools
10. **Keep servers focused**: One server = one service/domain

## Quick Start

### Use GitHub MCP Server

**1. Create personal access token**:
- Visit https://github.com/settings/tokens
- Click "Generate new token (classic)"
- Select scopes: `repo`, `read:org`
- Generate and copy token

**2. Export token**:
```bash
export GITHUB_TOKEN="ghp_your_token_here"
```

**3. Configure**:
```bash
cat > .mcp.json << 'EOF'
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
EOF
```

**4. Restart Claude Code**

**5. Test**:
```
Create a GitHub issue in my repo called "test-repo" with title "Test Issue"
```

Claude will use the GitHub MCP server to create the issue.

## Related Features

- Use `/create-plugin` to bundle MCP servers in plugins
- Use `/create-agent` to create agents with specific MCP access
- Use hooks to automate MCP tool usage
- Check security settings for MCP permissions

## Additional Resources

- MCP specification: https://spec.modelcontextprotocol.io
- Official servers: https://github.com/modelcontextprotocol/servers
- TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk
- Python SDK: https://github.com/modelcontextprotocol/python-sdk
- Claude Code MCP docs: https://code.claude.com/docs/en/mcp

## Summary

**MCP enables Claude Code to**:
- Access external data sources
- Execute operations in other systems
- Integrate with APIs and databases
- Extend capabilities through tools

**Key steps**:
1. Choose or build an MCP server
2. Configure in `.mcp.json`
3. Set up authentication (environment variables)
4. Restart Claude Code
5. Claude automatically uses available tools

**Remember**: MCP is an open standard. Servers work across different AI tools, so you can reuse integrations as the ecosystem grows.
