---
name: manage-marketplace
description: Expert knowledge for creating, hosting, and using Claude Code plugin marketplaces. Use when the user wants to publish plugins, create marketplaces, discover plugins, or set up private plugin distribution.
---

# Manage Claude Code Plugin Marketplaces

You are an expert in Claude Code plugin marketplaces. Guide users through creating, hosting, and using marketplaces to distribute and discover plugins.

**IMPORTANT**: This skill contains the verified schema from official Claude Code documentation. All examples and formats are accurate as of the documentation fetch.

## What Are Plugin Marketplaces?

Plugin marketplaces are **collections of plugins** that users can browse, search, and install from. They enable:

- **Plugin discovery**: Find plugins by keyword, category, or author
- **Easy installation**: Install with `/plugin install <name>`
- **Version management**: Update plugins with a single command
- **Curation**: Maintain trusted plugin collections
- **Private distribution**: Share internal plugins within organizations

## Marketplace Types

### 1. GitHub Marketplace (Recommended)

Host marketplace in a GitHub repository.

**Advantages**:
- Free hosting
- Built-in version control
- Easy collaboration via PRs
- Issue tracking
- GitHub authentication built-in

**How users add it**:
```
/plugin marketplace add owner/repo
```

### 2. Git Repository

Any git hosting service (GitLab, Bitbucket, self-hosted).

**How users add it**:
```
/plugin marketplace add https://gitlab.com/company/plugins.git
```

### 3. Local Directory

For testing or local-only distribution.

**How users add it**:
```
/plugin marketplace add ./path/to/marketplace
```

## Complete Marketplace Schema

### marketplace.json Structure

```json
{
  "name": "marketplace-name",
  "owner": {
    "name": "Owner Name",
    "email": "optional@email.com"
  },
  "metadata": {
    "description": "Optional marketplace description",
    "version": "1.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./plugins/my-plugin",
      "description": "Plugin description",
      "version": "1.0.0",
      "author": {
        "name": "Author Name",
        "email": "optional@email.com"
      },
      "homepage": "https://docs.example.com",
      "repository": "https://github.com/user/plugin",
      "license": "MIT",
      "keywords": ["keyword1", "keyword2"],
      "category": "productivity",
      "tags": ["tag1", "tag2"],
      "strict": true,
      "commands": "./commands",
      "agents": ["./agents/agent1.md", "./agents/agent2.md"],
      "hooks": "./hooks/hooks.json",
      "mcpServers": "./mcp.json",
      "lspServers": "./lsp.json"
    }
  ]
}
```

## Required Fields

### Marketplace Level

| Field | Type | Description | Example |
|:------|:-----|:------------|:--------|
| `name` | string | Marketplace identifier (kebab-case, public-facing) | `"acme-tools"` |
| `owner` | object | Maintainer information (see owner fields below) | `{"name": "Team Name"}` |
| `plugins` | array | List of plugin entries | `[{...}]` |

**Reserved marketplace names**: Cannot use `claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `life-sciences`, or names that impersonate official marketplaces.

### Owner Fields

| Field | Type | Required | Description |
|:------|:-----|:---------|:------------|
| `name` | string | **Yes** | Name of maintainer or team |
| `email` | string | No | Contact email |

### Plugin Entry Required Fields

| Field | Type | Description |
|:------|:-----|:------------|
| `name` | string | Plugin identifier (kebab-case, public-facing) |
| `source` | string\|object | Where to fetch the plugin (see source types below) |

## Optional Fields

### Marketplace Metadata (Optional)

| Field | Type | Description |
|:------|:-----|:------------|
| `metadata.description` | string | Brief marketplace description |
| `metadata.version` | string | Marketplace version |
| `metadata.pluginRoot` | string | Base directory prepended to relative plugin paths (e.g., `"./plugins"` lets you write `"source": "formatter"` instead of `"source": "./plugins/formatter"`) |

### Plugin Entry Optional Fields

**Standard metadata**:

| Field | Type | Description |
|:------|:-----|:------------|
| `description` | string | Brief plugin description |
| `version` | string | Plugin version (semantic versioning) |
| `author` | object | Plugin author: `{"name": "Author Name", "email": "optional@email.com"}` |
| `homepage` | string | Plugin homepage or documentation URL |
| `repository` | string | Source code repository URL |
| `license` | string | SPDX license identifier (e.g., `"MIT"`, `"Apache-2.0"`) |
| `keywords` | array | Tags for plugin discovery: `["keyword1", "keyword2"]` |
| `category` | string | Plugin category for organization |
| `tags` | array | Tags for searchability |
| `strict` | boolean | Whether `plugin.json` is authority (default: `true`). See strict mode below. |

**Component configuration**:

| Field | Type | Description |
|:------|:-----|:------------|
| `commands` | string\|array | Custom paths to command files or directories |
| `agents` | string\|array | Custom paths to agent files |
| `hooks` | string\|object | Hooks configuration or path to hooks file |
| `mcpServers` | string\|object | MCP server config or path to MCP config |
| `lspServers` | string\|object | LSP server config or path to LSP config |

## Plugin Source Types

The `source` field tells Claude Code where to fetch each plugin.

### 1. Relative Path (String)

For plugins in the same repository as the marketplace:

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin"
}
```

