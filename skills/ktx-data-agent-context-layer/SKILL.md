---
name: ktx-data-agent-context-layer
description: Install and use ktx, the executable context layer that teaches AI agents to query data warehouses accurately with approved metrics, semantic layer, and business knowledge
triggers:
  - set up ktx for data querying
  - configure ktx semantic layer
  - help me query the warehouse with ktx
  - install ktx context layer
  - use ktx to analyze business data
  - build ktx context from our database
  - search ktx wiki and metrics
  - integrate ktx with this agent
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It combines approved metric definitions, joinable column detection, wiki/business knowledge, and semantic-layer query planning into one searchable surface exposed via CLI and MCP.

**Key capabilities:**
- Automatically builds warehouse context from PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite
- Ingests dbt, MetricFlow, LookML, Looker, Metabase, and Notion content
- Detects joinable columns and resolves fan/chasm traps
- Flags contradictions across business knowledge sources
- Exposes CLI tools and MCP server for agent execution
- Read-only by design — never writes to your database

## Installation

### Global CLI

```bash
npm install -g @kaelio/ktx
```

### Project-scoped (recommended for teams)

```bash
npm install --save-dev @kaelio/ktx
```

Then use `npx ktx` or add scripts to `package.json`:

```json
{
  "scripts": {
    "ktx": "ktx",
    "ktx:setup": "ktx setup",
    "ktx:ingest": "ktx ingest"
  }
}
```

## Quick Setup

### Interactive setup (creates/resumes project)

```bash
ktx setup
```

This command:
1. Creates or resumes a local ktx project
2. Configures LLM and embedding providers
3. Adds database connections
4. Sets up context sources (dbt, Looker, Notion, etc.)
5. Builds initial context
6. Installs agent integration (MCP)

### Check project status

```bash
ktx status
```

Example output:

```text
ktx project: /home/user/analytics
Project ready: yes
LLM ready: yes (claude-sonnet-4-6)
Embeddings ready: yes (text-embedding-3-small)
Databases configured: yes (warehouse)
Context sources configured: yes (dbt_main)
ktx context built: yes
Agent integration ready: yes (codex:project)
```

## Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources (metrics, dimensions)
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**What to commit:** `ktx.yaml`, `semantic-layer/`, `wiki/`  
**What to ignore:** `.ktx/` (contains secrets and local state)

## Configuration

### ktx.yaml structure

```yaml
version: 1
name: analytics-project

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key stored in .ktx/config.yaml or ANTHROPIC_API_KEY env var

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key in OPENAI_API_KEY env var

databases:
  warehouse:
    type: postgres
    connection:
      host: localhost
      port: 5432
      database: analytics
      # Credentials in .ktx/config.yaml or environment

context_sources:
  dbt_main:
    type: dbt
    path: ../dbt-project
  
  notion_wiki:
    type: notion
    # Token in .ktx/config.yaml or NOTION_TOKEN env var
    
  looker_metrics:
    type: looker
    base_url: https://company.looker.com
    # Credentials in .ktx/config.yaml
```

### Environment variables

```bash
# LLM providers
export ANTHROPIC_API_KEY=your_key
export OPENAI_API_KEY=your_key
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Database connections
export POSTGRES_PASSWORD=your_password
export SNOWFLAKE_PASSWORD=your_password

# Context sources
export NOTION_TOKEN=your_token
export LOOKER_CLIENT_ID=your_id
export LOOKER_CLIENT_SECRET=your_secret

# Project location (optional)
export KTX_PROJECT_DIR=/path/to/project
```

## Core Commands

### Build and update context

```bash
# Build context from all configured sources
ktx ingest

# Build from specific connection only
ktx ingest --connection warehouse

# Rebuild from scratch (clears previous context)
ktx ingest --rebuild
```

### Search semantic layer

```bash
# Search metrics and dimensions
ktx sl "revenue"

# Get details about specific metric
ktx sl "monthly recurring revenue" --details
```

Example output:

