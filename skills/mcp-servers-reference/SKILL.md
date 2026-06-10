---
name: mcp-servers-reference
description: Expert guide to the 50 essential MCP servers for Claude, Gemini, and Codex - installation, configuration, and best practices
triggers:
  - how do I install MCP servers
  - what are the best MCP servers for development
  - show me MCP server configuration
  - which MCP servers should I use
  - how to configure MCP for Claude
  - what MCP servers are available for databases
  - help me set up MCP servers
  - MCP server recommendations
---

# MCP Servers Reference

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Expert knowledge of the 50 essential Model Context Protocol (MCP) servers for Claude, Gemini, and Codex. The MCP is a standardized protocol that allows AI agents to connect to external tools and services through a universal interface.

## What This Project Covers

This is a curated reference of 50 production-ready MCP servers across 9 categories:
- Core development (GitHub, filesystem, browser automation)
- Databases & backend (Postgres, SQLite, Redis, Supabase)
- Ops & infrastructure (AWS, Cloudflare, Vercel, Kubernetes)
- Corporate productivity (Slack, Linear, Notion, Jira)
- Payments & fintech (Stripe, PayPal, Plaid)
- Blockchain & Web3 (Base, Solana, Thirdweb)
- Trading & markets (Alpaca, CoinGecko, Polygon.io)
- Creator tools (Figma, ElevenLabs, YouTube)
- Reasoning & memory (Sequential thinking, Memory)

## The Three Golden Rules

1. **Don't install 50 servers** — Use 3-5 max. More causes tool bloat and degrades agent performance.
2. **Treat every server like untrusted code** — Pin versions, scope tokens read-only initially, prefer official vendors.
3. **Never enable writes against production** — Use read replicas, snapshots, or test environments.

## Installation Patterns

### Claude Code (Desktop)

```bash
# Local servers (npx/uvx)
claude mcp add github -- npx -y @modelcontextprotocol/server-github

# Hosted servers (HTTPS)
claude mcp add linear --transport http
```

### Codex

```bash
# CLI
codex mcp add github -- npx -y @modelcontextprotocol/server-github

# Or edit ~/.codex/config.toml
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

## Core Development Stack

### GitHub MCP Server

**Install:**
```bash
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```

**Environment:**
```bash
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_xxxxxxxxxxxx"
```

**Capabilities:**
- Create/update/search repositories
- Manage issues and pull requests
- Push commits, create branches
- Search code across repos

**Common patterns:**
```typescript
// Agent can now:
// - "create a new issue in my-org/my-repo about the login bug"
// - "search for all TODOs in the codebase"
// - "open a PR with the changes we just made"
// - "show me all open issues labeled 'bug'"
```

### Filesystem MCP Server

**Install:**
```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/allowed/directory
```

**Capabilities:**
- Read/write files in allowed directories
- List directory contents
- Move/rename files
- Create directories

**Security:**
```bash
# Only grant access to specific directories
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem \
  ~/projects/my-app \
  ~/documents/specs
```

### Playwright (Browser Automation)

**Install:**
```bash
claude mcp add playwright -- npx -y @executeautomation/playwright-mcp-server
```

**Use cases:**
- Automated testing workflows
- Web scraping
- Screenshot capture
- Form automation

### Brave Search

**Install:**
```bash
claude mcp add brave-search -- npx -y @modelcontextprotocol/server-brave-search
```

**Environment:**
```bash
export BRAVE_API_KEY="${BRAVE_API_KEY}"
```

## Database & Backend Stack

### PostgreSQL

**Install:**
```bash
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
```

**Environment:**
```bash
export POSTGRES_CONNECTION_STRING="postgresql://user:pass@localhost:5432/dbname"
```

**⚠️ CRITICAL: Use read replicas for production**

```bash
# Read-only replica
export POSTGRES_CONNECTION_STRING="postgresql://readonly:${DB_PASSWORD}@replica.example.com:5432/prod"