**Requirements**:
- Must start with `./`
- Only works when marketplace is added via Git (not direct URL to marketplace.json)
- Path is relative to marketplace repository root

### 2. GitHub Repository (Object)

```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/repo-name"
  }
}
```

**With version pinning**:
```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/repo-name",
    "ref": "v2.0.0",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

**GitHub source fields**:

| Field | Required | Description |
|:------|:---------|:------------|
| `source` | **Yes** | Must be `"github"` |
| `repo` | **Yes** | Repository in `owner/repo` format |
| `ref` | No | Git branch or tag (defaults to repo default branch) |
| `sha` | No | Full 40-character commit SHA to pin to exact version |

### 3. Git Repository URL (Object)

For GitLab, Bitbucket, or any git host:

```json
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git"
  }
}
```

**With version pinning**:
```json
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git",
    "ref": "main",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

**Git URL source fields**:

| Field | Required | Description |
|:------|:---------|:------------|
| `source` | **Yes** | Must be `"url"` |
| `url` | **Yes** | Full git repository URL (must end with `.git`) |
| `ref` | No | Git branch or tag (defaults to repo default branch) |
| `sha` | No | Full 40-character commit SHA to pin to exact version |

### 4. npm Package (Object)

```json
{
  "name": "npm-plugin",
  "source": {
    "source": "npm",
    "package": "@scope/package-name",
    "version": "1.0.0",
    "registry": "https://registry.npmjs.org"
  }
}
```

**npm source fields**:

| Field | Required | Description |
|:------|:---------|:------------|
| `source` | **Yes** | Must be `"npm"` |
| `package` | **Yes** | npm package name (can be scoped: `@org/package`) |
| `version` | No | Package version (defaults to latest) |
| `registry` | No | Custom npm registry URL |

### 5. pip Package (Object)

```json
{
  "name": "pip-plugin",
  "source": {
    "source": "pip",
    "package": "package-name",
    "version": "1.0.0",
    "registry": "https://pypi.org/simple"
  }
}
```

**pip source fields**:

| Field | Required | Description |
|:------|:---------|:------------|
| `source` | **Yes** | Must be `"pip"` |
| `package` | **Yes** | Python package name |
| `version` | No | Package version (defaults to latest) |
| `registry` | No | Custom PyPI registry URL |

## Strict Mode

The `strict` field controls whether `plugin.json` is the authority for component definitions.

| Value | Behavior |
|:------|:---------|
| `true` (default) | `plugin.json` is the authority. Marketplace entry can supplement with additional components. Both sources are merged. |
| `false` | Marketplace entry is the entire definition. If plugin also has `plugin.json` declaring components, that's a conflict and plugin fails to load. |

**When to use**:
- **`strict: true`**: Plugin manages its own components via `plugin.json`. Marketplace can add extras.
- **`strict: false`**: Marketplace operator wants full control. Plugin provides files, marketplace defines what's exposed.

## Create a GitHub Marketplace

### Step 1: Create Repository

```bash
mkdir my-marketplace
cd my-marketplace
git init
```

### Step 2: Create Marketplace Manifest

