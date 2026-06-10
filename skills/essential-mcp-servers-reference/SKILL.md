---
name: essential-mcp-servers-reference
description: Navigate, select, and install MCP servers from the curated 50 Essential MCP Servers collection for Claude, Gemini, and Codex
triggers:
  - what MCP servers should I install
  - show me essential MCP servers
  - which MCP servers do I need for development
  - help me configure MCP servers
  - recommend MCP servers for my use case
  - how do I install MCP server
  - what are the best MCP servers
  - MCP server selection guide
---

# Essential MCP Servers Reference

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The **50 Essential MCP Servers** collection is a curated reference guide for navigating the MCP (Model Context Protocol) ecosystem. As of mid-2026, over 20,000 MCP servers exist in public registries, but most are abandoned, insecure, or poorly maintained. This collection identifies the 50 highest-quality servers across nine categories and provides verified installation commands, security guidelines, and selection criteria.

MCP is the standard protocol that allows AI agents (Claude, Gemini, Codex, Cursor, Windsurf) to connect to external tools and services through a unified interface—like USB-C for AI.

## Core Principles (Critical)

### The Three Rules

1. **Install 3-5 servers maximum** — More than 5-7 connected servers causes tool bloat, slowing down your agent and degrading response quality
2. **Treat every server as untrusted code** — Pin versions, scope tokens to read-only until verified, prefer official servers over community forks
3. **Never enable writes to production** — Point agents at read replicas, snapshots, or staging environments

### Selection Strategy

```yaml
# Decision tree for choosing MCP servers
role:
  software_engineer:
    essential: [github, context7, playwright, postgres/supabase, sentry]
    servers: 5
  
  content_creator:
    essential: [higgsfield, davinci-resolve, elevenlabs, youtube, figma]
    servers: 5
  
  trader_web3:
    start_read_only: [coingecko, polygon-io, the-graph, dune]
    add_after_testing: [base-mcp, alpaca]
    servers: 4-6
  
  operations_manager:
    essential: [slack, linear/jira, notion, google-drive]
    servers: 4
```

## Installation Syntax by Agent

### Claude Desktop

```bash
# Local server (npx/uvx)
claude mcp add <name> npx -y <package>

# Hosted server (https)
claude mcp add <name> --transport http --url https://api.example.com/mcp

# With environment variables
claude mcp add github npx -y @modelcontextprotocol/server-github
# Then edit ~/.claude/claude_desktop_config.json to add env vars
```

### Codex

```bash
# Local server
codex mcp add <name> -- npx -y <package>

# Or edit ~/.codex/config.toml directly
```

```toml
[mcpServers.github]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-github"]
env = { GITHUB_TOKEN = "${GITHUB_TOKEN}" }
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
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

## Category-Based Server Selection

### Category 01: Core Development (Universal)

**For:** Every software developer

```bash
# GitHub - repository management
claude mcp add github npx -y @modelcontextprotocol/server-github

# Context7 - codebase context and search
claude mcp add context7 npx -y @context7/mcp-server

# Playwright - browser automation and testing
claude mcp add playwright npx -y @modelcontextprotocol/server-playwright

# Filesystem - local file operations
claude mcp add filesystem npx -y @modelcontextprotocol/server-filesystem

# Brave Search - web search without API limits
claude mcp add brave-search npx -y @modelcontextprotocol/server-brave-search
```

**Environment variables needed:**
```bash
export GITHUB_TOKEN="ghp_..." # GitHub personal access token
export BRAVE_API_KEY="..." # Brave Search API key (free tier available)
```

### Category 02: Databases & Backend

**Warning:** All database servers can modify data. Start read-only.

```bash
# PostgreSQL - direct postgres connection
claude mcp add postgres npx -y @modelcontextprotocol/server-postgres
# Note: Official server is archived but still functional

# Supabase - hosted postgres with auth
claude mcp add supabase --transport http --url https://api.supabase.com/mcp