```text
Found 3 semantic sources matching "revenue":

1. metric: monthly_recurring_revenue
   Type: derived_metric
   Expression: SUM(subscription_revenue) FILTER (active = true)
   Dimensions: customer_id, plan_type, region

2. dimension: revenue_category
   Type: categorical
   Source: fact_orders.revenue_tier

3. metric: total_revenue
   Type: simple_metric
   Expression: SUM(order_total)
```

### Search wiki knowledge

```bash
# Search business context
ktx wiki "refund policy"

# Search user-scoped notes
ktx wiki "data quality issues" --scope user
```

### MCP server for agents

```bash
# Start MCP server (blocks)
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status
```

### Project management

```bash
# Validate configuration
ktx validate

# Show project info
ktx info

# Reset project (careful!)
ktx reset --confirm
```

## Agent Integration

### Claude Code / Codex integration

When you run `ktx setup`, it automatically configures MCP integration. For manual setup:

```json
// Add to your MCP settings (usually auto-configured)
{
  "mcpServers": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "/path/to/project"]
    }
  }
}
```

### Available MCP tools

Once the MCP server is running, agents can use:

- `search_semantic_layer` — Query metrics and dimensions
- `search_wiki` — Search business knowledge
- `get_metric_definition` — Fetch canonical metric SQL
- `list_joinable_tables` — Discover valid join paths
- `explain_context` — Get warehouse documentation

## Real-World Patterns

### Pattern 1: Setting up a new analytics project

```bash
# Create project directory
mkdir analytics-project && cd analytics-project

# Initialize git (optional but recommended)
git init

# Run interactive setup
ktx setup
# Follow prompts to configure LLM, database, and dbt

# Build initial context
ktx ingest

# Verify everything works
ktx status
ktx sl "revenue"

# Add .ktx/ to .gitignore
echo ".ktx/" >> .gitignore
git add ktx.yaml semantic-layer/ wiki/
git commit -m "Initialize ktx project"
```

### Pattern 2: Adding a dbt context source

```typescript
// After running ktx setup, edit ktx.yaml:
/*
context_sources:
  dbt_main:
    type: dbt
    path: ../dbt-project
    # ktx will read manifest.json and semantic models
*/

// Then rebuild context
// $ ktx ingest --connection dbt_main
```

### Pattern 3: Querying metrics programmatically

```bash
# Get metric definition as JSON
ktx sl "customer_lifetime_value" --format json > clv_metric.json

# Use in analysis script
node analyze.js < clv_metric.json
```

### Pattern 4: Multi-warehouse setup

```yaml
# ktx.yaml
databases:
  production:
    type: snowflake
    connection:
      account: mycompany
      warehouse: analytics_wh
      database: prod
  
  staging:
    type: postgres
    connection:
      host: staging-db.internal
      database: staging_analytics
```

```bash
# Build context from both
ktx ingest --connection production
ktx ingest --connection staging

# Search across both
ktx sl "orders" --all-connections
```

### Pattern 5: Adding wiki content

```bash
# Create global wiki page
cat > wiki/global/refund-policy.md << 'EOF'
---
title: Refund Policy
tags: [finance, customer-success]
---

# Refund Policy

Refunds are processed within 7 days of request.
Full refunds are only available within 30 days of purchase.

Relevant metrics:
- refund_rate: COUNT(refunds) / COUNT(orders)
- refund_amount: SUM(refund_total)
EOF

# Rebuild wiki index
ktx ingest

# Search it
ktx wiki "refund"
```

### Pattern 6: CI/CD integration

```yaml
# .github/workflows/ktx-validate.yml
name: Validate ktx Context

on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install ktx
        run: npm install -g @kaelio/ktx
      
      - name: Validate configuration
        run: ktx validate
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          POSTGRES_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}
      
      - name: Check for contradictions
        run: |
          ktx ingest --check-only
          # Fails if contradictions found
```

## Troubleshooting

### "ktx project not found"

```bash
# Ensure you're in a directory with ktx.yaml or set:
export KTX_PROJECT_DIR=/path/to/project

# Or pass explicitly:
ktx status --project-dir /path/to/project
```

### "LLM provider not configured"