# Or local development only
export POSTGRES_CONNECTION_STRING="postgresql://localhost:5432/dev"
```

### Supabase

**Install:**
```bash
claude mcp add supabase --transport http
```

OAuth browser flow — follow prompts.

**Capabilities:**
- Query tables
- Real-time subscriptions
- Storage management
- Auth user management

### SQLite

**Install:**
```bash
claude mcp add sqlite -- npx -y @benborla/mcp-server-sqlite
```

**Configuration:**
```bash
# Point to specific database file
claude mcp add sqlite -- npx -y @benborla/mcp-server-sqlite --db-path ./data/app.db
```

### Redis

**Install:**
```bash
claude mcp add redis -- npx -y @jlowin/redis-mcp-server
```

**Environment:**
```bash
export REDIS_URL="redis://localhost:6379"
# Or with auth
export REDIS_URL="redis://:${REDIS_PASSWORD}@redis.example.com:6379"
```

## Ops & Infrastructure

### Cloudflare

**Install:**
```bash
claude mcp add cloudflare --transport http
```

**Capabilities:**
- Deploy Workers
- Manage DNS records
- Purge cache
- View analytics

**⚠️ Scope tokens appropriately:**
```bash
# Create a Cloudflare API token with specific permissions:
# - Workers: Edit
# - DNS: Read (not Edit unless needed)
# - Analytics: Read
```

### AWS

**Install:**
```bash
claude mcp add aws -- npx -y @modelcontextprotocol/server-aws
```

**Environment:**
```bash
export AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}"
export AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}"
export AWS_REGION="us-east-1"
```

**⚠️ Use least-privilege IAM policies:**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:ListBucket",
      "s3:GetObject",
      "ec2:DescribeInstances",
      "lambda:ListFunctions"
    ],
    "Resource": "*"
  }]
}
```

### Kubernetes

**Install:**
```bash
claude mcp add k8s -- npx -y @strowk/mcp-k8s-go
```

**Configuration:**
Uses your local `~/.kube/config`.

**⚠️ Point to dev clusters:**
```bash
# Switch context before enabling
kubectl config use-context dev-cluster

# Never point at production unless read-only
```

## Productivity & Team Tools

### Slack

**Install:**
```bash
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack
```

**Environment:**
```bash
export SLACK_BOT_TOKEN="${SLACK_BOT_TOKEN}"
export SLACK_TEAM_ID="${SLACK_TEAM_ID}"
```

**Patterns:**
```typescript
// Agent can:
// - "post the deployment summary to #engineering"
// - "search slack for discussions about the API refactor"
// - "send a DM to @alice with the bug report"
```

### Linear

**Install:**
```bash
claude mcp add linear --transport http
```

OAuth browser flow.

**Capabilities:**
- Create/update issues
- Search issues
- Manage projects and cycles
- Update issue status

### Notion

**Install:**
```bash
claude mcp add notion --transport http
```

**Patterns:**
```typescript
// - "create a new page in the Engineering wiki"
// - "update the sprint planning doc with today's decisions"
// - "search notion for the API documentation"
```

### Google Drive

**Install:**
```bash
claude mcp add gdrive -- npx -y @modelcontextprotocol/server-gdrive
```

**Environment:**
Requires Google OAuth credentials.

**Configuration:**
```json
{
  "env": {
    "GOOGLE_CLIENT_ID": "${GOOGLE_CLIENT_ID}",
    "GOOGLE_CLIENT_SECRET": "${GOOGLE_CLIENT_SECRET}"
  }
}
```

## Payments & Fintech Stack

### Stripe

**Install:**
```bash
# Hosted (recommended)
claude mcp add stripe --transport http

# Or local
claude mcp add stripe -- npx -y @modelcontextprotocol/server-stripe
```

**Environment:**
```bash
# Use test keys initially
export STRIPE_API_KEY="sk_test_xxxxx"

# Production (read-only recommended)
export STRIPE_API_KEY="sk_live_xxxxx"  # Restricted key
```

