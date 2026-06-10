---
name: mcp-server-catalog-reference
description: Navigate and recommend MCP servers from a curated catalog of 50 essential servers for AI agents with install commands and gotchas
triggers:
  - show me MCP servers for database integration
  - what MCP servers should I install for web3 development
  - help me choose the right MCP servers for my project
  - install MCP servers for productivity tools
  - what are the best MCP servers for GitHub and development
  - show me safe MCP servers for trading and fintech
  - how do I configure MCP servers for Claude
  - recommend MCP servers for content creation
---

# MCP Server Catalog Reference

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What This Project Does

The **50-essential-mcp-servers** catalog is a curated reference guide for MCP (Model Context Protocol) servers that work with Claude Code, Gemini CLI, and Codex. It provides:

- **50 vetted MCP servers** organized into 9 categories (core dev, databases, ops/infra, productivity, fintech, web3, trading, creator tools, reasoning)
- **Install commands** for each agent (Claude Code, Codex, Gemini)
- **Safety flags** for servers that handle money, production data, or irreversible actions
- **Practical selection advice** to avoid tool bloat and performance degradation

## The Three Critical Rules

Before recommending ANY MCP server, enforce these rules:

1. **Never recommend more than 5-7 servers** — tool bloat makes agents slower and dumber
2. **Treat every server like untrusted code** — pin versions, scope tokens read-only until proven
3. **Never enable writes to production** — point at read replicas or snapshots for databases

## Server Categories & Selection Guide

### Core Development (Category 01)
**Best for:** Software engineers, full-stack developers
**Top picks:** GitHub, Context7, Playwright, Filesystem, Brave Search

```bash
# GitHub (official, hosted)
claude mcp add github --transport http

# Context7 (official, hosted or local)
claude mcp add context7 --transport http
# OR local
claude mcp add context7 -- npx -y @context-labs/mcp-server-context7

# Playwright (official, local)
claude mcp add playwright -- npx -y @modelcontextprotocol/server-playwright

# Filesystem (official, local)
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem

# Brave Search (official, local, requires API key)
claude mcp add brave-search -- npx -y @modelcontextprotocol/server-brave-search
```

### Databases & Backend (Category 02)
**Best for:** Backend engineers, data teams
**Top picks:** Postgres, Supabase, Neon, SQLite, Redis
**⚠️ All flagged as "money or irreversible" — use read replicas**

```bash
# Postgres (official archived, local)
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
# Config: Set DATABASE_URL env var to read replica

# Supabase (official, hosted)
claude mcp add supabase --transport http
# Requires project URL and anon key with restricted permissions

# Neon (official, hosted)
claude mcp add neon --transport http

# SQLite (community, local)
claude mcp add sqlite -- npx -y @benborla29/mcp-server-sqlite

# Redis (official, local)
claude mcp add redis -- npx -y @modelcontextprotocol/server-redis
```

### Ops, Infra & Cloud (Category 03)
**Best for:** DevOps, SRE, platform engineers
**Top picks:** Cloudflare, Vercel, AWS, Sentry
**⚠️ Most flagged — can modify production infrastructure**

```bash
# Cloudflare (official, hosted)
claude mcp add cloudflare --transport http

# Vercel (official, hosted)
claude mcp add vercel --transport http

# AWS (official, local)
claude mcp add aws -- npx -y @modelcontextprotocol/server-aws
# Requires AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY env vars

# Sentry (official, hosted, read-only)
claude mcp add sentry --transport http

# Kubernetes (community, local)
claude mcp add kubernetes -- npx -y mcp-server-kubernetes
# ⚠️ Can modify cluster state — scope RBAC carefully
```

### Corporate Productivity (Category 04)
**Best for:** Product managers, team leads, knowledge workers
**Top picks:** Slack, Linear, Notion, Jira, Google Drive

```bash
# Slack (community, local)
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack
# ⚠️ Can send messages — review message drafts first

# Linear (official, hosted)
claude mcp add linear --transport http

# Notion (official, hosted)
claude mcp add notion --transport http

# Jira/Confluence (official, hosted)
claude mcp add jira --transport http

# Google Drive (official archived, local)
claude mcp add gdrive -- npx -y @modelcontextprotocol/server-gdrive
# Requires Google OAuth token with drive.readonly scope
```