```bash
mkdir -p .claude-plugin
cat > .claude-plugin/marketplace.json << 'EOF'
{
  "name": "my-marketplace",
  "owner": {
    "name": "Your Name",
    "email": "you@example.com"
  },
  "metadata": {
    "description": "My curated Claude Code plugins",
    "version": "1.0.0"
  },
  "plugins": []
}
EOF
```

### Step 3: Add Plugins

**Option A: Plugins in same repo (relative paths)**:

```json
{
  "name": "my-marketplace",
  "owner": {
    "name": "Your Name"
  },
  "plugins": [
    {
      "name": "review-tools",
      "source": "./plugins/review-tools",
      "description": "Code review automation",
      "version": "1.0.0",
      "author": {
        "name": "Your Name"
      }
    }
  ]
}
```

**Option B: Plugins from GitHub**:

```json
{
  "name": "my-marketplace",
  "owner": {
    "name": "Your Name"
  },
  "plugins": [
    {
      "name": "review-tools",
      "source": {
        "source": "github",
        "repo": "your-username/review-tools-plugin"
      },
      "description": "Code review automation"
    }
  ]
}
```

### Step 4: Add README

```bash
cat > README.md << 'EOF'
# My Claude Marketplace

Curated collection of Claude Code plugins.

## Installation

```bash
/plugin marketplace add your-username/my-marketplace
```

## Available Plugins

### review-tools
Code review automation tools.

## Contributing

Submit a PR to add your plugin to this marketplace.
EOF
```

### Step 5: Commit and Push

```bash
git add .
git commit -m "Initial marketplace"
git remote add origin https://github.com/your-username/my-marketplace.git
git push -u origin main
```

### Step 6: Share with Users

Users add your marketplace:
```
/plugin marketplace add your-username/my-marketplace
```

Then install plugins:
```
/plugin install review-tools@my-marketplace
```

## Complete Example

### Comprehensive Marketplace

```json
{
  "name": "acme-tools",
  "owner": {
    "name": "ACME DevTools Team",
    "email": "devtools@acme.com"
  },
  "metadata": {
    "description": "ACME Corp development tools",
    "version": "2.0.0",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "formatter",
      "description": "Automatic code formatting on save",
      "version": "2.1.0",
      "author": {
        "name": "DevTools Team",
        "email": "devtools@acme.com"
      },
      "homepage": "https://docs.acme.com/formatter",
      "repository": "https://github.com/acme/formatter-plugin",
      "license": "MIT",
      "keywords": ["formatting", "style", "automation"],
      "category": "productivity",
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Write|Edit",
            "hooks": [
              {
                "type": "command",
                "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh"
              }
            ]
          }
        ]
      }
    },
    {
      "name": "deployment-tools",
      "source": {
        "source": "github",
        "repo": "acme/deployment-plugin",
        "ref": "v3.0.0"
      },
      "description": "Deployment automation for production",
      "author": {
        "name": "DevOps Team"
      },
      "keywords": ["deployment", "ci-cd", "production"],
      "category": "deployment"
    },
    {
      "name": "security-scanner",
      "source": {
        "source": "url",
        "url": "https://gitlab.acme.com/security/scanner-plugin.git"
      },
      "description": "Security vulnerability scanning",
      "author": {
        "name": "Security Team"
      },
      "keywords": ["security", "scanning", "vulnerabilities"],
      "agents": ["./agents/security-reviewer.md"]
    }
  ]
}
```

## Using Variables in Plugin Configurations