# Neon - serverless postgres
claude mcp add neon --transport http --url https://api.neon.tech/mcp

# SQLite - local database
claude mcp add sqlite npx -y @benborla29/mcp-server-sqlite

# Redis - cache and key-value store
claude mcp add redis npx -y @modelcontextprotocol/server-redis
```

**Postgres connection string (read-only example):**
```bash
export POSTGRES_CONNECTION="postgresql://readonly_user:${DB_PASSWORD}@db.example.com:5432/mydb?options=-c%20default_transaction_read_only=on"
```

### Category 03: Ops, Infra & Cloud

**Warning:** These servers can deploy, delete, and modify infrastructure.

```bash
# Cloudflare - DNS, workers, pages
claude mcp add cloudflare --transport http --url https://api.cloudflare.com/mcp

# Vercel - deployments and serverless
claude mcp add vercel --transport http --url https://api.vercel.com/mcp

# AWS - cloud infrastructure
claude mcp add aws npx -y @modelcontextprotocol/server-aws

# Docker Hub - container registry
claude mcp add docker npx -y @calclavia/mcp-server-docker

# Sentry - error monitoring (read-only safe)
claude mcp add sentry --transport http --url https://sentry.io/api/mcp

# Kubernetes - cluster management
claude mcp add k8s npx -y @kubernetes/mcp-server
```

**AWS credentials (least-privilege example):**
```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Use IAM role with read-only policy for initial testing
```

### Category 04: Corporate Productivity

```bash
# Slack - team messaging
claude mcp add slack npx -y @modelcontextprotocol/server-slack

# Linear - issue tracking
claude mcp add linear --transport http --url https://api.linear.app/mcp

# Notion - documents and wikis
claude mcp add notion --transport http --url https://api.notion.com/mcp

# Jira & Confluence - Atlassian suite
claude mcp add atlassian --transport http --url https://api.atlassian.com/mcp

# Google Drive - file storage (archived but works)
claude mcp add gdrive npx -y @modelcontextprotocol/server-gdrive

# Gmail - email (can send!)
claude mcp add gmail npx -y @modelcontextprotocol/server-gmail

# Google Calendar - scheduling
claude mcp add gcal npx -y @modelcontextprotocol/server-google-calendar

# Asana - project management
claude mcp add asana --transport http --url https://app.asana.com/api/mcp

# Airtable - spreadsheets and databases
claude mcp add airtable npx -y @domdomegg/mcp-server-airtable
```

### Category 05: Payments & Fintech

**Warning:** All fintech servers handle real money or financial data. Test in sandbox mode.

```bash
# Stripe - payments
claude mcp add stripe --transport http --url https://api.stripe.com/mcp
# Also available local: npx -y @modelcontextprotocol/server-stripe

# PayPal - payments
claude mcp add paypal --transport http --url https://api.paypal.com/mcp

# Plaid - banking connections
claude mcp add plaid --transport http --url https://api.plaid.com/mcp

# QuickBooks - accounting
claude mcp add quickbooks --transport http --url https://api.intuit.com/mcp
```

**Stripe sandbox mode:**
```bash
export STRIPE_SECRET_KEY="sk_test_..." # Test key, not live key
```

### Category 06: Blockchain & Web3

**Warning:** All web3 servers can execute irreversible on-chain transactions.

```bash
# Base - Coinbase L2 blockchain
claude mcp add base --transport http --url https://api.base.org/mcp

# Solana Agent Kit - Solana blockchain
claude mcp add solana npx -y @sendaifun/solana-agent-kit-mcp

# Thirdweb - web3 development
claude mcp add thirdweb npx -y @thirdweb-dev/mcp

# The Graph - blockchain indexing (read-only safe)
claude mcp add thegraph npx -y @graphprotocol/mcp-server

# Dune Analytics - on-chain data (read-only safe)
claude mcp add dune npx -y @dune/mcp-server