**⚠️ Use restricted API keys:**
```typescript
// In Stripe dashboard, create restricted key with ONLY:
// - Charges: Read
// - Customers: Read
// - Invoices: Read
// - Payment Intents: Read
// NO write permissions until you've verified behavior
```

### PayPal

**Install:**
```bash
claude mcp add paypal --transport http
```

**⚠️ Sandbox first:**
```bash
# Use sandbox credentials for testing
export PAYPAL_CLIENT_ID="${PAYPAL_SANDBOX_CLIENT_ID}"
export PAYPAL_CLIENT_SECRET="${PAYPAL_SANDBOX_CLIENT_SECRET}"
export PAYPAL_MODE="sandbox"
```

## Web3 & Blockchain

### Base MCP

**Install:**
```bash
claude mcp add base --transport http
```

**⚠️ IRREVERSIBLE TRANSACTIONS:**
- Start with testnet (Base Sepolia)
- Require explicit confirmation for mainnet transactions
- Never auto-approve transactions over $X threshold

**Pattern:**
```typescript
// Agent workflow:
// 1. Agent proposes transaction
// 2. Shows you the details (amount, gas, recipient)
// 3. YOU approve explicitly
// 4. Agent executes

// Example confirmation prompt:
// "I want to send 0.1 ETH to 0x123... on Base mainnet. Gas: 0.002 ETH. Approve? (yes/no)"
```

### Solana Agent Kit

**Install:**
```bash
claude mcp add solana -- npx -y solana-agent-kit-mcp
```

**Environment:**
```bash
# Devnet first
export SOLANA_NETWORK="devnet"
export SOLANA_PRIVATE_KEY="${SOLANA_DEVNET_KEY}"

# Mainnet (after testing)
export SOLANA_NETWORK="mainnet-beta"
export SOLANA_PRIVATE_KEY="${SOLANA_MAINNET_KEY}"
```

### The Graph (Read-only)

**Install:**
```bash
claude mcp add thegraph -- npx -y @modelcontextprotocol/server-thegraph
```

**Use case:** Query blockchain data without risk.

```graphql
# Agent can query subgraphs like:
{
  tokens(first: 5, orderBy: volumeUSD, orderDirection: desc) {
    id
    symbol
    volumeUSD
  }
}
```

## Trading & Markets

### Alpaca (Stock Trading)

**Install:**
```bash
claude mcp add alpaca -- npx -y @modelcontextprotocol/server-alpaca
```

**Environment:**
```bash
# Paper trading (REQUIRED FIRST)
export ALPACA_API_KEY="${ALPACA_PAPER_KEY}"
export ALPACA_API_SECRET="${ALPACA_PAPER_SECRET}"
export ALPACA_PAPER="true"

# Live trading (after extensive paper testing)
export ALPACA_PAPER="false"
```

**⚠️ Paper trade for weeks before live:**
```typescript
// Workflow:
// 1. Install with paper trading
// 2. Run strategies for 2-4 weeks
// 3. Review every trade the agent made
// 4. If confident, switch to live with SMALL position sizes
// 5. Gradually increase as you build trust
```

### CoinGecko (Read-only)

**Install:**
```bash
claude mcp add coingecko --transport http
```

Safe — read-only market data. No API key required for basic tier.

### Polygon.io (Market Data)

**Install:**
```bash
claude mcp add polygon -- npx -y @modelcontextprotocol/server-polygon
```

**Environment:**
```bash
export POLYGON_API_KEY="${POLYGON_API_KEY}"
```

Read-only market data. Safe for research and analysis.

## Creator Tools

### Figma

**Install:**
```bash
# Hosted
claude mcp add figma --transport http

# Or local
claude mcp add figma -- npx -y @modelcontextprotocol/server-figma
```

**Environment:**
```bash
export FIGMA_PERSONAL_ACCESS_TOKEN="${FIGMA_TOKEN}"
```

**Capabilities:**
- Read designs
- Export assets
- Update component properties
- Comment on designs

