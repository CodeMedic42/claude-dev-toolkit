---
name: manage-marketplace
description: Expert knowledge for creating, hosting, and using Claude Code plugin marketplaces. Use when the user wants to publish plugins, create marketplaces, discover plugins, or set up private plugin distribution.
---

# Manage Claude Code Plugin Marketplaces

You are an expert in Claude Code plugin marketplaces. Guide users through creating, hosting, and using marketplaces to distribute and discover plugins.

## What Are Plugin Marketplaces?

Plugin marketplaces are **collections of plugins** that users can browse, search, and install from. They enable:

- **Plugin discovery**: Find plugins by keyword, category, or author
- **Easy installation**: Install with `/plugin install <name>`
- **Version management**: Update plugins with a single command
- **Curation**: Maintain trusted plugin collections
- **Private distribution**: Share internal plugins within organizations

## Marketplace Types

### 1. GitHub Marketplace

Host marketplace manifest on GitHub (most common).

**Advantages**:
- Free hosting
- Built-in versioning with git
- Easy collaboration via PRs
- Automatic updates via CI/CD
- GitHub's CDN for fast downloads

**Example**:
```
https://github.com/your-org/claude-plugins-marketplace
```

### 2. npm Marketplace

Publish marketplace as npm package.

**Advantages**:
- Familiar to JavaScript developers
- Built-in versioning with npm
- Package registry infrastructure
- Scoped packages for private marketplaces

**Example**:
```
@your-org/claude-marketplace
```

### 3. File-Based Marketplace

Serve marketplace manifest from any HTTP server.

**Advantages**:
- Full control over hosting
- Can use existing infrastructure
- Custom authentication possible
- Works with corporate firewalls

**Example**:
```
https://plugins.example.com/marketplace.json
```

### 4. Custom Marketplace

Build custom discovery and distribution systems.

**Advantages**:
- Complete customization
- Integration with existing systems
- Advanced search and filtering
- Custom authentication and authorization

## Marketplace Manifest Format

Every marketplace has a `marketplace.json` file:

```json
{
  "name": "my-marketplace",
  "version": "1.0.0",
  "description": "Collection of Claude Code plugins",
  "plugins": {
    "plugin-name": {
      "name": "plugin-name",
      "description": "What this plugin does",
      "version": "1.0.0",
      "author": "Author Name",
      "repository": "https://github.com/user/plugin-name",
      "source": "https://github.com/user/plugin-name/archive/refs/tags/v1.0.0.tar.gz",
      "keywords": ["tag1", "tag2"],
      "license": "MIT"
    }
  }
}
```

### Required Fields

| Field | Description |
|:------|:------------|
| `name` | Marketplace identifier |
| `version` | Marketplace version (semantic) |
| `plugins` | Object mapping plugin names to metadata |

### Plugin Entry Fields

| Field | Required | Description |
|:------|:---------|:------------|
| `name` | Yes | Plugin identifier (must match key) |
| `description` | Yes | Brief description |
| `version` | Yes | Plugin version |
| `source` | Yes | Download URL (tar.gz or zip) |
| `author` | No | Author name or organization |
| `repository` | No | Source code URL |
| `homepage` | No | Documentation URL |
| `keywords` | No | Array of search tags |
| `license` | No | SPDX license identifier |

## Create a GitHub Marketplace

### Step 1: Create Repository

```bash
mkdir claude-plugins-marketplace
cd claude-plugins-marketplace
git init
```

### Step 2: Create Marketplace Manifest

```bash
cat > marketplace.json << 'EOF'
{
  "name": "my-marketplace",
  "version": "1.0.0",
  "description": "Curated collection of Claude Code plugins",
  "plugins": {}
}
EOF
```

### Step 3: Add Plugins