### Fintech & Payments (Category 05)
**Best for:** Financial applications, payment processing
**⚠️ ALL servers in this category handle money — use sandbox/test mode**

```bash
# Stripe (official, hosted or local)
claude mcp add stripe --transport http
# OR local with test key
claude mcp add stripe -- npx -y @modelcontextprotocol/server-stripe
# Set STRIPE_SECRET_KEY=sk_test_... (never live key)

# PayPal (official, hosted)
claude mcp add paypal --transport http
# Use sandbox credentials only

# Plaid (official, hosted)
claude mcp add plaid --transport http
# Use development environment

# QuickBooks (community, hosted)
# See vendor docs for OAuth setup
```

### Web3 & Blockchain (Category 06)
**Best for:** Blockchain developers, dApp builders
**⚠️ ALL servers execute irreversible on-chain transactions — use testnets**

```bash
# Base (official, hosted)
claude mcp add base --transport http
# Configure for Base Sepolia testnet first

# Solana Agent Kit (community, local)
claude mcp add solana -- npx -y solana-agent-kit-mcp
# Set SOLANA_RPC_URL=https://api.devnet.solana.com

# Thirdweb (official, local)
claude mcp add thirdweb -- npx -y @thirdweb-dev/mcp-server
# Set THIRDWEB_SECRET_KEY and network to testnet

# The Graph (community, local, read-only)
claude mcp add the-graph -- npx -y @graphprotocol/mcp-server

# Dune (community, local, read-only)
claude mcp add dune -- npx -y dune-mcp-server
```

### Trading & Markets (Category 07)
**Best for:** Traders, quant developers, market analysis
**⚠️ Trading servers can place real orders — paper trade first**

```bash
# Alpaca (official, local)
claude mcp add alpaca -- npx -y @alpacahq/mcp-server-alpaca
# Set ALPACA_API_KEY and APCA_API_BASE_URL=https://paper-api.alpaca.markets

# CCXT (community, local)
claude mcp add ccxt -- npx -y ccxt-mcp-server
# Configure exchange with testnet=true

# Polygon.io (official, local, read-only)
claude mcp add polygon -- npx -y @polygon-io/mcp-server

# CoinGecko (official, hosted, read-only)
claude mcp add coingecko --transport http

# TradingView (community, local, read-only)
claude mcp add tradingview -- npx -y tradingview-mcp
```

### Creator Tools (Category 08)
**Best for:** Video editors, designers, content creators
**Top picks:** Higgsfield, Figma, ElevenLabs, YouTube

```bash
# Higgsfield (official, hosted)
claude mcp add higgsfield --transport http

# DaVinci Resolve (community, local)
claude mcp add davinci -- npx -y davinci-resolve-mcp

# Figma (official, hosted or local)
claude mcp add figma --transport http

# ElevenLabs (official, local)
claude mcp add elevenlabs -- npx -y @elevenlabs/mcp-server
# Set ELEVENLABS_API_KEY env var

# YouTube (community, local)
claude mcp add youtube -- npx -y youtube-mcp-server
# Requires YouTube Data API key
```

### Reasoning & Memory (Category 09)
**Best for:** Complex workflows, multi-step reasoning
**Official Anthropic servers for extended thinking**

```bash
# Sequential Thinking (official, local)
claude mcp add sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking

# Memory (official, local)
claude mcp add memory -- npx -y @modelcontextprotocol/server-memory
```

## Configuration Pattern

All three major agents use similar JSON config. Example for Claude Code's `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "github": {
      "transport": {
        "type": "http",
        "url": "https://github-mcp.glitch.me"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@read-replica.host:5432/db"
      }
    },
    "stripe": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-stripe"],
      "env": {
        "STRIPE_SECRET_KEY": "${STRIPE_TEST_KEY}"
      }
    }
  }
}
```

### Codex Equivalent (`~/.codex/config.toml`)

```toml
[mcp.github]
transport = "http"
url = "https://github-mcp.glitch.me"