### ElevenLabs (Text-to-Speech)

**Install:**
```bash
claude mcp add elevenlabs -- npx -y @modelcontextprotocol/server-elevenlabs
```

**Environment:**
```bash
export ELEVENLABS_API_KEY="${ELEVENLABS_API_KEY}"
```

**Use case:**
```typescript
// - "generate a voiceover for this script using the professional voice"
// - "create audio for each section of the tutorial"
```

### YouTube

**Install:**
```bash
claude mcp add youtube -- npx -y mcp-youtube
```

**Capabilities:**
- Search videos
- Get transcripts
- Fetch metadata
- Analyze content

## Reasoning & Memory

### Sequential Thinking

**Install:**
```bash
claude mcp add sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

**What it does:** Forces the agent to break complex problems into explicit steps before acting.

**Use case:**
```typescript
// Without: Agent jumps to solution
// With: Agent shows reasoning:
// 1. Understand the requirement
// 2. Break into sub-tasks
// 3. Identify dependencies
// 4. Propose approach
// 5. Execute
```

### Memory

**Install:**
```bash
claude mcp add memory -- npx -y @modelcontextprotocol/server-memory
```

**What it does:** Persists knowledge across sessions.

**Capabilities:**
- Store facts about your project
- Remember preferences
- Learn from corrections
- Build context over time

## Optimal Stack Configurations

### Full-Stack Developer

```bash
# 5 servers, maximum value
claude mcp add github -- npx -y @modelcontextprotocol/server-github
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/projects
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
claude mcp add playwright -- npx -y @executeautomation/playwright-mcp-server
claude mcp add sentry --transport http
```

### Data Engineer

```bash
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
claude mcp add sqlite -- npx -y @benborla/mcp-server-sqlite
claude mcp add aws -- npx -y @modelcontextprotocol/server-aws
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```

### DevOps/SRE

```bash
claude mcp add k8s -- npx -y @strowk/mcp-k8s-go
claude mcp add aws -- npx -y @modelcontextprotocol/server-aws
claude mcp add sentry --transport http
claude mcp add github -- npx -y @modelcontextprotocol/server-github
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack
```

### Product Manager

```bash
claude mcp add linear --transport http
claude mcp add notion --transport http
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack
claude mcp add github -- npx -y @modelcontextprotocol/server-github
```

### Web3 Developer

```bash
# Start read-only
claude mcp add thegraph -- npx -y @modelcontextprotocol/server-thegraph
claude mcp add coingecko --transport http
claude mcp add github -- npx -y @modelcontextprotocol/server-github

# Add transactional after testing
claude mcp add base --transport http  # testnet first
```

### Trader/Quant

```bash
# All read-only/paper initially
claude mcp add polygon -- npx -y @modelcontextprotocol/server-polygon
claude mcp add coingecko --transport http
claude mcp add alpaca -- npx -y @modelcontextprotocol/server-alpaca  # paper=true
```

## Configuration Best Practices

### Environment Variables Pattern

Create a `.env.mcp` file (add to `.gitignore`):

```bash
# .env.mcp
export GITHUB_PERSONAL_ACCESS_TOKEN="ghp_xxxxx"
export POSTGRES_CONNECTION_STRING="postgresql://localhost:5432/dev"
export STRIPE_API_KEY="sk_test_xxxxx"
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export SLACK_BOT_TOKEN="xoxb-..."
```

Load before starting agent:
```bash
source .env.mcp
claude  # or codex, gemini
```

### Version Pinning

Pin MCP server versions in production:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github@1.2.3"
      ]
    }
  }
}
```

### Read-Only First

For any server with write capabilities:

1. **Install with read-only token/scope**
2. **Test for 1-2 weeks**
3. **Review agent's behavior**
4. **Gradually add write permissions**

Example (GitHub):
```bash
# Week 1: Read-only token
# Scopes: repo:status, public_repo (read)

# Week 3: Add write after observation
# Scopes: repo (full)
```

## Troubleshooting

