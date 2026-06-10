---
name: mcp-servers-reference
description: Navigate and install from 50+ curated MCP servers for Claude, Gemini, and Codex with verified commands and security best practices
triggers:
  - show me mcp servers for database work
  - what mcp servers should i install for web development
  - how do i add github mcp server to claude
  - recommend mcp servers for my ai agent
  - what are safe mcp servers for production use
  - help me configure mcp servers for trading
  - show me how to install stripe mcp server
  - what mcp servers work with cursor and codex
---

# MCP Servers Reference

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The Model Context Protocol (MCP) is a standardized interface that lets AI agents (Claude Code, Cursor, Codex, Gemini CLI, Windsurf) connect to external tools and services. This reference catalogs 50 essential, vetted MCP servers across 9 categories: core development, databases, infrastructure, productivity, fintech, web3, trading, creator tools, and reasoning/memory.

**Critical context**: Over 20,000 MCP servers exist in public registries, but most are abandoned, insecure, or poorly maintained. This curated list focuses on official or well-maintained community servers with verified install commands.

## The Three Non-Negotiable Rules

1. **Don't install more than 5-7 servers** — Tool bloat degrades agent performance and reasoning quality. Pick 3-5 that match your actual workflow.

2. **Treat every server like untrusted code** — Pin versions, scope tokens to read-only until tested, prefer vendor-official over forks.

3. **Never enable writes against production from an agent loop** — Point at read replicas, snapshots, or dev environments for anything that can mutate data.

## Installation Patterns

### Claude Code (Desktop)
```bash
# Local server (npx/uvx)
claude mcp add github npx -y @modelcontextprotocol/server-github

# Hosted server (HTTP transport)
claude mcp add supabase --transport http https://mcp.supabase.com
```

### Codex
```bash
# Add via CLI
codex mcp add github -- npx -y @modelcontextprotocol/server-github

# Or edit ~/.codex/config.toml
[mcp.servers.github]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
env = { GITHUB_TOKEN = "github_pat_xxx" }
```

### Gemini CLI
```json
// Edit ~/.gemini/settings.json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "github_pat_xxx"
      }
    }
  }
}
```

## Server Categories & Recommendations

### Core Development (Start Here)

**GitHub** (Official, Hosted)
```bash
claude mcp add github npx -y @modelcontextprotocol/server-github
# Env: GITHUB_TOKEN (create at github.com/settings/tokens)
# Use cases: Create issues, review PRs, search repos, manage branches
# Gotcha: Needs repo/write scopes for PRs; start with read-only
```

**Context7** (Official, Hosted/Local)
```bash
# Hosted (recommended)
claude mcp add context7 --transport http https://api.context7.com/mcp
# Local
claude mcp add context7 npx -y @context7/mcp-server
# Use cases: Deep codebase indexing, semantic search across repos
# Gotcha: Heavy memory use on large monorepos
```

**Playwright** (Official, Local)
```bash
claude mcp add playwright npx -y @modelcontextprotocol/server-playwright
# Use cases: Browser automation, web scraping, E2E testing
# Gotcha: No write protection — can mutate sites you're logged into
```

**Filesystem** (Official, Local)
```bash
claude mcp add filesystem npx -y @modelcontextprotocol/server-filesystem
# Env: ALLOWED_DIRECTORIES=/path/to/project
# Use cases: Read/write files, navigate directories
# Gotcha: Scope to project dirs only; dangerous system-wide
```

**Brave Search** (Official, Local)
```bash
claude mcp add brave npx -y @modelcontextprotocol/server-brave-search
# Env: BRAVE_API_KEY (get at brave.com/search/api)
# Use cases: Web research, real-time data lookup
```

### Database & Backend

**PostgreSQL** (Official-Archived, Local)
```bash
claude mcp add postgres npx -y @modelcontextprotocol/server-postgres
# Env: DATABASE_URL=postgresql://user:pass@host:5432/db
# Gotcha: Full write access; ONLY point at dev/staging
```

