---
name: mcp-servers-essential-reference
description: Reference guide for installing and configuring 50+ essential MCP servers across dev, data, ops, productivity, fintech, web3, trading, and creator tools
triggers:
  - how do I install MCP servers
  - show me essential MCP servers
  - configure MCP server for Claude
  - what MCP servers should I use
  - set up github MCP server
  - MCP server best practices
  - troubleshoot MCP server connection
  - which MCP servers for development
---

# MCP Servers Essential Reference

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill provides expertise in selecting, installing, and configuring MCP (Model Context Protocol) servers for AI coding agents like Claude Code, Cursor, Codex, and Gemini CLI. It covers 50 essential servers across 9 categories with installation commands, security best practices, and troubleshooting.

## What MCP Servers Are

The Model Context Protocol (MCP) is a standardized interface that allows AI agents to connect to external tools and services. Instead of each AI model implementing custom integrations, MCP provides a "USB-C port for AI" — one protocol that both agents and tools speak.

**Key concepts:**
- **Server**: A service that exposes tools/resources via MCP (e.g., GitHub API, database access)
- **Transport**: How the agent connects (stdio for local, HTTP/SSE for hosted)
- **Tool**: A function the agent can call (e.g., `search_github`, `run_sql_query`)
- **Resource**: Read-only data the agent can access (e.g., file contents, API docs)

## The Three Critical Rules

Before installing any MCP server:

1. **Don't install more than 5-7 servers** — Tool bloat degrades agent performance and reasoning quality
2. **Treat every server like untrusted code** — Pin versions, scope tokens read-only, review source
3. **Never enable writes to production** — Point at replicas, snapshots, or staging environments

## Installation Methods by Agent

### Claude Code (Desktop)

```bash
# Local server (stdio transport)
claude mcp add github -- npx -y @modelcontextprotocol/server-github

# Hosted server (HTTP transport)
claude mcp add linear --transport http

# List installed servers
claude mcp list

# Remove a server
claude mcp remove github
```

Config file: `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)

### Codex

```bash
# Add server
codex mcp add postgres -- npx -y @modelcontextprotocol/server-postgres

# Edit config directly
vim ~/.codex/config.toml
```

### Gemini CLI

Edit `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

## Manual Configuration Format

All agents use similar JSON structure for `mcpServers`:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@scope/package-name"],
      "env": {
        "API_KEY": "${ENV_VAR_NAME}"
      }
    }
  }
}
```

**For hosted servers:**

```json
{
  "mcpServers": {
    "linear": {
      "url": "https://mcp.linear.app",
      "transport": "http"
    }
  }
}
```

## Category-Based Server Selection

### Core Development (Pick 3-4)

**GitHub** (Official, hosted)
```bash
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```
Environment: `GITHUB_PERSONAL_ACCESS_TOKEN`

Capabilities: Search repos/issues/PRs, create issues, manage PRs, push commits

**Filesystem** (Official, local)
```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/allowed/directory
```
⚠️ **Security**: Only expose specific directories, never `/` or `~`

**Brave Search** (Official, local)
```bash
claude mcp add brave-search -- npx -y @modelcontextprotocol/server-brave-search
```
Environment: `BRAVE_API_KEY`

Use case: Web search without leaving the agent context

**Playwright** (Official, local)
```bash
claude mcp add playwright -- npx -y @executeautomation/playwright-mcp-server
```
Capabilities: Browser automation, screenshot capture, web scraping

### Databases & Backend (Pick 1)

**PostgreSQL** (Official, local)
```bash
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres postgresql://user:pass@localhost/dbname
```
⚠️ **Critical**: Use read-only user or point at replica

**Supabase** (Official, hosted)
```bash
claude mcp add supabase --transport http
```
Requires: Supabase project URL and service role key

**SQLite** (Community, local)
```bash
claude mcp add sqlite -- npx -y @benborla/mcp-server-sqlite /path/to/database.db
```

### Ops & Infrastructure

**AWS** (Official, local)
```bash
claude mcp add aws -- npx -y @modelcontextprotocol/server-aws-kb-retrieval-server
```
Environment: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`

**Cloudflare** (Official, hosted)
```bash
claude mcp add cloudflare --transport http
```
⚠️ **Dangerous**: Can modify DNS, workers, R2 buckets

**Kubernetes** (Community, local)
```bash
claude mcp add kubernetes -- npx -y @strowk/mcp-k8s-go
```
Environment: `KUBECONFIG` path
⚠️ **Production risk**: Can delete pods, services, deployments

### Productivity & Team Tools

**Slack** (Community, local)
```bash
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack
```
Environment: `SLACK_BOT_TOKEN`, `SLACK_TEAM_ID`

**Linear** (Official, hosted)
```bash
claude mcp add linear --transport http
```
OAuth flow via browser

**Notion** (Official, hosted)
```bash
claude mcp add notion --transport http
```

**Google Drive** (Official, local - archived)
```bash
claude mcp add gdrive -- npx -y @modelcontextprotocol/server-gdrive
```
Requires OAuth credentials JSON

### Fintech & Payments (⚠️ All flagged as money-moving)