[mcp.postgres]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-postgres"]
[mcp.postgres.env]
DATABASE_URL = "postgresql://user:pass@read-replica.host:5432/db"
```

### Gemini CLI Equivalent (`~/.gemini/settings.json`)

```json
{
  "mcpServers": {
    "github": {
      "transport": "http",
      "url": "https://github-mcp.glitch.me"
    }
  }
}
```

## Selection Decision Tree

Use this logic when recommending servers:

```
1. What's the user's primary role?
   - Software dev → Core dev (01) + Database (02)
   - DevOps/SRE → Ops/infra (03) + Database (02)
   - Product/PM → Productivity (04)
   - Trader/quant → Trading (07) + Markets data
   - Blockchain dev → Web3 (06) + Development (01)
   - Content creator → Creator tools (08)

2. How many servers do they already have?
   - 0-2: Recommend up to 3 more
   - 3-5: Recommend 1-2 targeted additions
   - 6+: STOP — suggest removing before adding

3. Does the server handle money/irreversible actions?
   - YES: Warn explicitly, suggest sandbox/testnet, confirm human-in-loop
   - NO: Proceed with install

4. Is there an official version?
   - YES: Always prefer official over community
   - NO: Check if community server is actively maintained
```

## Safety Checklist for Dangerous Servers

Before installing servers flagged "money or irreversible":

### Databases (Postgres, Supabase, Neon, Redis)
```bash
# ✅ GOOD: Read replica
DATABASE_URL="postgresql://readonly:pass@replica.host/db"

# ❌ BAD: Production primary with write access
DATABASE_URL="postgresql://admin:pass@prod-primary.host/db"
```

### Fintech (Stripe, PayPal, Plaid)
```bash
# ✅ GOOD: Test mode
STRIPE_SECRET_KEY="sk_test_..."
PAYPAL_MODE="sandbox"

# ❌ BAD: Live keys
STRIPE_SECRET_KEY="sk_live_..."
```

### Web3 (Base, Thirdweb, Solana)
```bash
# ✅ GOOD: Testnet
SOLANA_RPC_URL="https://api.devnet.solana.com"
BASE_NETWORK="base-sepolia"

# ❌ BAD: Mainnet without confirmation
SOLANA_RPC_URL="https://api.mainnet-beta.solana.com"
```

### Trading (Alpaca, CCXT)
```bash
# ✅ GOOD: Paper trading
APCA_API_BASE_URL="https://paper-api.alpaca.markets"
CCXT_EXCHANGE_SANDBOX="true"

# ❌ BAD: Live trading without testing
APCA_API_BASE_URL="https://api.alpaca.markets"
```

## Common Patterns

### Pattern 1: Dev Stack (5 servers)
```bash
claude mcp add github --transport http
claude mcp add context7 --transport http
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres
claude mcp add sentry --transport http
claude mcp add playwright -- npx -y @modelcontextprotocol/server-playwright
```

### Pattern 2: Content Creator Stack (5 servers)
```bash
claude mcp add higgsfield --transport http
claude mcp add figma --transport http
claude mcp add elevenlabs -- npx -y @elevenlabs/mcp-server
claude mcp add youtube -- npx -y youtube-mcp-server
claude mcp add davinci -- npx -y davinci-resolve-mcp
```

### Pattern 3: Web3 Builder Stack (5 servers, testnet-safe)
```bash
claude mcp add github --transport http
claude mcp add base --transport http  # Configure for Sepolia
claude mcp add the-graph -- npx -y @graphprotocol/mcp-server
claude mcp add dune -- npx -y dune-mcp-server
claude mcp add context7 --transport http
```

### Pattern 4: Product/PM Stack (5 servers)
```bash
claude mcp add linear --transport http
claude mcp add notion --transport http
claude mcp add slack -- npx -y @modelcontextprotocol/server-slack
claude mcp add gdrive -- npx -y @modelcontextprotocol/server-gdrive
claude mcp add github --transport http
```

## Troubleshooting

### Server Not Appearing in Tool List
```bash
# 1. Verify config syntax
cat ~/.config/Claude/claude_desktop_config.json | jq .