**Supabase** (Official, Hosted)
```bash
claude mcp add supabase --transport http https://mcp.supabase.com
# Requires: Supabase project URL + anon/service key
# Use cases: Database queries, storage, auth operations
# Gotcha: Service role key has admin access; use anon key + RLS
```

**Neon** (Official, Hosted)
```bash
claude mcp add neon --transport http https://mcp.neon.tech
# Env: NEON_API_KEY (from neon.tech console)
# Use cases: Serverless Postgres, branch databases
```

**SQLite** (Community, Local)
```bash
claude mcp add sqlite npx -y @benborla29/mcp-server-sqlite
# Env: SQLITE_DB_PATH=/path/to/db.sqlite
# Use cases: Local dev databases, lightweight analytics
```

**Redis** (Official, Local)
```bash
claude mcp add redis npx -y @modelcontextprotocol/server-redis
# Env: REDIS_URL=redis://localhost:6379
# Use cases: Cache inspection, session debugging
```

### Infrastructure & Ops

**Cloudflare** (Official, Hosted)
```bash
claude mcp add cloudflare --transport http https://mcp.cloudflare.com
# Env: CLOUDFLARE_API_TOKEN (read-only recommended)
# Use cases: DNS management, worker deployment, analytics
# Gotcha: Can deploy code and change DNS; read-only until tested
```

**Vercel** (Official, Hosted)
```bash
claude mcp add vercel --transport http https://mcp.vercel.com
# Env: VERCEL_TOKEN (from vercel.com/account/tokens)
# Use cases: Deploy previews, check build logs, manage env vars
# Gotcha: Can trigger production deployments
```

**AWS** (Official, Local)
```bash
claude mcp add aws npx -y @modelcontextprotocol/server-aws
# Env: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION
# Use cases: S3 operations, Lambda management, CloudWatch logs
# Gotcha: Use IAM policies to restrict; never admin credentials
```

**Sentry** (Official, Hosted)
```bash
claude mcp add sentry --transport http https://mcp.sentry.io
# Env: SENTRY_AUTH_TOKEN (read-only scope)
# Use cases: Error investigation, release tracking
```

**Kubernetes** (Community, Local)
```bash
claude mcp add k8s npx -y @kubernetes/mcp-server
# Env: KUBECONFIG=/path/to/config (or uses default)
# Use cases: Pod inspection, log streaming, deployment status
# Gotcha: Can delete resources; use read-only RBAC
```

### Productivity & Team Tools

**Slack** (Community, Local)
```bash
claude mcp add slack npx -y @modelcontextprotocol/server-slack
# Env: SLACK_BOT_TOKEN, SLACK_APP_TOKEN
# Use cases: Send messages, search history, manage channels
# Gotcha: Can post to any channel bot is in; test in sandbox first
```

**Linear** (Official, Hosted)
```bash
claude mcp add linear --transport http https://mcp.linear.app
# Auth: OAuth flow on first connect
# Use cases: Create/update issues, query project status
```

**Notion** (Official, Hosted)
```bash
claude mcp add notion --transport http https://mcp.notion.so
# Auth: OAuth flow (grants page-level access)
# Use cases: Create pages, update databases, search workspace
```

**Jira/Confluence** (Official, Hosted)
```bash
claude mcp add atlassian --transport http https://mcp.atlassian.com
# Auth: OAuth (select Jira sites on connect)
# Use cases: Ticket management, wiki search
```

**Google Drive** (Official-Archived, Local)
```bash
claude mcp add gdrive npx -y @modelcontextprotocol/server-gdrive
# Env: GOOGLE_CREDENTIALS_JSON (service account JSON)
# Use cases: File search, doc reading, folder navigation
```

### Fintech & Payments

**Stripe** (Official, Hosted/Local)
```bash
# Hosted (recommended)
claude mcp add stripe --transport http https://mcp.stripe.com
# Local
claude mcp add stripe npx -y @modelcontextprotocol/server-stripe
# Env: STRIPE_SECRET_KEY (use test key: sk_test_...)
# Gotcha: ONLY use test keys; can create real charges in live mode
```

