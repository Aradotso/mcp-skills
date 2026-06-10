---
name: 50-essential-mcp-servers-reference
description: Expert guide to selecting, installing, and configuring MCP servers for Claude, Gemini, and Codex with security best practices
triggers:
  - show me essential mcp servers
  - how do I choose mcp servers
  - install mcp server for database
  - what mcp servers should I use
  - configure mcp servers for claude
  - secure mcp server setup
  - mcp server for github integration
  - best practices for mcp servers
---

# 50 Essential MCP Servers Reference

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill provides expert knowledge of the **50 Essential MCP Servers** curated list — a reference guide for selecting, installing, and configuring Model Context Protocol (MCP) servers for AI coding agents like Claude Code, Gemini CLI, and Codex.

## What This Project Provides

The 50 Essential MCP Servers is a curated reference of battle-tested MCP servers organized by category:

1. **Core Dev** — GitHub, Context7, Playwright, Filesystem, Brave Search
2. **Databases & Backend** — Postgres, Supabase, Neon, SQLite, Redis
3. **Ops, Infra & Cloud** — Cloudflare, Vercel, AWS, Docker Hub, Sentry, Kubernetes
4. **Corporate/Team & Productivity** — Slack, Linear, Notion, Jira, Google Drive, Gmail
5. **Payments & Fintech** — Stripe, PayPal, Plaid, QuickBooks
6. **Blockchain & Web3** — Base, Solana Agent Kit, Thirdweb, The Graph, Dune
7. **Trading & Markets** — Alpaca, CCXT, Polygon.io, CoinGecko, TradingView
8. **Creator Tools** — Higgsfield, DaVinci Resolve, Figma, ElevenLabs, Blender
9. **Reasoning & Memory** — Sequential Thinking, Memory

## The Three Golden Rules

**Before installing ANY MCP server, follow these rules:**

1. **Don't install 50 servers** — More than 5-7 connected servers causes tool bloat, making your agent slower and dumber. Pick 3-5 that match your actual workflow.

2. **Treat every server like a CLI from a stranger's GitHub** — Pin versions, scope tokens to read-only initially, prefer vendor-official over community forks.

3. **Never enable writes against production from an agent loop** — Point at read replicas, staging environments, or snapshots.

## Installation Syntax by Agent

### Claude Code

```bash
# Local server (npx/uvx)
claude mcp add <name> npx -y <package>

# Hosted server (https endpoint)
claude mcp add <name> --transport http <url>
```

### Codex

```bash
# Local server
codex mcp add <name> -- npx -y <package>
```

Or edit `~/.codex/config.toml`:

```toml
[mcp.servers.<name>]
command = "npx"
args = ["-y", "<package>"]
```

### Gemini CLI

Edit `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "<name>": {
      "command": "npx",
      "args": ["-y", "<package>"]
    }
  }
}
```

## Common Installation Patterns

### Pattern 1: Core Development Stack (5 servers)

**Best for:** Software engineers building web applications

```bash
# Claude Code
claude mcp add github npx -y @modelcontextprotocol/server-github
claude mcp add postgres npx -y @modelcontextprotocol/server-postgres
claude mcp add playwright npx -y @executeautomation/playwright-mcp-server
claude mcp add filesystem npx -y @modelcontextprotocol/server-filesystem
claude mcp add sentry --transport http https://mcp.sentry.io
```

**Configuration:**

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
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user@localhost:5432/dbname"
      }
    }
  }
}
```

### Pattern 2: Creator Pipeline (5 servers)

**Best for:** Content creators, video editors, designers

```bash
claude mcp add higgsfield --transport http https://mcp.higgsfield.ai
claude mcp add elevenlabs npx -y @modelcontextprotocol/server-elevenlabs
claude mcp add figma npx -y @modelcontextprotocol/server-figma
claude mcp add youtube npx -y @angheljf/server-youtube
claude mcp add davinci npx -y davinci-resolve-mcp-server
```

### Pattern 3: Web3/Trading (Read-Only Start)

**Best for:** Blockchain developers, traders (start read-only)

```bash
# Read-only market data first
claude mcp add coingecko --transport http https://mcp.coingecko.com
claude mcp add thegraph npx -y @modelcontextprotocol/server-thegraph
claude mcp add dune npx -y dune-mcp-server
claude mcp add polygon npx -y @polygon-io/mcp-server

# Only add write-capable servers after testing read-only
# claude mcp add base --transport http https://mcp.base.org
```

### Pattern 4: Team Productivity (4 servers)

**Best for:** Product managers, operations teams

```bash
claude mcp add slack npx -y @modelcontextprotocol/server-slack
claude mcp add linear --transport http https://mcp.linear.app
claude mcp add notion --transport http https://mcp.notion.com
claude mcp add gdrive npx -y @modelcontextprotocol/server-gdrive
```

## Security Configuration Best Practices

### Read-Only Tokens (Recommended Starting Point)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN_READONLY}"
      }
    }
  }
}
```

**GitHub token scopes (read-only):**
- `repo:status`
- `public_repo` (if only public repos)
- `read:org`
- `read:user`

**Avoid initially:**
- `repo` (full access)
- `delete_repo`
- `admin:*`

