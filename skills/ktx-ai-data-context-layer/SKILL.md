---
name: ktx-ai-data-context-layer
description: Install, configure, and use ktx - the executable context layer for AI agents to query data warehouses accurately using approved metrics, semantic layers, and business knowledge
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer and wiki
  - help me query the data warehouse with ktx
  - use ktx to build context from our dbt project
  - integrate ktx with Claude Code for analytics
  - troubleshoot ktx semantic layer configuration
  - ingest data warehouse metadata with ktx
  - search ktx wiki and semantic sources
---

# ktx AI Data Context Layer Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It automatically ingests company knowledge (wikis, dbt, Looker, Metabase, Notion), builds semantic layers with approved metric definitions, detects joinable columns, resolves fan/chasm traps, and exposes everything through CLI and MCP tools for AI agents.

**Key capabilities:**
- Learns from company knowledge sources automatically
- Maps data stack metadata and usage patterns
- Builds semantic layer combining raw tables and metrics
- Detects joinable columns and resolves SQL query traps
- Serves agents via CLI and MCP with semantic search
- Read-only by design - never writes to your database

**Supported databases:** PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite

**Supported integrations:** dbt, MetricFlow, LookML, Looker, Metabase, Notion

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install --save-dev @kaelio/ktx
```

### Verify Installation

```bash
ktx --version
ktx --help
```

## Quick Start

### Initial Setup

```bash
# Create or resume a ktx project
ktx setup

# Check project status
ktx status
```

The `ktx setup` command is interactive and will guide you through:
1. Project initialization
2. LLM provider configuration (Anthropic, Vertex AI, AI Gateway, or Claude Code)
3. Embedding provider configuration
4. Database connections
5. Context source configuration (dbt, Looker, Metabase, Notion)
6. Initial context ingestion
7. Agent integration installation

### Example Project Status Output

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

## Configuration

### Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

### ktx.yaml Configuration

```yaml
# LLM provider configuration
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # Credentials stored in .ktx/secrets.yaml

# Embedding provider configuration
embeddings:
  provider: openai
  model: text-embedding-3-small

# Database connections
connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    # Credentials in .ktx/secrets.yaml

# Context sources
context_sources:
  dbt_main:
    type: dbt
    project_dir: ./dbt_project
    profiles_dir: ~/.dbt
    target: prod
  
  notion_docs:
    type: notion
    # API token in .ktx/secrets.yaml
    database_ids:
      - abc123def456

# Agent integrations
agent_integrations:
  - type: codex
    scope: project
```

### Environment Variables

ktx respects these environment variables:

```bash
# Project directory override
export KTX_PROJECT_DIR=/path/to/project

# LLM provider credentials
export ANTHROPIC_API_KEY=your_key_here
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
export OPENAI_API_KEY=your_key_here

# Database credentials (example)
export DB_PASSWORD=your_db_password
```

## Core Commands

### Setup and Status

```bash
# Interactive setup wizard
ktx setup

# Check project status
ktx status

# Check specific component
ktx status --component llm
ktx status --component embeddings
ktx status --component connections
```

### Context Ingestion

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific source type
ktx ingest --source-type dbt
ktx ingest --source-type notion

# Force re-ingestion
ktx ingest --force
```

### Search Commands

```bash
# Search semantic layer sources
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn rate"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "data retention rules"

# Combined search with limit
ktx search "revenue metrics" --limit 10
```

### MCP Server

```bash
# Start MCP server for agent integration
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Start with custom port
ktx mcp start --port 3001
```

## Real-World Usage Examples

### Setting Up a New Data Analytics Project

```typescript
// Install ktx in your project
// In package.json:
{
  "name": "my-analytics-project",
  "scripts": {
    "ktx:setup": "ktx setup",
    "ktx:ingest": "ktx ingest",
    "ktx:status": "ktx status"
  },
  "devDependencies": {
    "@kaelio/ktx": "latest"
  }
}
```

```bash
# Run setup
npm run ktx:setup

# Verify configuration
npm run ktx:status

# Ingest initial context
npm run ktx:ingest
```

### Integrating with dbt Project

```yaml
# ktx.yaml
context_sources:
  dbt_production:
    type: dbt
    project_dir: ./dbt_project
    profiles_dir: ~/.dbt
    target: prod
    # ktx will ingest models, metrics, tests, and documentation
  
connections:
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    database: ANALYTICS
    warehouse: COMPUTE_WH
    role: ANALYST
    # Credentials in .ktx/secrets.yaml
```

```bash
# Ingest dbt metadata and Snowflake schema
ktx ingest --source-type dbt
ktx ingest --connection snowflake_prod
```

### Searching Semantic Layer from Agent