**PayPal** (Official, Hosted)
```bash
claude mcp add paypal --transport http https://mcp.paypal.com
# Env: PAYPAL_CLIENT_ID, PAYPAL_SECRET (sandbox credentials)
# Gotcha: Verify you're in sandbox mode; can process real payments
```

**Plaid** (Official, Hosted)
```bash
claude mcp add plaid --transport http https://mcp.plaid.com
# Env: PLAID_CLIENT_ID, PLAID_SECRET (sandbox/dev environment)
# Gotcha: Has access to real banking data; use dev environment only
```

### Web3 & Blockchain

**Base MCP** (Official, Hosted)
```bash
claude mcp add base --transport http https://mcp.base.org
# Env: BASE_RPC_URL (use testnet: sepolia-base)
# Gotcha: Can sign transactions; ONLY connect testnet wallets
```

**Solana Agent Kit** (Community, Local)
```bash
claude mcp add solana npx -y solana-agent-kit-mcp
# Env: SOLANA_RPC_URL, SOLANA_PRIVATE_KEY (devnet only)
# Use cases: Read wallet state, fetch NFTs, query programs
# Gotcha: Can broadcast transactions; use burner wallets
```

**Thirdweb** (Official, Local)
```bash
claude mcp add thirdweb npx -y @thirdweb-dev/mcp-server
# Env: THIRDWEB_SECRET_KEY (from thirdweb.com/dashboard)
# Use cases: Contract deployment, IPFS upload, wallet management
# Gotcha: Has write access to contracts; use testnets
```

**The Graph** (Community, Local)
```bash
claude mcp add graph npx -y @graphprotocol/mcp-server
# Env: GRAPH_API_KEY (read-only, from thegraph.com)
# Use cases: Query subgraphs, blockchain data indexing
```

**Etherscan** (Community, Local)
```bash
claude mcp add etherscan npx -y etherscan-mcp-server
# Env: ETHERSCAN_API_KEY (free tier at etherscan.io)
# Use cases: Contract verification, transaction lookup, gas prices
```

### Trading & Markets

**Alpaca** (Official, Local)
```bash
claude mcp add alpaca npx -y @alpacahq/mcp-server
# Env: APCA_API_KEY_ID, APCA_API_SECRET_KEY, APCA_PAPER=true
# Gotcha: ALWAYS set APCA_PAPER=true; can place real trades
```

**CCXT** (Community, Local)
```bash
claude mcp add ccxt npx -y ccxt-mcp-server
# Env: CCXT_EXCHANGE=binance, CCXT_API_KEY, CCXT_SECRET (testnet)
# Use cases: Multi-exchange trading, market data
# Gotcha: Can execute trades on live accounts; use testnet
```

**Polygon.io** (Official, Local)
```bash
claude mcp add polygon npx -y @polygon.io/mcp-server
# Env: POLYGON_API_KEY (from polygon.io)
# Use cases: Real-time stock/crypto prices, historical data
```

**CoinGecko** (Official, Hosted)
```bash
claude mcp add coingecko --transport http https://mcp.coingecko.com
# Env: COINGECKO_API_KEY (optional, for pro tier)
# Use cases: Crypto prices, market cap, trending coins
```

### Creator Tools & Media

**Higgsfield** (Official, Hosted)
```bash
claude mcp add higgsfield --transport http https://mcp.higgsfield.ai
# Auth: OAuth flow
# Use cases: AI video generation, style transfer
```

**DaVinci Resolve** (Community, Local)
```bash
claude mcp add resolve npx -y davinci-resolve-mcp
# Requires: DaVinci Resolve installed with scripting API enabled
# Use cases: Timeline editing, color grading automation, render jobs
```

**Figma** (Official, Hosted/Local)
```bash
# Hosted
claude mcp add figma --transport http https://mcp.figma.com
# Auth: OAuth (grants file access)
# Use cases: Read designs, export assets, update prototypes
```

**ElevenLabs** (Official, Local)
```bash
claude mcp add elevenlabs npx -y @elevenlabs/mcp-server
# Env: ELEVEN_API_KEY (from elevenlabs.io)
# Use cases: Text-to-speech, voice cloning, audio generation
```