### Database Connections (Point to Replicas)

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://readonly_user:${DB_PASSWORD}@read-replica.example.com:5432/dbname"
      }
    }
  }
}
```

**PostgreSQL read-only user:**

```sql
CREATE ROLE mcp_readonly WITH LOGIN PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE mydb TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_readonly;
```

### Financial/Trading Servers (Paper Trading First)

```json
{
  "mcpServers": {
    "alpaca": {
      "command": "npx",
      "args": ["-y", "@alpacahq/mcp-server"],
      "env": {
        "ALPACA_API_KEY": "${ALPACA_PAPER_KEY}",
        "ALPACA_SECRET_KEY": "${ALPACA_PAPER_SECRET}",
        "ALPACA_PAPER": "true"
      }
    }
  }
}
```

## Key Server Examples

### GitHub Server

**Install:**
```bash
claude mcp add github npx -y @modelcontextprotocol/server-github
```

**Capabilities:**
- Search repositories, issues, PRs
- Create/update issues and PRs
- Manage branches, commits
- Read/write files in repos

**Example prompts:**
- "Show me open issues in my repo labeled 'bug'"
- "Create a new branch called feature/user-auth"
- "Search for TypeScript repos with MCP in the name"

### Supabase Server

**Install:**
```bash
claude mcp add supabase --transport http https://mcp.supabase.com
```

**Requires:** Supabase project URL and anon/service key

**Capabilities:**
- Query tables via REST API
- Invoke Edge Functions
- Manage storage buckets
- Real-time subscriptions

**Configuration:**
```json
{
  "supabase": {
    "transport": "http",
    "url": "https://mcp.supabase.com",
    "env": {
      "SUPABASE_URL": "${SUPABASE_PROJECT_URL}",
      "SUPABASE_KEY": "${SUPABASE_ANON_KEY}"
    }
  }
}
```

### Playwright Server

**Install:**
```bash
claude mcp add playwright npx -y @executeautomation/playwright-mcp-server
```

**Capabilities:**
- Browser automation
- Screenshot/PDF generation
- Form filling
- Web scraping

**Example use:**
- "Take a screenshot of example.com"
- "Fill out the login form on staging.myapp.com"
- "Extract all product prices from this page"

### Context7 Server

**Install:**
```bash
claude mcp add context7 npx -y @context7/mcp-server
# or
claude mcp add context7 --transport http https://mcp.context7.com
```

**Capabilities:**
- Semantic code search
- Codebase understanding
- Dependency analysis

**Configuration:**
```json
{
  "context7": {
    "command": "npx",
    "args": ["-y", "@context7/mcp-server"],
    "env": {
      "CONTEXT7_API_KEY": "${CONTEXT7_KEY}",
      "WORKSPACE_PATH": "/path/to/project"
    }
  }
}
```

### Stripe Server

**Install:**
```bash
# Hosted (recommended)
claude mcp add stripe --transport http https://mcp.stripe.com

# Local
claude mcp add stripe npx -y @stripe/mcp-server
```

**⚠️ WARNING: Money-moving server — use test mode initially**

**Configuration:**
```json
{
  "stripe": {
    "transport": "http",
    "url": "https://mcp.stripe.com",
    "env": {
      "STRIPE_SECRET_KEY": "${STRIPE_TEST_KEY}"
    }
  }
}
```

**Start with test keys:**
- Test key: `sk_test_...`
- Live key: `sk_live_...` (only after thorough testing)

## Version Pinning (Recommended)

Instead of using latest (`-y`), pin to specific versions:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-github@1.2.3"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

Check installed version:
```bash
npm info @modelcontextprotocol/server-github version
```

## Troubleshooting

### Issue: Agent is slow or gives irrelevant tool suggestions

**Cause:** Too many connected servers (tool bloat)

**Fix:** Reduce to 3-5 most-used servers
```bash
# List current servers
claude mcp list

# Remove unused ones
claude mcp remove <server-name>
```

### Issue: "Connection refused" or "Server not found"

**Cause:** Server package not installed or wrong command

**Fix:** Verify package exists
```bash
npm info <package-name>
```

For hosted servers, check official docs for current URL.

### Issue: Authentication errors

**Cause:** Missing or incorrect environment variables

**Fix:** Check env vars are set
```bash
# Check if env var exists
echo $GITHUB_TOKEN

# Set if missing
export GITHUB_TOKEN="ghp_..."
```

Restart agent after setting env vars.

### Issue: Database queries fail or timeout

**Cause:** Connection string incorrect or network issue

**Fix:**
```bash
# Test connection directly
psql "${POSTGRES_CONNECTION_STRING}" -c "SELECT 1"

# For read replicas, ensure replica is reachable
ping read-replica.example.com
```

### Issue: Server works in one agent but not another

**Cause:** Different config file formats

**Fix:** Each agent has its own config location:
- Claude Code: `~/.config/claude/mcp_config.json` (or similar)
- Codex: `~/.codex/config.toml`
- Gemini: `~/.gemini/settings.json`

Verify the config syntax matches the agent's expected format.

## Quick Selection Guide by Role

### Software Engineer
```
github + postgres/supabase + playwright + filesystem + sentry
```

### DevOps/SRE
```
aws + kubernetes + cloudflare + sentry + docker-hub
```

### Product Manager
```
linear/jira + notion + slack + google-drive
```

### Data Analyst
```
postgres + supabase + sqlite + google-drive + notion
```

### Content Creator
```
higgsfield + elevenlabs + figma + youtube + canva
```

### Web3 Developer (read-only first)
```
coingecko + thegraph + dune + etherscan
# then add: base or solana-agent-kit (with caution)
```

### Trader
```
polygon.io + coingecko + tradingview (read-only)
# then add: alpaca (paper mode only initially)
```

## Resources

- **Source Article:** [@exploraX_ on X](https://x.com/exploraX_/status/2062448236439155173?s=20)
- **MCP Specification:** [modelcontextprotocol.org](https://modelcontextprotocol.org)
- **Official MCP Servers:** [github.com/modelcontextprotocol](https://github.com/modelcontextprotocol)
- **Project Repository:** [github.com/Moh4696/50-essential-mcp-servers](https://github.com/Moh4696/50-essential-mcp-servers)

## License

MIT — Free to use and modify
