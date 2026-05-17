---
name: awesome-mcp-servers-discovery
description: Discover, explore, and learn about MCP (Model Context Protocol) servers from a curated list of 6000+ implementations across 30+ categories
triggers:
  - find MCP servers for database integration
  - show me available MCP servers
  - what MCP servers exist for file systems
  - how do I discover MCP servers
  - list MCP servers by category
  - search for official MCP implementations
  - browse community MCP servers
  - find tools to manage MCP servers
---

# Awesome MCP Servers Discovery

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill enables discovery and exploration of the Awesome MCP Servers repository, a curated list of Model Context Protocol (MCP) servers, tools, frameworks, clients, and utilities. The repository catalogs 6000+ MCP server implementations across 30+ categories including file systems, databases, cloud platforms, AI services, and more.

## What is This Repository

**Awesome MCP Servers** is a community-driven collection that helps developers:
- Discover MCP servers by category (databases, file systems, cloud storage, etc.)
- Find official vs. community implementations
- Access reference servers and tutorials
- Learn security best practices for MCP
- Discover tools to install and manage MCP servers

The repository includes:
- **Reference Servers**: Example implementations from the official MCP protocol
- **Official Servers**: Vendor-maintained implementations (marked with ⭐)
- **Community Servers**: 6000+ community-built implementations
- **Tools & Utilities**: CLI tools, managers, and frameworks for MCP
- **Multi-language documentation**: English, 简体中文, 繁體中文, 日本語, 한국어

## Installation & Access

### Accessing the Repository

```bash
# Clone the repository
git clone https://github.com/YuzeHao2023/Awesome-MCP-Servers.git
cd Awesome-MCP-Servers

# View the full Excel list of 6000+ servers
# Open Full-List-of-MCP-Servers.xlsx
```

### Installing MCP Server Management Tools

```bash
# Install mcp-get (CLI tool for installing MCP servers)
npm install -g @michaellatman/mcp-get

# Install mcp-cli (inspector for MCP servers)
npm install -g @wong2/mcp-cli

# Install yamcp (workspace manager)
npm install -g yamcp
```

## Key Categories

### 📂 File Systems
Servers providing file system access with configurable permissions.

**Official Reference**:
```bash
# Filesystem server from modelcontextprotocol
npx @modelcontextprotocol/server-filesystem <allowed-directory>
```

**Community Examples**:
- `mcp-everything-search` - Everything Search integration
- `fast-filesystem-mcp` - Fast filesystem implementation
- `llm-context` - Context-aware file access

### 🗄️ Databases
Database access with schema inspection and query capabilities.

**PostgreSQL Example**:
```bash
# Official PostgreSQL MCP server
npx @modelcontextprotocol/server-postgres postgresql://user:password@localhost/dbname
```

**Other Database Servers**:
- SQLite: `@modelcontextprotocol/server-sqlite`
- MongoDB: `@kiliczsh/mcp-mongo-server`
- Redis: Official from `redis/mcp-redis`
- BigQuery: `@LucasHild/mcp-server-bigquery`
- Neon (⭐): `@neondatabase/mcp-server-neon`

### 🔄 Version Control
Git and version control integration.

**GitHub Official Server**:
```bash
# Install GitHub MCP server
npm install @github/github-mcp-server

# Configure in Claude Desktop (claude_desktop_config.json)
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@github/github-mcp-server"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### ☁️ Cloud Storage
Access to cloud storage platforms.

**Google Drive**:
```bash
npx @modelcontextprotocol/server-gdrive
```

**Box (Official ⭐)**:
- Documentation: https://developer.box.com/guides/box-mcp/

### 📦 Sandbox & Virtualization
Secure sandbox environments for code execution.

**E2B (Official ⭐)**:
```bash
npm install @e2b/mcp-server

# Configure
{
  "mcpServers": {
    "e2b": {
      "command": "npx",
      "args": ["-y", "@e2b/mcp-server"],
      "env": {
        "E2B_API_KEY": "${E2B_API_KEY}"
      }
    }
  }
}
```

**Microsandbox (Official ⭐)**:
- Secure sandbox for code execution
- GitHub: https://github.com/microsandbox/microsandbox

### ⚡ Cloud Platforms
Integration with cloud providers.

**Cloudflare (Official ⭐)**:
```bash
npm install @cloudflare/mcp-server-cloudflare