**YouTube** (Community, Local)
```bash
claude mcp add youtube npx -y youtube-mcp-server
# Env: YOUTUBE_API_KEY (from console.cloud.google.com)
# Use cases: Search videos, fetch transcripts, channel analytics
```

### Reasoning & Memory

**Sequential Thinking** (Official, Local)
```bash
claude mcp add sequential-thinking npx -y @modelcontextprotocol/server-sequential-thinking
# Use cases: Multi-step reasoning, complex problem decomposition
# No env vars needed; pure reasoning augmentation
```

**Memory** (Official, Local)
```bash
claude mcp add memory npx -y @modelcontextprotocol/server-memory
# Use cases: Persistent context across sessions, user preferences
# Stores: ~/.mcp/memory.json (local file storage)
```

## Configuration Best Practices

### Environment Variables
```bash
# Create .env file (never commit)
GITHUB_TOKEN=github_pat_xxxxx
DATABASE_URL=postgresql://user:pass@localhost:5432/devdb
STRIPE_SECRET_KEY=sk_test_xxxxx
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
SLACK_BOT_TOKEN=xoxb-...

# Load in agent config
# Claude: Automatically reads from .env in project root
# Codex: Specify in ~/.codex/config.toml env section
# Gemini: Specify in ~/.gemini/settings.json env object
```

### Token Scoping (GitHub Example)
```bash
# Read-only (safe exploration)
# Scopes: repo:status, public_repo, read:user

# Write access (for PRs, commits)
# Scopes: repo, workflow

# Start narrow, expand only when needed
```

### Multi-Environment Pattern
```toml
# ~/.codex/config.toml
[mcp.servers.postgres-dev]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-postgres"]
env = { DATABASE_URL = "postgresql://localhost:5432/dev" }

[mcp.servers.postgres-staging]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-postgres"]
env = { DATABASE_URL = "postgresql://staging.example.com:5432/app" }

# Never add postgres-prod
```

## Workflow Examples

### Full-Stack Developer Setup
```bash
# Core 5 servers
claude mcp add github npx -y @modelcontextprotocol/server-github
claude mcp add postgres npx -y @modelcontextprotocol/server-postgres
claude mcp add playwright npx -y @modelcontextprotocol/server-playwright
claude mcp add sentry --transport http https://mcp.sentry.io
claude mcp add vercel --transport http https://mcp.vercel.com

# Usage: "Check Sentry for errors in the last deploy, then write a Playwright test to reproduce"
```

### Content Creator Setup
```bash
# Media pipeline
claude mcp add higgsfield --transport http https://mcp.higgsfield.ai
claude mcp add elevenlabs npx -y @elevenlabs/mcp-server
claude mcp add youtube npx -y youtube-mcp-server
claude mcp add figma --transport http https://mcp.figma.com

# Usage: "Generate a 30-second promo video, add voiceover, export thumbnail from Figma, upload to YouTube"
```

### Crypto Trader Setup (Read-Only First)
```bash
# Start with market data only
claude mcp add coingecko --transport http https://mcp.coingecko.com
claude mcp add graph npx -y @graphprotocol/mcp-server
claude mcp add etherscan npx -y etherscan-mcp-server

# After 2 weeks of testing, add execution (paper trading only)
claude mcp add alpaca npx -y @alpacahq/mcp-server
# APCA_PAPER=true in .env

# Usage: "Show me BTC price action, correlate with on-chain volume from The Graph, suggest entry points"
```

## Troubleshooting

### "Too many tools" / Slow responses
**Cause**: Agent is overwhelmed by 10+ servers with dozens of tools each.
**Fix**: Remove unused servers. Keep 5 or fewer active.
```bash
# List active servers
claude mcp list

# Remove unused
claude mcp remove server-name
```

### "Authentication failed" / 401 errors
**Cause**: Token expired, wrong scope, or env var not loaded.
**Fix**:
1. Verify token in browser/curl: `curl -H "Authorization: Bearer $TOKEN" https://api.example.com/user`
2. Check token scopes match server requirements
3. Restart agent to reload env vars