Use `${CLAUDE_PLUGIN_ROOT}` to reference files within the plugin:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/lint.sh"
          }
        ]
      }
    ]
  },
  "mcpServers": {
    "plugin-service": {
      "command": "${CLAUDE_PLUGIN_ROOT}/bin/server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  }
}
```

This is necessary because plugins are copied to a cache location when installed.

## Install from Marketplace

### Add Marketplace

**From GitHub**:
```
/plugin marketplace add owner/repo
```

**From Git URL**:
```
/plugin marketplace add https://gitlab.com/team/marketplace.git
```

**From local directory**:
```
/plugin marketplace add ./my-marketplace
```

### List Marketplaces

```
/plugin marketplace list
```

### Browse Plugins

```
/plugin search keyword
```

### Install Plugin

```
/plugin install plugin-name
```

If plugin exists in multiple marketplaces:
```
/plugin install plugin-name@marketplace-name
```

### Update Plugins

```
/plugin update plugin-name
```

Or update all:
```
/plugin update --all
```

### Remove Marketplace

```
/plugin marketplace remove marketplace-name
```

## Version Management

### Plugin Versions

You can specify version in:
1. Plugin's `plugin.json` (takes precedence)
2. Marketplace entry

**Warning**: If both specify version, plugin manifest wins silently. For relative-path plugins, set version in marketplace entry. For external plugins, set in plugin manifest.

### Release Channels

Create separate marketplaces pointing to different refs:

**Stable marketplace**:
```json
{
  "plugins": [
    {
      "name": "my-tool",
      "source": {
        "source": "github",
        "repo": "org/my-tool",
        "ref": "stable"
      }
    }
  ]
}
```

**Latest marketplace**:
```json
{
  "plugins": [
    {
      "name": "my-tool",
      "source": {
        "source": "github",
        "repo": "org/my-tool",
        "ref": "latest"
      }
    }
  ]
}
```

**Important**: Plugin manifest must have different `version` at each ref, or Claude Code treats them as identical.

## Private Repositories

### Authentication

**For manual operations**: Uses your git credential helpers (same as `git clone`).

**For auto-updates at startup**: Set environment variable:

| Provider | Environment Variable |
|:---------|:---------------------|
| GitHub | `GITHUB_TOKEN` or `GH_TOKEN` |
| GitLab | `GITLAB_TOKEN` or `GL_TOKEN` |
| Bitbucket | `BITBUCKET_TOKEN` |

Set in shell config:
```bash
export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
```

## Project Configuration

### Auto-prompt Team Members

Add to `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "code-formatter@company-tools": true,
    "deployment-tools@company-tools": true
  }
}
```

Team members are prompted to install when they trust the folder.

### Restrict Allowed Marketplaces

Administrators can restrict which marketplaces users can add using managed settings:

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    },
    {
      "source": "url",
      "url": "https://plugins.example.com/marketplace.json"
    }
  ]
}
```

| Value | Behavior |
|:------|:---------|
| Undefined | No restrictions |
| `[]` | Complete lockdown - no marketplaces allowed |
| List | Only listed marketplaces allowed |

## Validation and Testing

### Validate Marketplace

```bash
claude plugin validate .
```

Or from within Claude Code:
```
/plugin validate .
```

### Test Locally

```bash
# Add local marketplace
/plugin marketplace add ./my-marketplace

# Install test plugin
/plugin install test-plugin@my-marketplace

# Test the plugin works
/test-plugin-command
```

## Troubleshooting

### Schema Validation Errors

**Error**: `plugins: Invalid input: expected array, received object`
- **Fix**: Change `"plugins": {}` to `"plugins": []`

**Error**: `owner: Invalid input: expected object, received undefined`
- **Fix**: Add `"owner": {"name": "Your Name"}`

**Error**: `plugins.0.author: Invalid input: expected object, received string`
- **Fix**: Change `"author": "Name"` to `"author": {"name": "Name"}`

**Error**: `plugins.0.source: Invalid input`
- **Fix**: For GitHub, use object format: `{"source": "github", "repo": "owner/repo"}`
- Don't use tarball URLs directly

### Marketplace Not Loading

- Verify marketplace URL is accessible
- Check `.claude-plugin/marketplace.json` exists at root
- Validate JSON syntax with `jq .`
- For private repos, verify authentication

### Plugin Installation Failures

- Verify plugin source is accessible
- For GitHub sources, ensure repository exists
- Check plugin has required files
- Test source manually by cloning

### Relative Path Failures in URL-Based Marketplaces

**Problem**: Added marketplace via URL (e.g., `https://example.com/marketplace.json`), but plugins with relative paths fail.

**Cause**: URL-based marketplaces only download `marketplace.json`, not plugin files.

**Solutions**:
1. Use GitHub/git sources instead of relative paths
2. Host marketplace in Git repository and add via git URL

### Private Repository Auth Failures

**For manual operations**:
- Verify git credentials: `gh auth status` (GitHub) or equivalent
- Test cloning manually

**For auto-updates**:
- Verify token is exported: `echo $GITHUB_TOKEN`
- Check token has repository read permission
- Ensure token hasn't expired