### Agent is slow/confused

**Cause:** Too many servers connected (tool bloat).

**Fix:**
```bash
# List active servers
claude mcp list

# Remove unused
claude mcp remove server-name

# Restart with 3-5 core servers only
```

### Authentication failures

**Check:**
```bash
# Environment variables loaded?
echo $GITHUB_PERSONAL_ACCESS_TOKEN

# Token still valid? (many expire)
# Check provider dashboard

# Correct scopes?
# Review server docs for required permissions
```

### Server not showing up

```bash
# Verify installation
claude mcp list

# Check server is running
npx @modelcontextprotocol/server-github  # should not error

# Restart Claude/agent
```

### Transactions failing (Web3/Trading)

**Checklist:**
- [ ] Using testnet/paper trading first?
- [ ] Sufficient balance (including gas)?
- [ ] Network congestion (check block explorer)?
- [ ] Transaction parameters correct (amount, recipient)?

### Database errors

```bash
# Check connection string
echo $POSTGRES_CONNECTION_STRING

# Test connection manually
psql $POSTGRES_CONNECTION_STRING -c "SELECT 1;"

# Verify user has required permissions
# SELECT: read queries
# INSERT/UPDATE/DELETE: writes (dangerous)
```

## Security Checklist

Before enabling any MCP server in production:

- [ ] Using environment variables (not hardcoded secrets)
- [ ] Tokens scoped to minimum required permissions
- [ ] Read-only where possible (databases, APIs)
- [ ] Version pinned (not `@latest`)
- [ ] Pointing at dev/test environment (not production)
- [ ] Reviewed server source code (if community/unofficial)
- [ ] Tested for 1-2 weeks before production
- [ ] Money/irreversible servers require explicit confirmation
- [ ] Logs/audit trail enabled
- [ ] Rotation policy for credentials (90 days max)

## Quick Reference: Install Commands

```bash
# Core development
claude mcp add github -- npx -y @modelcontextprotocol/server-github
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/projects
claude mcp add playwright -- npx -y @executeautomation/playwright-mcp-server
claude mcp add brave-search -- npx -y @modelcontextprotocol/server-brave-search

# Databases
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
claude mcp add sqlite -- npx -y @benborla/mcp-server-sqlite
claude mcp add redis -- npx -y @jlowin/redis-mcp-server
claude mcp add supabase --transport http
claude mcp add neon --transport http

# Infrastructure
claude mcp add aws -- npx -y @modelcontextprotocol/server-aws
claude mcp add cloudflare --transport http
claude mcp add vercel --transport http
claude mcp add k8s -- npx -y @strowk/mcp-k8s-go
claude mcp add sentry --transport http

# Productivity
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack
claude mcp add linear --transport http
claude mcp add notion --transport http
claude mcp add gdrive -- npx -y @modelcontextprotocol/server-gdrive

# Fintech (test mode)
claude mcp add stripe --transport http
claude mcp add paypal --transport http

# Web3 (read-only safe)
claude mcp add thegraph -- npx -y @modelcontextprotocol/server-thegraph
claude mcp add coingecko --transport http

# Trading (paper mode)
claude mcp add alpaca -- npx -y @modelcontextprotocol/server-alpaca
claude mcp add polygon -- npx -y @modelcontextprotocol/server-polygon

# Creator tools
claude mcp add figma --transport http
claude mcp add elevenlabs -- npx -y @modelcontextprotocol/server-elevenlabs
claude mcp add youtube -- npx -y mcp-youtube

# Reasoning
claude mcp add sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
claude mcp add memory -- npx -y @modelcontextprotocol/server-memory
```

## Resources

- **Official MCP Spec:** https://spec.modelcontextprotocol.io
- **Server Registry:** https://github.com/modelcontextprotocol/servers
- **Source Article:** https://x.com/exploraX_/status/2062448236439155173

---

**Remember:** Pick 3-5 servers that match your actual workflow. More is not better. Quality over quantity wins every time.
