---
name: github-mcp-server
description: Official GitHub MCP Server for AI-powered repository management, issue/PR automation, CI/CD monitoring, and code analysis through natural language
triggers:
  - "help me interact with GitHub repositories"
  - "set up the GitHub MCP server"
  - "query GitHub issues and pull requests"
  - "analyze GitHub Actions workflows"
  - "manage GitHub repositories with AI"
  - "troubleshoot GitHub MCP server connection"
  - "configure GitHub MCP authentication"
  - "browse GitHub code with MCP"
---

# github-mcp-server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The GitHub MCP Server is GitHub's official Model Context Protocol server that connects AI tools directly to GitHub's platform. It enables AI agents to read repositories, manage issues and PRs, analyze code, monitor CI/CD pipelines, and automate workflows through natural language interactions.

## What It Does

- **Repository Management**: Browse code, search files, analyze commits, understand project structure
- **Issue & PR Automation**: Create, update, and manage issues and pull requests
- **CI/CD Intelligence**: Monitor GitHub Actions, analyze build failures, manage releases
- **Code Analysis**: Review security findings, Dependabot alerts, code patterns
- **Team Collaboration**: Access discussions, manage notifications, analyze team activity

## Installation

### Remote Server (Recommended)

The remote GitHub MCP Server is hosted by GitHub and requires no local installation.

#### VS Code (1.101+)

**Using OAuth (Recommended)**:
```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

**Using GitHub PAT**:
```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${input:github_mcp_pat}"
      }
    }
  },
  "inputs": [
    {
      "type": "promptString",
      "id": "github_mcp_pat",
      "description": "GitHub Personal Access Token",
      "password": true
    }
  ]
}
```

#### Claude Desktop

```json
{
  "mcpServers": {
    "github": {
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer YOUR_GITHUB_PAT"
      }
    }
  }
}
```

**Note**: Replace `YOUR_GITHUB_PAT` with `${GITHUB_PAT}` if your host supports environment variables.

#### Cursor

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${GITHUB_PAT}"
      }
    }
  }
}
```

### Local Server (Docker)

For hosts without remote MCP support or GitHub Enterprise Server:

```json
{
  "servers": {
    "github": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PAT}"
      }
    }
  }
}
```

## Authentication

### Creating a GitHub Personal Access Token (PAT)

1. Go to https://github.com/settings/personal-access-tokens/new
2. Select permissions based on your needs:
   - `repo` - Full repository access (read/write code, issues, PRs)
   - `read:org` - Read organization data
   - `workflow` - Update GitHub Actions workflows
   - `admin:org` - Manage organization (if needed)
3. Generate token and store securely
4. Set as environment variable:
   ```bash
   export GITHUB_PAT=ghp_yourTokenHere
   ```

### Token Security

Store tokens in environment variables, never hardcode:

```bash
# .env file
GITHUB_PAT=ghp_yourTokenHere

# Add to .gitignore
echo ".env" >> .gitignore
```

## Configuration

### Toolsets