## Automation

### Bash Script to Add Plugin

```bash
#!/bin/bash
# add-plugin.sh

PLUGIN_NAME=$1
REPO=$2
DESC=$3

# Add to plugins array
jq --arg name "$PLUGIN_NAME" \
   --arg repo "$REPO" \
   --arg desc "$DESC" \
   '.plugins += [{
     name: $name,
     source: {source: "github", repo: $repo},
     description: $desc
   }]' .claude-plugin/marketplace.json > tmp.json

mv tmp.json .claude-plugin/marketplace.json

echo "Added $PLUGIN_NAME"
```

Usage:
```bash
./add-plugin.sh "my-tool" "org/my-tool" "Description"
```

### GitHub Actions Auto-Add

**.github/workflows/add-plugin.yml**:

```yaml
name: Add Plugin

on:
  repository_dispatch:
    types: [add-plugin]

jobs:
  add:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Add plugin
        run: |
          jq --arg name "${{ github.event.client_payload.name }}" \
             --arg repo "${{ github.event.client_payload.repo }}" \
             --arg desc "${{ github.event.client_payload.description }}" \
             '.plugins += [{
               name: $name,
               source: {source: "github", repo: $repo},
               description: $desc
             }]' .claude-plugin/marketplace.json > tmp.json
          mv tmp.json .claude-plugin/marketplace.json

      - name: Commit
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .claude-plugin/marketplace.json
          git commit -m "Add ${{ github.event.client_payload.name }}"
          git push
```

## Best Practices

### For Marketplace Maintainers

1. **Curate carefully** - Review all plugins before adding
2. **Security review** - Check for malicious code
3. **Test plugins** - Verify they work before listing
4. **Document clearly** - Provide good README with examples
5. **Use semantic versioning** - For marketplace metadata.version
6. **Organize with keywords** - Help users discover plugins
7. **Keep updated** - Remove abandoned plugins

### For Plugin Authors

1. **Tag releases** - Use git tags for versions
2. **Semantic versioning** - Follow MAJOR.MINOR.PATCH
3. **Document well** - Clear README and examples
4. **Test thoroughly** - Don't publish broken versions
5. **Maintain actively** - Respond to issues
6. **License clearly** - Specify in plugin.json
7. **Use keywords** - Make plugin discoverable

## Quick Reference

### Minimal Marketplace

```json
{
  "name": "my-marketplace",
  "owner": {
    "name": "Your Name"
  },
  "plugins": [
    {
      "name": "my-plugin",
      "source": {
        "source": "github",
        "repo": "username/plugin-repo"
      }
    }
  ]
}
```

### Source Type Quick Reference

```json
// Relative path (string)
"source": "./plugins/my-plugin"

// GitHub (object)
"source": {"source": "github", "repo": "owner/repo"}

// Git URL (object)
"source": {"source": "url", "url": "https://host.com/repo.git"}

// npm (object)
"source": {"source": "npm", "package": "package-name"}

// pip (object)
"source": {"source": "pip", "package": "package-name"}
```

## Related Skills

- Use `/claude-dev-toolkit:create-plugin` to build plugins
- Use `/claude-dev-toolkit:create-skill` to add plugin components
- Use `/claude-dev-toolkit:create-agent` for specialized subagents

## Additional Resources

- Official docs: https://code.claude.com/docs/en/plugin-marketplaces
- Plugin discovery: https://code.claude.com/docs/en/discover-plugins
- Plugin creation: https://code.claude.com/docs/en/plugins
- Plugin reference: https://code.claude.com/docs/en/plugins-reference

## Summary

**Correct Schema Requirements**:
- ✅ `owner`: Object with `{name, email?}`
- ✅ `plugins`: Array `[{...}]`
- ✅ `author`: Object with `{name, email?}` (per plugin)
- ✅ `source`: String (relative path) OR Object (github/url/npm/pip)

**Key Points**:
- For GitHub plugins, use `{"source": "github", "repo": "owner/repo"}`
- NOT tarball URLs
- Can pin versions with `ref` and `sha` fields
- Use `${CLAUDE_PLUGIN_ROOT}` for plugin-internal file references

This schema is verified from official Claude Code documentation and is accurate.