### Server crashes on startup
**Cause**: Missing dependency, wrong Node version, or port conflict.
**Fix**:
```bash
# Check Node version (most servers need Node 18+)
node --version

# Manually test server
npx -y @modelcontextprotocol/server-github
# Read error output

# Check logs
# Claude: ~/Library/Logs/Claude/mcp.log
# Codex: ~/.codex/logs/mcp.log
```

### "Permission denied" file/database errors
**Cause**: Server trying to access restricted paths or production databases.
**Fix**:
```bash
# Filesystem: Set ALLOWED_DIRECTORIES
ALLOWED_DIRECTORIES=/Users/me/projects:/tmp

# Postgres: Verify DATABASE_URL points to dev
echo $DATABASE_URL
# Should NOT be production host

# AWS: Check IAM policy
aws sts get-caller-identity
# Verify it's a restricted role
```

### Agent ignores newly added server
**Cause**: Config cache not refreshed.
**Fix**:
```bash
# Claude: Restart app
# Codex: codex mcp reload
# Gemini: gemini mcp reload

# Verify server appears
claude mcp list
```

## Security Checklist

Before enabling any server:

- [ ] Read the server's README for security warnings
- [ ] Check "money or irreversible?" flag in the reference table
- [ ] Use test/sandbox credentials for fintech/trading/web3 servers
- [ ] Scope tokens to minimum required permissions (read-only if possible)
- [ ] Point database servers at dev/staging, never production
- [ ] Pin server versions (avoid `latest` tag)
- [ ] Test in a throwaway project first
- [ ] Set up monitoring for API usage (catch runaway loops)
- [ ] Review agent logs weekly for unexpected tool use
- [ ] Rotate tokens quarterly

## Common Patterns

### Read-first, Write-later
```bash
# Week 1: Install with read-only token
claude mcp add github npx -y @modelcontextprotocol/server-github
# GITHUB_TOKEN has repo:status, read:user only

# Week 3: After observing behavior, upgrade to write
# Update GITHUB_TOKEN with repo, workflow scopes
# Restart agent
```

### Seasonal Rotation
```bash
# Install for project duration
claude mcp add stripe npx -y @modelcontextprotocol/server-stripe

# Remove after project ships
claude mcp remove stripe

# Keeps tool count lean
```

### Prompt-level Server Activation
```
# In your prompt to Claude:
"Using only the GitHub and Sentry servers, investigate why the payment flow is failing in production."

# Agent will ignore other connected servers, reducing noise
```

## Quick Reference: Top 5 by Use Case

| Use Case | Servers |
|---|---|
| **Web dev** | github, postgres, playwright, sentry, vercel |
| **Mobile dev** | github, supabase, sentry, firebase, linear |
| **Data science** | postgres, redis, jupyter, pandas, plotly |
| **DevOps** | aws, kubernetes, sentry, cloudflare, docker |
| **Content** | higgsfield, elevenlabs, figma, youtube, canva |
| **Trading** | coingecko, polygon.io, alpaca (paper), ccxt (testnet), dune |
| **Web3 dev** | thirdweb (testnet), the graph, etherscan, dune, base (testnet) |
| **Startup ops** | github, linear, notion, slack, stripe (test) |

## Resources

- **MCP Specification**: https://spec.modelcontextprotocol.io
- **Official Server Registry**: https://github.com/modelcontextprotocol/servers
- **Security Audit (March 2026)**: Reference mentions 2/3 of popular servers had issues; always verify official sources
- **Community Discord**: Most servers link to support channels in their READMEs

## Version Notes

This reference reflects the MCP ecosystem as of June 2026. The protocol and server landscape evolve rapidly:
- Check server GitHub repos for latest install commands
- Watch for deprecation notices (several official servers archived between 2024-2026)
- Subscribe to agent release notes (Claude Code, Codex, Gemini) for breaking changes
- Revisit your server config quarterly

**The cardinal rule**: Three to five well-chosen, thoroughly tested servers beat twenty that sounded useful when you installed them. Pick for your workflow, not for coverage.