# Etherscan - Ethereum explorer (read-only safe)
claude mcp add etherscan npx -y @etherscan/mcp-server
```

**Safe pattern - read-only first:**
```bash
# Start with read-only data servers
export DUNE_API_KEY="..."
export ETHERSCAN_API_KEY="..."
export THE_GRAPH_API_KEY="..."

# Only after verification, add write-capable servers
# export BASE_PRIVATE_KEY="..." # For testnet only initially
```

### Category 07: Trading & Markets

**Warning:** Trading servers execute real financial transactions.

```bash
# Alpaca - stock/crypto trading API
claude mcp add alpaca npx -y @alpacahq/mcp-server

# CCXT - multi-exchange crypto trading
claude mcp add ccxt npx -y @ccxt/mcp-server

# Polygon.io - market data (read-only safe)
claude mcp add polygon npx -y @polygon-io/mcp-server

# CoinGecko - crypto prices (read-only safe)
claude mcp add coingecko --transport http --url https://api.coingecko.com/mcp

# TradingView - charts and indicators (read-only safe)
claude mcp add tradingview npx -y @tradingview/mcp-server
```

**Paper trading first:**
```bash
export ALPACA_API_KEY="PK..." # Paper trading key
export ALPACA_SECRET_KEY="..."
export ALPACA_BASE_URL="https://paper-api.alpaca.markets"
```

### Category 08: Creator Tools - Media & Design

```bash
# Higgsfield - AI video generation
claude mcp add higgsfield --transport http --url https://api.higgsfield.ai/mcp

# DaVinci Resolve - video editing
claude mcp add resolve npx -y @community/davinci-resolve-mcp

# Figma - design tools
claude mcp add figma --transport http --url https://api.figma.com/mcp
# Also local: npx -y @modelcontextprotocol/server-figma

# ElevenLabs - voice synthesis
claude mcp add elevenlabs npx -y @modelcontextprotocol/server-elevenlabs

# Blender - 3D modeling
claude mcp add blender npx -y @community/blender-mcp

# Canva - graphic design
claude mcp add canva --transport http --url https://api.canva.com/mcp

# Ableton Live - music production
claude mcp add ableton npx -y @community/ableton-mcp

# YouTube - video upload and management
claude mcp add youtube npx -y @modelcontextprotocol/server-youtube
```

### Category 09: Reasoning & Memory Layer

```bash
# Sequential Thinking - structured reasoning
claude mcp add sequential-thinking npx -y @modelcontextprotocol/server-sequential-thinking

# Memory - persistent agent memory
claude mcp add memory npx -y @modelcontextprotocol/server-memory
```

## Configuration Examples

### Minimal Developer Setup (5 servers)

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION": "${POSTGRES_READONLY_URL}"
      }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-playwright"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "env": {
        "ALLOWED_DIRECTORIES": "/home/user/projects"
      }
    },
    "sentry": {
      "transport": "http",
      "url": "https://sentry.io/api/mcp",
      "env": {
        "SENTRY_AUTH_TOKEN": "${SENTRY_TOKEN}"
      }
    }
  }
}
```

### Content Creator Setup (5 servers)

```json
{
  "mcpServers": {
    "figma": {
      "transport": "http",
      "url": "https://api.figma.com/mcp",
      "env": {
        "FIGMA_ACCESS_TOKEN": "${FIGMA_TOKEN}"
      }
    },
    "elevenlabs": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-elevenlabs"],
      "env": {
        "ELEVENLABS_API_KEY": "${ELEVENLABS_KEY}"
      }
    },
    "youtube": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-youtube"],
      "env": {
        "YOUTUBE_CLIENT_ID": "${YOUTUBE_CLIENT_ID}",
        "YOUTUBE_CLIENT_SECRET": "${YOUTUBE_CLIENT_SECRET}"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "env": {
        "ALLOWED_DIRECTORIES": "/home/user/content,/home/user/renders"
      }
    },
    "higgsfield": {
      "transport": "http",
      "url": "https://api.higgsfield.ai/mcp",
      "env": {
        "HIGGSFIELD_API_KEY": "${HIGGSFIELD_KEY}"
      }
    }
  }
}
```