When integrated with Claude Code, Codex, or Cursor:

```text
User: What metrics do we have for revenue?

Agent uses ktx:
$ ktx sl "revenue"

Results:
- monthly_recurring_revenue (MRR)
  Definition: Sum of active subscription values normalized to monthly
  Source: semantic-layer/snowflake_prod/metrics.yaml
  
- annual_recurring_revenue (ARR)
  Definition: MRR * 12
  Source: semantic-layer/snowflake_prod/metrics.yaml
  
- net_revenue
  Definition: Gross revenue minus refunds and chargebacks
  Source: dbt_production/models/finance/net_revenue.sql
```

### Creating Custom Wiki Pages

```bash
# Create global wiki page
mkdir -p wiki/global
cat > wiki/global/refund-policy.md << 'EOF'
# Refund Policy

## Business Rules
- Refunds processed within 30 days of purchase
- Partial refunds allowed for annual subscriptions
- Refund amounts exclude transaction fees

## Data Impact
- Revenue recognition: Refunds reduce net_revenue metric
- Customer status: Refunded customers remain in customer_dim with is_active=false
- Reporting: Include refunds in monthly_revenue_adjustments table

## Related Metrics
- net_revenue
- refund_rate
- customer_lifetime_value
EOF

# Ingest wiki updates
ktx ingest --source-type wiki
```

### Semantic Layer YAML Format

```yaml
# semantic-layer/warehouse/customers.yaml
version: 1
type: semantic_source

sources:
  customers:
    description: Customer dimension table
    table: analytics.dim_customers
    columns:
      customer_id:
        type: integer
        description: Primary key
        primary_key: true
      email:
        type: string
        description: Customer email address
      created_at:
        type: timestamp
        description: Account creation timestamp
      
    metrics:
      total_customers:
        type: count
        description: Total number of customers
      
      active_customers:
        type: count_distinct
        sql: "CASE WHEN is_active THEN customer_id END"
        description: Count of currently active customers

    joins:
      - to: orders
        type: one_to_many
        sql_on: "customers.customer_id = orders.customer_id"
```

## Common Patterns

### 1. Regular Context Updates

```bash
#!/bin/bash
# scripts/update-ktx-context.sh

set -e

echo "Updating ktx context..."

# Pull latest dbt changes
cd dbt_project
git pull origin main
cd ..

# Re-ingest all sources
ktx ingest --force

# Verify status
ktx status

echo "Context update complete"
```

Add to cron or CI/CD pipeline:

```yaml
# .github/workflows/update-ktx.yml
name: Update ktx Context
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  workflow_dispatch:

jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install -g @kaelio/ktx
      - run: ktx ingest --force
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

### 2. Multi-Database Setup

```yaml
# ktx.yaml - Supporting multiple warehouses
connections:
  production_snowflake:
    type: snowflake
    account: prod.us-east-1
    database: ANALYTICS
    
  staging_postgres:
    type: postgres
    host: staging-db.example.com
    database: analytics_staging
    
  local_duckdb:
    type: duckdb
    path: ./local_data/analytics.duckdb

context_sources:
  dbt_prod:
    type: dbt
    project_dir: ./dbt_project
    target: prod
    connection: production_snowflake
  
  dbt_staging:
    type: dbt
    project_dir: ./dbt_project
    target: staging
    connection: staging_postgres
```

### 3. Agent Integration via MCP

```json
// Claude Code configuration
// .claude/mcp.json
{
  "mcpServers": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "/path/to/project"]
    }
  }
}
```

```json
// Cursor configuration
// .cursor/mcp.json
{
  "servers": {
    "ktx": {
      "command": "npx",
      "args": ["@kaelio/ktx", "mcp", "start"]
    }
  }
}
```

### 4. Custom Metric Definitions

```yaml
# semantic-layer/warehouse/revenue_metrics.yaml
version: 1
type: semantic_source

sources:
  revenue_analysis:
    description: Comprehensive revenue metrics
    
    metrics:
      mrr:
        type: sum
        sql: "subscription_value / EXTRACT(day FROM subscription_period_days) * 30"
        description: Monthly Recurring Revenue
        filters:
          - "subscription_status = 'active'"
        
      arr:
        type: derived
        sql: "mrr * 12"
        description: Annual Recurring Revenue
        
      net_revenue:
        type: sum
        sql: "gross_revenue - refund_amount - chargeback_amount"
        description: Net revenue after refunds and chargebacks
        
      revenue_per_customer:
        type: ratio
        numerator: net_revenue
        denominator: total_customers
        description: Average revenue per customer