**Stripe** (Official, hosted/local)
```bash
claude mcp add stripe -- npx -y @stripe/mcp
```
Environment: `STRIPE_SECRET_KEY` (use test keys first)

**Plaid** (Official, hosted)
Documentation: Check Plaid developer portal for latest MCP endpoint

⚠️ **All fintech servers**: Start in sandbox/test mode, never use production keys until you've verified agent behavior

### Web3 & Blockchain (⚠️ Irreversible transactions)

**Base MCP** (Official, hosted)
```bash
claude mcp add base --transport http
```
Capabilities: Deploy contracts, send transactions, query onchain data

⚠️ **Critical**: Transactions are irreversible. Use testnet first.

**The Graph** (Community, local - read-only)
```bash
claude mcp add thegraph -- npx -y @rainprotocol/mcp-subgraph
```
Safe for querying blockchain data via GraphQL

**Dune** (Community, local - read-only)
Environment: `DUNE_API_KEY`
Use case: SQL queries against blockchain data

### Trading & Markets

**Alpaca** (Official, local)
```bash
claude mcp add alpaca -- npx -y @alpacahq/mcp-server
```
Environment: `APCA_API_KEY_ID`, `APCA_API_SECRET_KEY`, `APCA_API_BASE_URL`

⚠️ **Paper trade first**: Set `APCA_API_BASE_URL=https://paper-api.alpaca.markets`

**Polygon.io** (Official, local - read-only)
```bash
claude mcp add polygon -- npx -y @modelcontextprotocol/server-polygon
```
Environment: `POLYGON_API_KEY`

**CoinGecko** (Official, hosted - read-only)
Safe for market data queries

### Creator Tools & Media

**ElevenLabs** (Official, local)
```bash
claude mcp add elevenlabs -- npx -y @modelcontextprotocol/server-elevenlabs
```
Environment: `ELEVENLABS_API_KEY`

**Figma** (Official, hosted/local)
```bash
claude mcp add figma -- npx -y @modelcontextprotocol/server-figma
```
Environment: `FIGMA_PERSONAL_ACCESS_TOKEN`

**YouTube** (Community, local)
Environment: `YOUTUBE_API_KEY`
Capabilities: Search videos, retrieve metadata, manage playlists

### Reasoning & Memory

**Sequential Thinking** (Official, local)
```bash
claude mcp add sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```
Forces step-by-step reasoning for complex problems

**Memory** (Official, local)
```bash
claude mcp add memory -- npx -y @modelcontextprotocol/server-memory
```
Persistent knowledge graph across sessions

## Real-World Configuration Examples

### Minimal Dev Setup (5 servers)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/you/projects"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://readonly:${DB_PASSWORD}@localhost/myapp_dev"]
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

### Creator Pipeline (4 servers)

```json
{
  "mcpServers": {
    "elevenlabs": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-elevenlabs"],
      "env": {
        "ELEVENLABS_API_KEY": "${ELEVENLABS_API_KEY}"
      }
    },
    "figma": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-figma"],
      "env": {
        "FIGMA_PERSONAL_ACCESS_TOKEN": "${FIGMA_TOKEN}"
      }
    },
    "youtube": {
      "command": "npx",
      "args": ["-y", "@benborla/mcp-server-youtube"],
      "env": {
        "YOUTUBE_API_KEY": "${YOUTUBE_API_KEY}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/you/content"]
    }
  }
}
```

### Team Productivity (5 servers)

```json
{
  "mcpServers": {
    "linear": {
      "url": "https://mcp.linear.app",
      "transport": "http"
    },
    "notion": {
      "url": "https://mcp.notion.so",
      "transport": "http"
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}",
        "SLACK_TEAM_ID": "${SLACK_TEAM_ID}"
      }
    },
    "gdrive": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-gdrive"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/credentials.json"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

## Security Best Practices

### Token Scoping (GitHub Example)

Create read-only token for initial testing:
- Permissions: `repo:status`, `public_repo` (read)
- No write permissions until you've audited agent behavior

Upgrade to write access only when needed:
- Add: `repo` (full control)
- Monitor first 10-20 agent actions manually

### Database Access Patterns

**Create read-only user:**
```sql
-- PostgreSQL
CREATE USER mcp_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE myapp TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_readonly;
```

**Connection string:**
```
postgresql://mcp_readonly:${MCP_DB_PASSWORD}@localhost:5432/myapp
```

### File System Isolation

```json
{
  "filesystem": {
    "command": "npx",
    "args": [
      "-y",
      "@modelcontextprotocol/server-filesystem",
      "/Users/you/safe-workspace",
      "--readonly"
    ]
  }
}
```

Never expose:
- `/` (system root)
- `~` (home directory)
- `/etc`, `/usr`, `/bin`
- `.ssh`, `.aws`, `.config` directories

## Troubleshooting

### Server Won't Start

**Check logs:**
```bash
# Claude Desktop logs (macOS)
tail -f ~/Library/Logs/Claude/mcp-server-*.log