# 2. Check server can start independently
npx -y @modelcontextprotocol/server-postgres

# 3. Restart agent completely
# Claude Code: Cmd+Q and reopen
# Codex: codex restart
```

### Agent Getting Slow/Confused
```bash
# Count your servers
jq '.mcpServers | keys | length' ~/.config/Claude/claude_desktop_config.json

# If > 7: Remove least-used servers
# Keep only what you used this week
```

### Permission/Auth Errors
```bash
# For hosted servers: Re-authenticate
claude mcp remove github
claude mcp add github --transport http  # Triggers OAuth

# For local servers: Check env vars are set
npx -y @modelcontextprotocol/server-stripe  # Will fail if STRIPE_SECRET_KEY missing
```

### Server Version Conflicts
```bash
# Pin to specific version
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres@1.2.3

# Check installed version
npm list -g @modelcontextprotocol/server-postgres
```

### Dangerous Action Attempted
```bash
# If agent tries to execute money/irreversible action:
# 1. STOP immediately
# 2. Review the exact command/API call
# 3. Verify environment is test/sandbox
# 4. If prod: Remove server or switch to read-only

# Example: Switch Stripe to test
claude mcp remove stripe
# Edit config to use sk_test_ key
claude mcp add stripe -- npx -y @modelcontextprotocol/server-stripe
```

## Quick Reference: Server Safety Matrix

| Server | Read-Only? | Requires Sandbox? | Production Risk |
|--------|------------|-------------------|-----------------|
| GitHub | ✅ Can be | ❌ No | Low (code reviews exist) |
| Postgres | ❌ No | ✅ Yes | **HIGH** (data loss) |
| Stripe | ❌ No | ✅ Yes | **CRITICAL** (money) |
| Slack | ❌ No | ❌ No | Medium (can spam) |
| Base/Solana | ❌ No | ✅ Yes | **CRITICAL** (irreversible) |
| Alpaca | ❌ No | ✅ Yes | **CRITICAL** (real trades) |
| Sentry | ✅ Yes | ❌ No | None |
| CoinGecko | ✅ Yes | ❌ No | None |

## Environment Variables Reference

```bash
# Database servers
DATABASE_URL="postgresql://user:pass@host:5432/db"
REDIS_URL="redis://localhost:6379"

# Fintech
STRIPE_SECRET_KEY="sk_test_..."
PLAID_CLIENT_ID="..."
PLAID_SECRET="..."
PLAID_ENV="sandbox"

# Web3
SOLANA_RPC_URL="https://api.devnet.solana.com"
SOLANA_PRIVATE_KEY="..."
THIRDWEB_SECRET_KEY="..."
THIRDWEB_NETWORK="base-sepolia"

# Trading
ALPACA_API_KEY="..."
ALPACA_SECRET_KEY="..."
APCA_API_BASE_URL="https://paper-api.alpaca.markets"

# APIs
BRAVE_SEARCH_API_KEY="..."
ELEVENLABS_API_KEY="..."
YOUTUBE_API_KEY="..."
COINGECKO_API_KEY="..."

# Cloud providers
AWS_ACCESS_KEY_ID="..."
AWS_SECRET_ACCESS_KEY="..."
AWS_REGION="us-east-1"
CLOUDFLARE_API_TOKEN="..."
```

## Key Recommendations for AI Agents

When a user asks for MCP server help:

1. **Always ask their role first** — don't guess
2. **Count their existing servers** — enforce the 5-7 limit
3. **Warn on every dangerous server** — be explicit about risks
4. **Provide the exact install command** — don't make them hunt
5. **Suggest test/sandbox by default** — never assume prod is okay
6. **Recommend removal** — if adding server #8, suggest dropping #1
7. **Link to official docs** — catalog is a map, not the territory

The catalog exists because the MCP ecosystem is huge and dangerous. Your job is to be the safety rail between "sounds cool" and "lost money/data."