### Read-Only Data Analysis Setup (4 servers)

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION": "postgresql://readonly:${PASSWORD}@db:5432/analytics?options=-c%20default_transaction_read_only=on"
      }
    },
    "dune": {
      "command": "npx",
      "args": ["-y", "@dune/mcp-server"],
      "env": {
        "DUNE_API_KEY": "${DUNE_KEY}"
      }
    },
    "polygon": {
      "command": "npx",
      "args": ["-y", "@polygon-io/mcp-server"],
      "env": {
        "POLYGON_API_KEY": "${POLYGON_KEY}"
      }
    },
    "coingecko": {
      "transport": "http",
      "url": "https://api.coingecko.com/mcp"
    }
  }
}
```

## Security Patterns

### Token Scoping

```bash
# GitHub - limit to specific repos
# Create fine-grained token at https://github.com/settings/tokens
# Scopes: Contents (read-only), Pull requests (read/write on specific repos only)
export GITHUB_TOKEN="github_pat_..."

# Stripe - separate keys for test/live
export STRIPE_TEST_KEY="sk_test_..."  # For development
export STRIPE_LIVE_KEY="sk_live_..."  # For production (never give to agent)

# AWS - use least-privilege IAM role
# Create policy with only required permissions, attach to user
export AWS_ACCESS_KEY_ID="AKIA..."
```

### Read-Only Database Access

```sql
-- PostgreSQL: create read-only user
CREATE ROLE agent_readonly WITH LOGIN PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE mydb TO agent_readonly;
GRANT USAGE ON SCHEMA public TO agent_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO agent_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO agent_readonly;

-- Enforce at connection level
ALTER ROLE agent_readonly SET default_transaction_read_only = on;
```

### Filesystem Restrictions

```json
{
  "filesystem": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-filesystem"],
    "env": {
      "ALLOWED_DIRECTORIES": "/home/user/safe_projects,/tmp/agent_workspace",
      "DENIED_PATTERNS": "*.env,*.pem,*.key,id_rsa,*.secret"
    }
  }
}
```

## Common Patterns

### Pattern: Safe Production Writes

```json
{
  "mcpServers": {
    "postgres-production": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION": "${PROD_READONLY_URL}"
      }
    },
    "postgres-staging": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION": "${STAGING_URL}"
      }
    }
  }
}
```

Agent instruction:
> "Always query postgres-production to understand the data structure. If you need to test writes, use postgres-staging. Never write to postgres-production directly."

### Pattern: Multi-Stage Financial Operations

```json
{
  "mcpServers": {
    "stripe-sandbox": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-stripe"],
      "env": {
        "STRIPE_SECRET_KEY": "${STRIPE_TEST_KEY}"
      }
    }
  }
}
```

Workflow:
1. Test payment flow in sandbox
2. Human reviews proposed transaction
3. Human manually executes in production dashboard

### Pattern: Gradual Privilege Escalation

```bash
# Week 1: Read-only servers only
claude mcp add github npx -y @modelcontextprotocol/server-github  # Read repos
claude mcp add postgres npx -y @modelcontextprotocol/server-postgres  # Readonly user

# Week 2: Add write to staging after verification
# Update GitHub token to include write access to feature branches
# Update postgres connection to staging database

# Week 3+: Production writes only for validated workflows
# Keep production read-only by default
# Enable specific write operations with explicit human confirmation
```

## Troubleshooting

### Agent Is Slow or Confused

**Symptom:** Responses take 10+ seconds, agent keeps selecting wrong tools

**Cause:** Too many connected servers (>7)

**Fix:**
```bash
# List current servers
claude mcp list