Configure which GitHub APIs are available:

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "X-GitHub-MCP-Toolsets": "repos,issues,pulls"
      }
    }
  }
}
```

**Available Toolsets**:
- `repos` - Repository operations
- `issues` - Issue management
- `pulls` - Pull request operations
- `actions` - GitHub Actions workflows
- `search` - Code and repository search
- `gists` - Gist management
- `notifications` - Notification access

**Default Toolset** (when not specified):
`repos,issues,pulls,actions,search`

### GitHub Enterprise Cloud (ghe.com)

```json
{
  "servers": {
    "github-enterprise": {
      "type": "http",
      "url": "https://copilot-api.octocorp.ghe.com/mcp",
      "headers": {
        "Authorization": "Bearer ${GITHUB_ENTERPRISE_PAT}"
      }
    }
  }
}
```

### GitHub Enterprise Server

```json
{
  "servers": {
    "github-ghes": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GITHUB_PERSONAL_ACCESS_TOKEN",
        "-e",
        "GITHUB_HOST",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_PAT}",
        "GITHUB_HOST": "https://github.yourcompany.com"
      }
    }
  }
}
```

### Insiders Mode

Access experimental features:

**Via URL**:
```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/insiders"
    }
  }
}
```

**Via Header**:
```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "X-MCP-Insiders": "true"
      }
    }
  }
}
```

## Usage Patterns

### Repository Operations

**List repository files**:
```
Show me all Python files in github/github-mcp-server
```

**Read file contents**:
```
Show me the contents of main.go in github/github-mcp-server
```

**Search code**:
```
Find all functions that use the GitHub API in github/github-mcp-server
```

**Analyze commits**:
```
Show recent commits in github/github-mcp-server main branch
```

### Issue Management

**List issues**:
```
Show open issues in github/github-mcp-server labeled as bug
```

**Create issue**:
```
Create an issue in github/github-mcp-server titled "Add support for X" with description "We need X because..."
```

**Update issue**:
```
Add comment to issue #123 in github/github-mcp-server saying "Working on this"
```

**Close issue**:
```
Close issue #456 in github/github-mcp-server
```

### Pull Request Operations

**List PRs**:
```
Show all open pull requests in github/github-mcp-server
```

**Create PR**:
```
Create a pull request in github/github-mcp-server from feature-branch to main with title "Add new feature"
```

**Review PR**:
```
Show me the diff for PR #789 in github/github-mcp-server
```

**Merge PR**:
```
Merge pull request #101 in github/github-mcp-server
```

### GitHub Actions

**List workflows**:
```
Show all GitHub Actions workflows in github/github-mcp-server
```

**View workflow runs**:
```
Show recent workflow runs for CI in github/github-mcp-server
```

**Analyze failures**:
```
Why did the latest build fail in github/github-mcp-server?
```

**Trigger workflow**:
```
Trigger the deploy workflow in github/github-mcp-server
```

### Code Analysis

**Security alerts**:
```
Show Dependabot alerts for github/github-mcp-server
```

**Code search across organization**:
```
Find all repositories in github org using deprecated API
```

**Analyze repository structure**:
```
What's the directory structure of github/github-mcp-server?
```

## Common Patterns

### Multi-Repository Analysis

```
Compare the CI/CD setup between github/repo1 and github/repo2
```

### Automated Triage

```
Label all open issues in github/github-mcp-server that mention "authentication" with "auth"
```

### Release Management

```
Create a release for github/github-mcp-server version 2.0.0 with notes from recent PRs
```

### Team Coordination

```
Show me all PRs assigned to me across github organization
```

### Code Review Assistance

```
Review PR #234 in github/github-mcp-server and suggest improvements
```

## Troubleshooting

### Connection Issues

**Problem**: Server not connecting
```
Error: Unable to connect to GitHub MCP server
```

**Solution**: Verify server configuration
1. Check URL is correct: `https://api.githubcopilot.com/mcp/`
2. For local Docker: Ensure Docker is running
3. Restart your IDE/MCP host

### Authentication Failures

**Problem**: 401 Unauthorized
```
Error: Authentication failed
```

**Solution**: Verify token
1. Check PAT is valid: https://github.com/settings/tokens
2. Ensure token has required scopes
3. Verify environment variable is set correctly:
   ```bash
   echo $GITHUB_PAT
   ```
4. Restart MCP host after updating token

### Permission Errors

**Problem**: 403 Forbidden or 404 Not Found
```
Error: Resource not accessible
```

**Solution**: Check token permissions
1. Verify PAT has required scope (e.g., `repo` for private repos)
2. Confirm you have access to the repository
3. For organizations, check if SSO is required:
   - Go to https://github.com/settings/tokens
   - Click "Configure SSO" next to your token
   - Authorize the organization

### Docker Issues

**Problem**: Image pull failed
```
Error: failed to pull ghcr.io/github/github-mcp-server
```

**Solution**: 
1. Check Docker is running: `docker ps`
2. Logout if you have expired credentials:
   ```bash
   docker logout ghcr.io
   ```
3. Pull manually to test:
   ```bash
   docker pull ghcr.io/github/github-mcp-server
   ```

### Rate Limiting

**Problem**: API rate limit exceeded
```
Error: API rate limit exceeded
```

**Solution**: Authenticated requests have higher limits
1. Ensure PAT is properly configured
2. Wait for rate limit reset (check response headers)
3. For heavy usage, consider GitHub Apps instead of PATs

### Enterprise Server Connection

**Problem**: Cannot connect to GitHub Enterprise Server
```
Error: Connection timeout
```

**Solution**: Check GITHUB_HOST configuration
1. Ensure URL includes `https://` scheme:
   ```json
   "GITHUB_HOST": "https://github.company.com"
   ```
2. Verify server is accessible from your network
3. Check firewall/VPN settings

### Toolset Not Available

**Problem**: Tool not found
```
Error: Tool 'create_gist' not available
```

**Solution**: Enable required toolset
```json
{
  "headers": {
    "X-GitHub-MCP-Toolsets": "repos,issues,pulls,gists"
  }
}
```

### VS Code Integration Issues

**Problem**: Server not appearing in VS Code
```
Server 'github' not found
```

**Solution**: 
1. Ensure VS Code version is 1.101+
2. Reload VS Code: Cmd/Ctrl + Shift + P → "Developer: Reload Window"
3. Check settings location: `.vscode/settings.json` or user settings
4. Enable Agent mode in Copilot Chat

### Environment Variable Not Resolved

**Problem**: Token showing as literal string
```
Authorization: Bearer ${GITHUB_PAT}
```

**Solution**: Host-specific syntax
- Some hosts use `${GITHUB_PAT}`
- Others use `${input:github_token}`
- Some require hardcoded values
- Check host documentation for variable syntax

### Testing Connection

Verify server is working:
```
List repositories in github organization
```

If successful, you should see a list of repositories. If not, check logs in your MCP host.