```json
{
  "name": "my-marketplace",
  "version": "1.0.0",
  "description": "Curated collection of Claude Code plugins",
  "plugins": {
    "git-workflows": {
      "name": "git-workflows",
      "description": "Enhanced git workflow commands",
      "version": "1.0.0",
      "author": "Your Name",
      "repository": "https://github.com/your-org/git-workflows-plugin",
      "source": "https://github.com/your-org/git-workflows-plugin/archive/refs/tags/v1.0.0.tar.gz",
      "keywords": ["git", "workflow", "automation"],
      "license": "MIT"
    },
    "code-review": {
      "name": "code-review",
      "description": "Automated code review tools",
      "version": "2.0.0",
      "author": "Your Name",
      "repository": "https://github.com/your-org/code-review-plugin",
      "source": "https://github.com/your-org/code-review-plugin/archive/refs/tags/v2.0.0.tar.gz",
      "keywords": ["review", "quality", "security"],
      "license": "MIT"
    }
  }
}
```

### Step 4: Add README

```bash
cat > README.md << 'EOF'
# My Claude Plugins Marketplace

Curated collection of Claude Code plugins for our team.

## Installation

```bash
/plugin marketplace add https://raw.githubusercontent.com/your-org/claude-plugins-marketplace/main/marketplace.json
```

## Available Plugins

### git-workflows
Enhanced git workflow commands for common tasks.

### code-review
Automated code review tools with security scanning.

## Contributing

To add a plugin to this marketplace, submit a PR updating `marketplace.json`.
EOF
```

### Step 5: Commit and Push

```bash
git add .
git commit -m "Initial marketplace with plugins"
git remote add origin https://github.com/your-org/claude-plugins-marketplace.git
git push -u origin main
```

### Step 6: Share Marketplace URL

Users install with:
```
/plugin marketplace add https://raw.githubusercontent.com/your-org/claude-plugins-marketplace/main/marketplace.json
```

## Create an npm Marketplace

### Step 1: Initialize npm Package

```bash
mkdir claude-marketplace
cd claude-marketplace
npm init -y
```

### Step 2: Create Marketplace Manifest

```bash
cat > marketplace.json << 'EOF'
{
  "name": "@your-org/claude-marketplace",
  "version": "1.0.0",
  "description": "Claude Code plugins for our organization",
  "plugins": {
    "internal-tools": {
      "name": "internal-tools",
      "description": "Internal development tools",
      "version": "1.0.0",
      "source": "https://internal.example.com/plugins/internal-tools-1.0.0.tar.gz"
    }
  }
}
EOF
```

### Step 3: Update package.json

```json
{
  "name": "@your-org/claude-marketplace",
  "version": "1.0.0",
  "description": "Claude Code plugin marketplace",
  "main": "marketplace.json",
  "files": ["marketplace.json"],
  "keywords": ["claude", "plugins", "marketplace"],
  "license": "MIT"
}
```

### Step 4: Publish to npm

```bash
npm login
npm publish --access public
```

### Step 5: Share Package Name

Users install with:
```
/plugin marketplace add npm:@your-org/claude-marketplace
```

## Publish a Plugin to a Marketplace

### As Plugin Author

**1. Create GitHub release with tarball**:
```bash
# Tag your plugin
git tag v1.0.0
git push origin v1.0.0

# GitHub automatically creates tarball at:
# https://github.com/user/plugin/archive/refs/tags/v1.0.0.tar.gz
```

**2. Submit PR to marketplace**:
```json
{
  "plugins": {
    "your-plugin": {
      "name": "your-plugin",
      "description": "What your plugin does",
      "version": "1.0.0",
      "author": "Your Name",
      "repository": "https://github.com/user/your-plugin",
      "source": "https://github.com/user/your-plugin/archive/refs/tags/v1.0.0.tar.gz",
      "keywords": ["keyword1", "keyword2"],
      "license": "MIT"
    }
  }
}
```

**3. Wait for marketplace maintainer approval**

### As Marketplace Maintainer

**Review checklist**:
- [ ] Plugin has valid `plugin.json` manifest
- [ ] Source URL is accessible
- [ ] Version follows semantic versioning
- [ ] Description is clear and accurate
- [ ] Keywords are relevant
- [ ] License is specified
- [ ] No malicious code (security review)
- [ ] Plugin name doesn't conflict with existing