# Configure
{
  "mcpServers": {
    "cloudflare": {
      "command": "npx",
      "args": ["-y", "@cloudflare/mcp-server-cloudflare"],
      "env": {
        "CLOUDFLARE_API_TOKEN": "${CLOUDFLARE_API_TOKEN}"
      }
    }
  }
}
```

### 🤖 AI Services
AI and LLM service integrations.

**Notable Servers**:
- Anthropic: Official Claude API integration
- OpenAI: OpenAI API integration
- Hugging Face: Model inference and datasets
- AgentQL: Web scraping for AI agents

### 💻 Development Tools
Development and coding assistance.

**Memory Server (Reference)**:
```bash
# Persistent memory across conversations
npx @modelcontextprotocol/server-memory
```

**Sequential Thinking (Reference)**:
```bash
# Extended reasoning capabilities
npx @modelcontextprotocol/server-sequentialthinking
```

## Using Server Management Tools

### mcp-get: Install and Manage Servers

```bash
# Search for servers
mcp-get search database

# Install a server
mcp-get install @modelcontextprotocol/server-postgres

# List installed servers
mcp-get list

# Remove a server
mcp-get remove @modelcontextprotocol/server-postgres
```

### mcp-cli: Inspect Servers

```bash
# Inspect a running MCP server
mcp-cli inspect http://localhost:3000

# List available tools from a server
mcp-cli tools @modelcontextprotocol/server-filesystem /path/to/dir

# Test a server
mcp-cli test @modelcontextprotocol/server-sqlite database.db
```

### yamcp: Workspace Manager

```bash
# Initialize a workspace
yamcp init

# Add a server to workspace
yamcp add filesystem --path /allowed/directory

# Start all workspace servers
yamcp start

# Stop all servers
yamcp stop
```

## Configuration Patterns

### Claude Desktop Configuration

Located at:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`
- Linux: `~/.config/Claude/claude_desktop_config.json`

**Multi-Server Configuration**:
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/projects"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@github/github-mcp-server"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### Environment Variables Pattern

```bash
# .env file (never commit this)
DATABASE_URL=postgresql://user:pass@localhost/db
GITHUB_TOKEN=ghp_xxxxxxxxxxxxx
OPENAI_API_KEY=sk-xxxxxxxxxxxxx
CLOUDFLARE_API_TOKEN=xxxxxxxxxxxxx

# Load in shell
export $(cat .env | xargs)
```

## Security Best Practices

⚠️ **Critical Security Warnings**:

1. **Arbitrary Code Execution**: MCP servers can execute code with your user permissions
2. **System Access**: Full access to files, network, and system resources
3. **Prompt Injection**: Malicious prompts could trigger unintended actions
4. **Data Exposure**: Sensitive data may be accessed or leaked

**Best Practices**:

```bash
# 1. Use official implementations when available (⭐ marked)
# 2. Review code before installation
git clone <mcp-server-repo>
# Review the code thoroughly

# 3. Run in isolated containers (Docker example)
docker run --rm -it \
  -v /allowed/path:/data:ro \
  --network=none \
  mcp-server-image

# 4. Limit permissions
chmod 500 /path/to/mcp-server  # Read + execute only
chown root:mcp /path/to/mcp-server

# 5. Monitor activity
# Enable logging in server config
{
  "mcpServers": {
    "myserver": {
      "command": "npx",
      "args": ["-y", "mcp-server", "--log-level", "debug"],
      "logFile": "/var/log/mcp/myserver.log"
    }
  }
}
```

## Finding Servers by Use Case

### Database Query & Analysis

```bash
# Search the Excel file or README for "database"
# Official options:
- PostgreSQL: @modelcontextprotocol/server-postgres
- SQLite: @modelcontextprotocol/server-sqlite
- MongoDB: @kiliczsh/mcp-mongo-server
- Redis: redis/mcp-redis (⭐)
- BigQuery: @LucasHild/mcp-server-bigquery
```

### File Operations

```bash
# Reference implementation
npx @modelcontextprotocol/server-filesystem /path/to/allowed/directory

# Fast implementation
npm install fast-filesystem-mcp

# With search capabilities
npm install mcp-everything-search
```

### Cloud Platform Management

```bash
# AWS
# Check awslabs repositories under modelcontextprotocol

# Cloudflare (⭐)
npm install @cloudflare/mcp-server-cloudflare

# Azure
# Check Azure repositories for official MCP servers
```

### Web Scraping & Search

```bash
# Brave Search (⭐)
npm install @modelcontextprotocol/server-brave-search

# AgentQL
npm install @tinyfish-io/agentql-mcp

# Fetch (reference)
npx @modelcontextprotocol/server-fetch
```

## Common Patterns

### Pattern: Adding a New Server to Your Setup

```bash
# 1. Find the server in the repository
# Browse categories in README or search Full-List-of-MCP-Servers.xlsx

# 2. Review the server (security check)
git clone <server-repo-url>
cd <server-repo>
cat README.md  # Check documentation
cat package.json  # Check dependencies

# 3. Install via mcp-get or npm
mcp-get install <server-package>
# OR
npm install <server-package>

# 4. Configure in Claude Desktop
# Edit claude_desktop_config.json
{
  "mcpServers": {
    "newserver": {
      "command": "npx",
      "args": ["-y", "<server-package>", "<args>"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}

# 5. Restart Claude Desktop
# Server should now be available
```