# Check if process is running
ps aux | grep npx
```

**Common issues:**
1. **Missing environment variable**: Check `.env` or shell profile
2. **Port conflict**: Another server using same port (hosted servers)
3. **npx version mismatch**: Update Node.js to LTS version
4. **Permission denied**: File system server needs directory read/write access

**Fix:**
```bash
# Reload environment
source ~/.zshrc  # or ~/.bashrc

# Clear npx cache
npx clear-npx-cache

# Reinstall server
claude mcp remove server-name
claude mcp add server-name -- npx -y @scope/package
```

### Agent Can't See Tools

**Verify server is connected:**
```bash
claude mcp list
# Should show "connected" status
```

**Check agent prompt:**
- Some agents require explicit mention: "use the github server to..."
- Tool discovery happens on session start — restart the agent

**Test with simple command:**
- "List all available tools"
- "What can you do with the github server?"

### Performance Degradation

**Symptom**: Agent responses slow, frequent timeouts

**Diagnosis:**
```bash
# Count active servers
claude mcp list | grep connected | wc -l

# Check context window usage (if agent provides)
# Claude: look for "context: 45% used" in UI
```

**Fix**: Disable servers you're not actively using
```bash
claude mcp remove server1
claude mcp remove server2
# Keep only 3-5 essential servers
```

### Authentication Errors

**GitHub "Bad credentials":**
```bash
# Verify token has required scopes
curl -H "Authorization: token ${GITHUB_TOKEN}" https://api.github.com/user

# Regenerate token at https://github.com/settings/tokens
# Update environment variable
export GITHUB_TOKEN="ghp_newtoken"
```

**Database "Connection refused":**
```bash
# Check database is running
pg_isready -h localhost -p 5432

# Test connection manually
psql postgresql://user:pass@localhost/dbname
```

**Hosted server OAuth failures:**
- Clear browser cache/cookies
- Try OAuth flow in incognito window
- Check redirect URL matches agent's expected callback

### Version Pinning (Recommended)

Avoid breaking changes by pinning versions:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-github@0.5.0"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Update workflow:**
```bash
# Test in isolated config first
cp claude_desktop_config.json claude_desktop_config.json.backup

# Update one server
# Edit config: @scope/package@0.5.0 → @scope/package@0.6.0

# Restart agent and test core workflows

# If broken, revert:
mv claude_desktop_config.json.backup claude_desktop_config.json
```

## Common Patterns

### Conditional Server Loading

Use environment variable to toggle expensive servers:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "sh",
      "args": [
        "-c",
        "[ \"$ENABLE_BROWSER\" = \"true\" ] && npx -y @executeautomation/playwright-mcp-server || echo 'disabled'"
      ]
    }
  }
}
```

Usage:
```bash
# Enable for this session
export ENABLE_BROWSER=true
claude
```

### Multi-Environment Configs

Maintain separate configs for different workflows:

```bash
# ~/.config/claude/
├── config-dev.json       # GitHub, Postgres, Filesystem
├── config-trading.json   # Polygon, CoinGecko, Alpaca (paper)
├── config-creator.json   # ElevenLabs, Figma, YouTube
└── config-prod.json      # Minimal read-only servers

# Symlink active config
ln -sf ~/.config/claude/config-dev.json ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Monitoring Agent Actions

For high-risk servers (databases, cloud infra), log all operations:

**PostgreSQL audit log:**
```sql
-- Enable query logging
ALTER SYSTEM SET log_statement = 'all';
ALTER SYSTEM SET log_directory = 'pg_log';
SELECT pg_reload_conf();

-- Monitor in real-time
tail -f /var/lib/postgresql/data/pg_log/postgresql-*.log
```

**GitHub webhook for audit trail:**
Create webhook to log all agent-initiated changes to separate audit system.

## Quick Reference: 50 Servers by Risk Level

### ✅ Safe (Read-only, no credentials)
- Sequential Thinking, Memory, Brave Search (with API key)

### ⚠️ Low Risk (Read-only with auth)
- GitHub (read scope), Linear, Notion, Sentry, Polygon.io, CoinGecko, The Graph, Dune

### 🟡 Medium Risk (Can modify data)
- Filesystem (scoped), Slack, Gmail, Google Calendar, Figma, ElevenLabs, YouTube

### 🔴 High Risk (Infrastructure/money)
- **Databases**: Postgres, Supabase, Neon, SQLite, Redis (write access)
- **Cloud**: AWS, Cloudflare, Vercel, Kubernetes, Docker Hub
- **Fintech**: Stripe, PayPal, Plaid, QuickBooks
- **Trading**: Alpaca, CCXT
- **Web3**: Base, Solana Agent Kit, Thirdweb (testnet OK)

**Rule**: Start all 🔴 servers in read-only/sandbox/testnet mode. Graduate to write access only after 20+ successful manual-approved operations.

## Resources

- **Official MCP Docs**: https://modelcontextprotocol.io
- **Server Registry**: https://github.com/modelcontextprotocol/servers
- **Security Audit**: Review 2026 Q1 security findings before installing community servers
- **SDK Downloads**: npm `@modelcontextprotocol/sdk` (~97M monthly downloads as of March 2026)

**Remember**: This is a menu, not a mandate. Pick 3-5 servers that match your actual workflow, secure them properly, and ship.