**Approval process**:
1. Review PR changes
2. Test plugin installation
3. Verify plugin functionality
4. Merge PR
5. Increment marketplace version
6. Users get update automatically

## Install from Marketplace

### Add Marketplace Source

```
/plugin marketplace add https://raw.githubusercontent.com/org/marketplace/main/marketplace.json
```

Or with a name:
```
/plugin marketplace add my-team https://raw.githubusercontent.com/org/marketplace/main/marketplace.json
```

### List Available Marketplaces

```
/plugin marketplace list
```

### Browse Plugins

```
/plugin search keyword
```

Claude shows plugins matching the keyword from all configured marketplaces.

### Install Plugin

```
/plugin install plugin-name
```

If multiple marketplaces have the same plugin:
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

### Semantic Versioning

Both marketplaces and plugins use semantic versioning:

```
MAJOR.MINOR.PATCH
```

- **MAJOR**: Breaking changes
- **MINOR**: New features, backwards-compatible
- **PATCH**: Bug fixes

### Update Plugin Version

When releasing a new plugin version:

**1. Update plugin version**:
```json
{
  "name": "my-plugin",
  "version": "1.1.0"  // Was 1.0.0
}
```

**2. Create git tag**:
```bash
git tag v1.1.0
git push origin v1.1.0
```

**3. Update marketplace**:
```json
{
  "plugins": {
    "my-plugin": {
      "version": "1.1.0",  // Update version
      "source": "https://github.com/user/my-plugin/archive/refs/tags/v1.1.0.tar.gz"  // Update URL
    }
  }
}
```

**4. Increment marketplace version**:
```json
{
  "name": "my-marketplace",
  "version": "1.0.1"  // Was 1.0.0
}
```

### Version Constraints

Marketplaces can specify version ranges:

```json
{
  "plugins": {
    "my-plugin": {
      "version": "^1.0.0",  // 1.x.x compatible
      "source": "https://github.com/user/my-plugin/archive/refs/tags/v1.0.0.tar.gz"
    }
  }
}
```

## Private Enterprise Marketplaces

### Requirements

- Internal hosting (file server, S3, internal GitHub)
- Network access from developer machines
- Optional: Authentication

### Example: S3-Hosted Marketplace

**1. Create marketplace manifest**:
```json
{
  "name": "acme-internal",
  "version": "1.0.0",
  "description": "ACME Corp internal plugins",
  "plugins": {
    "acme-tools": {
      "name": "acme-tools",
      "description": "Internal development tools",
      "version": "1.0.0",
      "source": "https://acme-plugins.s3.amazonaws.com/acme-tools-1.0.0.tar.gz"
    }
  }
}
```

**2. Upload to S3**:
```bash
aws s3 cp marketplace.json s3://acme-plugins/marketplace.json --acl public-read
```

**3. Share internal URL**:
```
/plugin marketplace add acme https://acme-plugins.s3.amazonaws.com/marketplace.json
```

### Example: Authenticated Marketplace

For marketplaces requiring authentication:

**1. Set up HTTP server with auth**:
```javascript
// server.js
const express = require('express');
const app = express();

app.use((req, res, next) => {
  const auth = req.headers['authorization'];
  if (auth !== 'Bearer YOUR_TOKEN') {
    return res.status(401).send('Unauthorized');
  }
  next();
});

app.get('/marketplace.json', (req, res) => {
  res.sendFile(__dirname + '/marketplace.json');
});

app.listen(3000);
```

**2. Configure authentication**:
```bash
export CLAUDE_MARKETPLACE_TOKEN="YOUR_TOKEN"
```

**3. Add marketplace with auth**:
```
/plugin marketplace add private https://marketplace.example.com/marketplace.json
```

Claude automatically includes the token in requests.

## Automate Marketplace Updates

### GitHub Actions Example

Auto-update marketplace when plugins are released:

```yaml
# .github/workflows/update-marketplace.yml
name: Update Marketplace

on:
  repository_dispatch:
    types: [plugin-release]

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Update plugin version
        run: |
          jq '.plugins["${{ github.event.client_payload.plugin }}"].version = "${{ github.event.client_payload.version }}"' marketplace.json > tmp.json
          mv tmp.json marketplace.json

      - name: Update source URL
        run: |
          jq '.plugins["${{ github.event.client_payload.plugin }}"].source = "${{ github.event.client_payload.source }}"' marketplace.json > tmp.json
          mv tmp.json marketplace.json

      - name: Increment marketplace version
        run: |
          VERSION=$(jq -r '.version' marketplace.json)
          NEW_VERSION=$(echo $VERSION | awk -F. '{$NF++; print}' OFS=.)
          jq ".version = \"$NEW_VERSION\"" marketplace.json > tmp.json
          mv tmp.json marketplace.json

      - name: Commit changes
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add marketplace.json
          git commit -m "Update ${{ github.event.client_payload.plugin }} to ${{ github.event.client_payload.version }}"
          git push
```

Trigger from plugin repository:
```bash
curl -X POST \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/your-org/marketplace/dispatches \
  -d '{"event_type":"plugin-release","client_payload":{"plugin":"my-plugin","version":"1.1.0","source":"https://..."}}'
```

## Marketplace Best Practices

### For Marketplace Maintainers

1. **Curate carefully**: Review all plugins before adding
2. **Security review**: Check for malicious code
3. **Test plugins**: Verify they work before listing
4. **Document requirements**: Clear submission guidelines
5. **Version properly**: Increment marketplace version on changes
6. **Keep updated**: Remove abandoned plugins
7. **Categorize**: Use keywords for discoverability
8. **Provide examples**: Show how to use listed plugins

### For Plugin Authors

1. **Semantic versioning**: Follow MAJOR.MINOR.PATCH
2. **Release notes**: Document changes in each version
3. **Test thoroughly**: Don't publish broken versions
4. **Document well**: Clear README and examples
5. **Maintain actively**: Respond to issues promptly
6. **License clearly**: Specify license in plugin.json
7. **Tag releases**: Use git tags for versions
8. **Update marketplace**: Keep marketplace entries current

## Security Considerations

### For Users

**When adding marketplaces**:
- [ ] Trust the marketplace maintainer
- [ ] Verify HTTPS URLs
- [ ] Review marketplace source on GitHub
- [ ] Check marketplace reputation
- [ ] Prefer official or well-known marketplaces

**When installing plugins**:
- [ ] Review plugin description and author
- [ ] Check source repository for red flags
- [ ] Read plugin documentation
- [ ] Start with trusted authors
- [ ] Report suspicious plugins

### For Maintainers

**Security review checklist**:
- [ ] Scan for obvious malware patterns
- [ ] Review plugin code manually
- [ ] Check for suspicious network calls
- [ ] Verify source repository ownership
- [ ] Test in isolated environment
- [ ] Check dependencies for known vulnerabilities
- [ ] Require plugins to document permissions needed
- [ ] Remove plugins with security issues immediately

**Hosting security**:
- [ ] Use HTTPS for marketplace URLs
- [ ] Enable CORS headers if needed
- [ ] Implement rate limiting
- [ ] Monitor access logs
- [ ] Rotate authentication tokens
- [ ] Use CDN for DDoS protection

## Troubleshooting

**Marketplace won't add**:
- Verify URL is accessible (test in browser)
- Check JSON syntax is valid
- Ensure HTTPS (HTTP may be blocked)
- Verify CORS headers if cross-origin
- Check network/firewall settings

**Plugin won't install**:
- Verify source URL is accessible
- Check plugin name matches manifest
- Ensure tarball structure is correct
- Test source URL manually: `curl -L <url>`
- Check for version conflicts

**Updates not working**:
- Verify marketplace version incremented
- Check plugin version increased
- Clear cache: `rm -rf ~/.claude/cache`
- Re-add marketplace

**Authentication failures**:
- Verify token is exported correctly
- Check token hasn't expired
- Ensure token has required permissions
- Test auth with curl first