```

## Troubleshooting

### "ktx mcp start" Required After Setup

**Symptom:** `ktx status` shows: "Please run: ktx mcp start --project-dir ..."

**Solution:** Start the MCP server before opening your agent client:

```bash
ktx mcp start --project-dir /path/to/project
```

Add to shell startup or run in a separate terminal.

### LLM Provider Authentication Errors

**Symptom:** `Error: Could not authenticate with LLM provider`

**Solution:** Check credentials in `.ktx/secrets.yaml` or environment variables:

```bash
# For Anthropic
export ANTHROPIC_API_KEY=sk-ant-...

# For OpenAI (embeddings)
export OPENAI_API_KEY=sk-...

# For Vertex AI
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json

# Verify
ktx status --component llm
```

### Database Connection Failures

**Symptom:** `Error: Failed to connect to database`

**Solution:** Test connection and verify credentials:

```bash
# Check connection config
cat ktx.yaml | grep -A 10 "connections:"

# Verify credentials
cat .ktx/secrets.yaml

# Test specific connection
ktx ingest --connection warehouse --dry-run
```

For Snowflake:
```bash
# Check account identifier format
# Should be: <account>.<region> (e.g., xy12345.us-east-1)
# NOT: https://xy12345.snowflakecomputing.com
```

### Ingestion Fails or Incomplete

**Symptom:** `ktx ingest` completes but no sources appear

**Solution:**

```bash
# Check raw-sources directory
ls -la raw-sources/

# Re-run with verbose logging
ktx ingest --force --verbose

# Check specific source type
ktx ingest --source-type dbt --verbose
```

For dbt sources:
```bash
# Ensure dbt can compile
cd dbt_project
dbt compile --target prod

# Verify profiles.yml
cat ~/.dbt/profiles.yml
```

### Semantic Layer Search Returns No Results

**Symptom:** `ktx sl "metric_name"` returns empty

**Solution:**

```bash
# Verify semantic layer files exist
ls -la semantic-layer/*/

# Check if ingestion completed
ktx status

# Rebuild embeddings
ktx ingest --force --connection <connection-id>

# Search with broader terms
ktx sl "revenue"  # vs "monthly_recurring_revenue"
```

### Wiki Search Not Finding Pages

**Symptom:** `ktx wiki "topic"` returns no results

**Solution:**

```bash
# Verify wiki directory structure
tree wiki/

# Wiki pages must be in:
# - wiki/global/ (shared)
# - wiki/user/<user-id>/ (personal)

# Re-ingest wiki
ktx ingest --source-type wiki --force

# Check file format (must be .md)
find wiki/ -name "*.md"
```

### MCP Server Won't Start

**Symptom:** `ktx mcp start` fails or agent can't connect

**Solution:**

```bash
# Check if port is in use
lsof -i :3000

# Use different port
ktx mcp start --port 3001

# Verify project directory
ktx mcp start --project-dir $(pwd)

# Check agent MCP configuration
cat .claude/mcp.json  # or .cursor/mcp.json
```

### Conflicting Metric Definitions

**Symptom:** ktx reports contradictions between sources

**Solution:**

ktx flags contradictions for human review. Check ingestion reports:

```bash
# View ingestion report
cat raw-sources/<connection>/ingestion-report.json

# Resolve by:
# 1. Updating source definitions to match
# 2. Marking one as deprecated
# 3. Documenting difference in wiki
```

### Performance Issues with Large Schemas

**Symptom:** Ingestion takes too long or times out

**Solution:**

```bash
# Ingest specific schemas only
# In ktx.yaml:
connections:
  warehouse:
    type: postgres
    schemas:
      - analytics  # Only ingest specific schemas
      - finance

# Sample fewer rows during introspection
# (Future ktx.yaml option)

# Split into multiple connections
connections:
  warehouse_analytics:
    type: postgres
    database: warehouse
    schemas: [analytics]
  
  warehouse_finance:
    type: postgres
    database: warehouse
    schemas: [finance]
```

## Best Practices

1. **Version control:** Commit `ktx.yaml`, `semantic-layer/`, and `wiki/` directories. Ignore `.ktx/`.

2. **Regular ingestion:** Schedule `ktx ingest` to run daily or when dbt models are updated.

3. **Wiki documentation:** Use `wiki/global/` for shared business context that agents should always know.

4. **Metric governance:** Define metrics in semantic layer YAML files, not scattered across SQL queries.

5. **Connection security:** Store credentials in `.ktx/secrets.yaml` (git-ignored) or environment variables, never in `ktx.yaml`.

6. **Agent prompts:** When asking agents to use ktx, be specific: "Use ktx to find the definition of MRR" vs. "What's MRR?"

7. **Multi-environment:** Create separate ktx projects or connections for dev/staging/prod environments.