### Pattern: Testing a Server Before Deployment

```bash
# Use mcp-cli to test
mcp-cli test <server-package> <args>

# Example: Test filesystem server
mcp-cli test @modelcontextprotocol/server-filesystem /tmp

# Check available tools
mcp-cli tools <server-package> <args>

# Inspect server metadata
mcp-cli inspect <server-package>
```

### Pattern: Creating a Multi-Environment Setup

```bash
# Development config (dev_claude_config.json)
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home/user/dev"]
    },
    "postgres": {
      "env": {
        "DATABASE_URL": "postgresql://localhost/dev_db"
      }
    }
  }
}

# Production config (prod_claude_config.json)
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/var/www"]
    },
    "postgres": {
      "env": {
        "DATABASE_URL": "${PROD_DATABASE_URL}"
      }
    }
  }
}

# Swap configs as needed
ln -sf ~/dev_claude_config.json ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

## Troubleshooting

### Server Not Appearing in Claude

```bash
# 1. Check config syntax
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json | jq .

# 2. Verify server installation
npx <server-package> --help

# 3. Check server logs
# Add logging to config:
{
  "mcpServers": {
    "myserver": {
      "command": "npx",
      "args": ["-y", "server", "--verbose"]
    }
  }
}

# 4. Restart Claude Desktop completely
killall "Claude"
open -a "Claude"
```

### Server Crashes or Errors

```bash
# 1. Test server standalone
npx <server-package> <args>

# 2. Check permissions
ls -la $(which <server-binary>)

# 3. Verify environment variables are set
echo $GITHUB_TOKEN  # Should show token

# 4. Check for conflicting servers
# Remove other servers temporarily from config

# 5. Update server to latest version
npm update <server-package>
```

### Finding a Server for Specific Use Case

```bash
# 1. Search README by category
# Browse https://github.com/YuzeHao2023/Awesome-MCP-Servers

# 2. Search Excel file
# Open Full-List-of-MCP-Servers.xlsx
# Use Excel search/filter features

# 3. Use GitHub search
# Search within repository for keywords

# 4. Check official implementations first
# Look for ⭐ markers in README

# 5. Review community discussions
# Check Issues and Discussions tabs on GitHub
```

### Performance Issues

```bash
# 1. Limit server scope
# Example: Restrict filesystem access
{
  "filesystem": {
    "args": ["-y", "@modelcontextprotocol/server-filesystem", "/specific/path"]
  }
}

# 2. Use caching where available
# Check server documentation for cache options

# 3. Monitor resource usage
top -p $(pgrep -f mcp-server)

# 4. Consider lighter alternatives
# Check category for alternative implementations
```

## Reference: Key Server Categories

| Category | Count | Notable Servers |
|----------|-------|-----------------|
| 📂 File Systems | ~50 | filesystem (⭐), everything-search, fast-filesystem-mcp |
| 🗄️ Databases | ~100 | postgres (⭐), sqlite (⭐), mongodb, redis (⭐), bigquery |
| 🔄 Version Control | ~30 | github (⭐), gitlab, git (⭐) |
| ☁️ Cloud Storage | ~40 | gdrive (⭐), box (⭐), ms-365 |
| ⚡ Cloud Platforms | ~60 | cloudflare (⭐), aws, azure, google |
| 🤖 AI Services | ~80 | anthropic, openai, huggingface |
| 💻 Dev Tools | ~200 | memory (⭐), sequential-thinking (⭐), docker |
| 🔍 Search & Web | ~150 | brave-search (⭐), fetch (⭐), agentql |
| 💬 Communication | ~40 | slack, linear, atlassian |
| 📈 Monitoring | ~30 | sentry (⭐), metoro, signoz |

## Additional Resources

- **Official MCP Documentation**: https://modelcontextprotocol.io/
- **Discord Community**: https://glama.ai/mcp/discord
- **Reddit**: https://www.reddit.com/r/mcp
- **Full Server List (Excel)**: `Full-List-of-MCP-Servers.xlsx` in repository
- **Tutorials**: Search repository for "Tutorials" section

## Contributing to the List

If you want to add a server to Awesome MCP Servers:

```bash
# 1. Fork the repository
gh repo fork YuzeHao2023/Awesome-MCP-Servers

# 2. Add your server to appropriate category in README.md
# Format: - Name — URL

# 3. Create pull request
git checkout -b add-my-server
git add README.md
git commit -m "Add my-server to Category"
git push origin add-my-server
gh pr create
```