## Example Marketplaces

### Community Marketplace

```json
{
  "name": "claude-community",
  "version": "1.0.0",
  "description": "Community-contributed Claude Code plugins",
  "plugins": {
    "git-enhanced": {
      "name": "git-enhanced",
      "description": "Enhanced git commands and workflows",
      "version": "1.2.0",
      "author": "Community",
      "repository": "https://github.com/community/git-enhanced",
      "source": "https://github.com/community/git-enhanced/archive/refs/tags/v1.2.0.tar.gz",
      "keywords": ["git", "workflow"],
      "license": "MIT"
    },
    "api-tools": {
      "name": "api-tools",
      "description": "API development and testing tools",
      "version": "2.0.0",
      "author": "Community",
      "repository": "https://github.com/community/api-tools",
      "source": "https://github.com/community/api-tools/archive/refs/tags/v2.0.0.tar.gz",
      "keywords": ["api", "testing"],
      "license": "Apache-2.0"
    }
  }
}
```

### Enterprise Marketplace

```json
{
  "name": "acme-enterprise",
  "version": "1.0.0",
  "description": "ACME Corp approved plugins",
  "plugins": {
    "acme-standards": {
      "name": "acme-standards",
      "description": "ACME coding standards and linters",
      "version": "3.0.0",
      "author": "ACME DevTools Team",
      "repository": "https://github.acme.com/devtools/acme-standards",
      "source": "https://artifacts.acme.com/plugins/acme-standards-3.0.0.tar.gz",
      "keywords": ["standards", "linting", "internal"]
    },
    "acme-deploy": {
      "name": "acme-deploy",
      "description": "ACME deployment workflows",
      "version": "2.5.0",
      "author": "ACME DevOps Team",
      "repository": "https://github.acme.com/devops/acme-deploy",
      "source": "https://artifacts.acme.com/plugins/acme-deploy-2.5.0.tar.gz",
      "keywords": ["deployment", "ci-cd", "internal"]
    }
  }
}
```

## Quick Start: Create Your First Marketplace

```bash
# 1. Create repository
mkdir my-marketplace
cd my-marketplace
git init

# 2. Create manifest
cat > marketplace.json << 'EOF'
{
  "name": "my-marketplace",
  "version": "1.0.0",
  "description": "My curated Claude Code plugins",
  "plugins": {}
}
EOF

# 3. Add README
cat > README.md << 'EOF'
# My Claude Marketplace

Install with:
```
/plugin marketplace add https://raw.githubusercontent.com/YOUR_USERNAME/my-marketplace/main/marketplace.json
```
EOF

# 4. Commit and push
git add .
git commit -m "Initial marketplace"
git remote add origin https://github.com/YOUR_USERNAME/my-marketplace.git
git push -u origin main

# 5. Share the URL
echo "Users can add your marketplace with:"
echo "/plugin marketplace add https://raw.githubusercontent.com/YOUR_USERNAME/my-marketplace/main/marketplace.json"
```

## Related Features

- Use `/create-plugin` to build plugins for distribution
- Check plugin validation before publishing
- Review security guidelines for safe plugin development
- Use CI/CD for automated marketplace updates

## Additional Resources

- Official docs: https://code.claude.com/docs/en/plugin-marketplaces
- Plugin discovery: https://code.claude.com/docs/en/discover-plugins
- Example marketplaces: https://github.com/anthropics/claude-marketplaces
- Semantic versioning: https://semver.org

## Summary

**Marketplaces enable**:
- Plugin discovery and installation
- Version management and updates
- Curated collections
- Private enterprise distribution

**Key concepts**:
- Marketplace manifest (`marketplace.json`)
- Plugin metadata (name, version, source)
- Hosting options (GitHub, npm, custom)
- Semantic versioning
- Security and curation

**Get started**:
1. Create marketplace manifest
2. Add plugin entries
3. Host on GitHub or npm
4. Share marketplace URL
5. Users install with `/plugin marketplace add`

Marketplaces make it easy to discover, install, and share Claude Code plugins across teams and communities!