# Remove servers you're not actively using
claude mcp remove server-name

# Aim for 3-5 servers maximum
```

### Server Won't Connect

**Symptom:** `MCP server failed to start` or timeout errors

**Fix:**
```bash
# Check server is available
npx -y @modelcontextprotocol/server-github --version

# Verify environment variables
echo $GITHUB_TOKEN  # Should print token

# Check config file syntax
cat ~/.claude/claude_desktop_config.json | jq .

# Restart agent
# Claude Desktop: Quit and reopen
# Codex: codex restart
```

### Permission Denied Errors

**Symptom:** `403 Forbidden` or `Access denied` from server

**Fix:**
```bash
# Verify token has required scopes
# GitHub: https://github.com/settings/tokens (check scopes)
# API services: check dashboard for API key permissions

# Test token directly
curl -H "Authorization: Bearer $GITHUB_TOKEN" https://api.github.com/user

# Regenerate token if needed
```

### Database Connection Fails

**Symptom:** `could not connect to server` or `password authentication failed`

**Fix:**
```bash
# Test connection string directly
psql "$POSTGRES_CONNECTION"

# Common issues:
# - URL encoding: special chars in password need % encoding
# - SSL mode: add ?sslmode=require if needed
# - Host access: check pg_hba.conf allows connection from your IP
# - Network: verify no firewall blocking port 5432

# Verify connection format
# postgresql://user:password@host:port/database?options
```

### Hosted Server OAuth Fails

**Symptom:** Browser doesn't open or returns `invalid_client`

**Fix:**
```bash
# Clear cached tokens
rm -rf ~/.claude/oauth_cache/
rm -rf ~/.codex/oauth_cache/

# Verify redirect URL matches
# Most hosted servers use http://localhost:* callback
# Check your OAuth app settings match

# Try manual browser flow
# Copy URL from error message, paste in browser
```

### Writes Happening When They Shouldn't

**Symptom:** Agent modifying data unexpectedly

**Fix (Immediate):**
```bash
# Disconnect server immediately
claude mcp remove server-name

# Verify what happened
# Check server logs, database audit logs, API dashboard
```

**Prevention:**
```json
{
  "postgres": {
    "env": {
      "POSTGRES_CONNECTION": "...?options=-c%20default_transaction_read_only=on"
    }
  }
}
```

Always add `default_transaction_read_only=on` for database connections.

## Version Pinning

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github@1.2.3"
      ],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**Find current version:**
```bash
npm view @modelcontextprotocol/server-github version
```

**Update workflow:**
1. Check changelog for breaking changes
2. Update version in config
3. Test with read-only operations first
4. Verify behavior matches expectations

## Quick Reference Table

| Use Case | Servers | Count |
|---|---|---|
| Software dev (solo) | github, playwright, postgres, filesystem | 4 |
| Software dev (team) | github, slack, linear, postgres, sentry | 5 |
| Frontend dev | github, playwright, filesystem, figma, vercel | 5 |
| Data analysis | postgres, sqlite, brave-search, filesystem | 4 |
| Content creation | elevenlabs, youtube, figma, filesystem, higgsfield | 5 |
| Trading (safe start) | coingecko, polygon, dune, filesystem | 4 |
| Web3 dev (read-only) | the-graph, dune, etherscan, filesystem | 4 |
| DevOps | github, aws, kubernetes, sentry, slack | 5 |
| Product manager | linear, notion, slack, github, figma | 5 |

## Resources

- **Official MCP Spec:** https://spec.modelcontextprotocol.io
- **Server Registry:** https://github.com/modelcontextprotocol/servers
- **Security Audit (Q1 2026):** https://modelcontextprotocol.io/security/audit-2026-q1
- **Source Article:** https://x.com/exploraX_/status/2062448236439155173

---

**Remember:** This is a menu, not a shopping list. Pick 3-5 servers that match what you actually do. More servers = slower, dumber agent. Quality over quantity always wins.