```bash
# Run setup again to configure:
ktx setup

# Or manually edit .ktx/config.yaml and add:
# llm:
#   api_key: your_key

# Or use environment variable:
export ANTHROPIC_API_KEY=your_key
```

### "Database connection failed"

```bash
# Test connection:
ktx validate --connection warehouse

# Common fixes:
# 1. Check credentials in .ktx/config.yaml
# 2. Verify network access (firewall, VPN)
# 3. Confirm database permissions are read-only
# 4. Check connection string format in ktx.yaml
```

### "MCP server won't start"

```bash
# Check for port conflicts:
ktx mcp status

# Try explicit project directory:
ktx mcp start --project-dir $(pwd)

# Check agent client MCP settings:
# - command should be "ktx" (or full path)
# - args should be ["mcp", "start", "--project-dir", "/full/path"]
```

### "Context ingestion is slow"

```bash
# Sample large tables instead of full scan:
# Edit ktx.yaml:
# databases:
#   warehouse:
#     sampling:
#       enabled: true
#       max_rows: 10000

# Then rebuild:
ktx ingest --rebuild --connection warehouse
```

### "Contradictions detected"

```bash
# ktx flags when wiki and semantic layer disagree
# Review the contradiction report:
ktx ingest 2>&1 | grep "CONTRADICTION"

# Resolve by:
# 1. Updating wiki content in wiki/global/
# 2. Fixing semantic layer YAML in semantic-layer/
# 3. Correcting upstream dbt or Looker definitions
```

## Advanced Usage

### Custom semantic sources

Create YAML files in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/customer-metrics.yaml
metrics:
  - name: customer_lifetime_value
    type: derived_metric
    expression: |
      SUM(order_total) OVER (
        PARTITION BY customer_id
        ORDER BY order_date
      )
    dimensions:
      - customer_id
      - cohort_month
    
  - name: active_customers
    type: simple_metric
    expression: COUNT(DISTINCT customer_id) FILTER (last_order_date >= CURRENT_DATE - INTERVAL '90 days')
    dimensions:
      - region
      - plan_type

dimensions:
  - name: customer_segment
    type: categorical
    source: dim_customers.segment
    values:
      - enterprise
      - mid_market
      - smb
```

Then rebuild:

```bash
ktx ingest --connection warehouse
```

### TypeScript/JavaScript API

```typescript
// ktx also exposes a programmatic API
import { KtxProject } from '@kaelio/ktx';

async function queryMetrics() {
  const project = await KtxProject.load();
  
  // Search semantic layer
  const results = await project.semanticLayer.search('revenue');
  
  for (const result of results) {
    console.log(`${result.type}: ${result.name}`);
    console.log(`Expression: ${result.expression}`);
  }
  
  // Search wiki
  const wikiPages = await project.wiki.search('refund policy');
  console.log(`Found ${wikiPages.length} wiki pages`);
}

queryMetrics();
```

### LLM provider options

```yaml
# Anthropic API
llm:
  provider: anthropic
  model: claude-sonnet-4-6

# Google Vertex AI
llm:
  provider: vertex
  model: claude-3-5-sonnet-v2@20241022
  project: my-gcp-project
  location: us-central1

# AI Gateway (for caching/rate limiting)
llm:
  provider: ai-gateway
  upstream_provider: anthropic
  gateway_url: https://gateway.internal

# Local Claude Code session
llm:
  provider: claude-agent-sdk
  # Uses session from Claude Code/Codex automatically
```

## Key Concepts

**Semantic layer:** Metrics and dimensions defined in YAML that ktx uses to generate accurate SQL. Auto-built from dbt, Looker, or hand-written.

**Wiki:** Business knowledge stored in `wiki/global/` (shared) or `wiki/user/<id>/` (personal). Markdown files with frontmatter tags.

**Context sources:** Integrations that feed the context layer — dbt projects, Looker instances, Notion databases, Metabase collections.

**Join graph:** ktx automatically detects joinable columns and resolves fan/chasm traps so agents can fetch multi-table metrics declaratively.

**Contradiction detection:** ktx flags when wiki content and semantic definitions disagree, prompting human review.

## Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
