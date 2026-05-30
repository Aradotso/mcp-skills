---
name: ktx-ai-data-agents-context-layer
description: Expert in using ktx, the executable context layer for data and analytics agents that enables accurate querying through MCP with skills, memory and a semantic layer
triggers:
  - set up ktx for my data warehouse
  - configure ktx to work with my database
  - help me build a semantic layer with ktx
  - integrate ktx with Claude Code for data queries
  - ingest my dbt or Looker definitions into ktx
  - search ktx wiki and semantic layer
  - configure ktx MCP server for agents
  - troubleshoot ktx context building
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It combines:

- **Automatic context building** from your warehouse schema, usage patterns, and joinable columns
- **Semantic layer** with approved metric definitions and automatic join graph resolution
- **Wiki integration** that absorbs knowledge from dbt, Looker, Metabase, Notion, and team documentation
- **MCP server** that exposes tools for agent execution with full-text and semantic search

**ktx** runs locally with your own LLM API keys (Anthropic, Google Vertex AI, AI Gateway, or Claude Code session). No hosted service, no extra usage billing.

## Installation

### Global installation

```bash
npm install -g @kaelio/ktx
```

### Project-specific installation

```bash
cd my-analytics-project
npx @kaelio/ktx setup
```

### Requirements

- Node.js 18+
- Python 3.9+ (for semantic layer query planning)
- A SQL warehouse: PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, or SQLite
- LLM provider: Anthropic API, Google Vertex AI, AI Gateway, or Claude Code

## Quick Setup

```bash
# Create or resume a ktx project
ktx setup

# Check project readiness
ktx status

# Build context from all configured sources
ktx ingest

# Start MCP server for agent integration
ktx mcp start
```

The `ktx setup` command walks through:
1. LLM provider configuration (API keys stored in `.ktx/`)
2. Database connection setup
3. Context source configuration (dbt, Looker, Metabase, Notion)
4. Initial context ingestion
5. Agent integration (Claude Code, Codex, Cursor, OpenCode)

## Project Structure

```
my-project/
├── ktx.yaml                         # Project configuration (commit)
├── semantic-layer/<connection-id>/  # YAML semantic sources (commit)
├── wiki/global/                     # Shared business context (commit)
├── wiki/user/<user-id>/             # User-scoped notes (commit)
├── raw-sources/<connection-id>/     # Ingest artifacts and reports (commit)
└── .ktx/                            # Local state and secrets (git-ignore)
```

## Configuration

### ktx.yaml

```yaml
version: 1
projectName: my-analytics
llmProvider: anthropic  # or google-vertex, ai-gateway, claude-agent-sdk
embeddingProvider: openai

databases:
  warehouse:
    type: postgres  # or snowflake, bigquery, clickhouse, mysql, mssql, sqlite
    host: "${DB_HOST}"
    port: 5432
    database: "${DB_NAME}"
    user: "${DB_USER}"
    password: "${DB_PASSWORD}"
    # For read-only safety:
    readOnly: true

contextSources:
  dbt_main:
    type: dbt
    manifestPath: ./target/manifest.json
    catalogPath: ./target/catalog.json
  
  looker_metrics:
    type: lookml
    projectPath: ./looker
  
  company_wiki:
    type: notion
    apiKey: "${NOTION_API_KEY}"
    databaseId: "${NOTION_DATABASE_ID}"
```

### Environment variables

Store secrets in `.ktx/env` or your shell environment:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export DB_HOST=warehouse.company.com
export DB_USER=readonly_user
export DB_PASSWORD=...
export NOTION_API_KEY=secret_...
```

## Key Commands

### Project management

```bash
# Initialize or resume project
ktx setup

# Check configuration and readiness
ktx status

# Validate configuration without running ingestion
ktx validate
```

### Context building

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Force re-ingestion (skip cache)
ktx ingest --force

# Dry run to preview changes
ktx ingest --dry-run
```

### Searching context

```bash
# Search semantic layer (metrics, dimensions, sources)
ktx sl "revenue"
ktx sl "customer churn rate"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "data governance"

# Combined search across all context
ktx search "active users"
```

### MCP server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with custom project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status

# Stop MCP server
ktx mcp stop
```

### Semantic layer operations

```bash
# List all semantic sources
ktx sl list

# Describe a specific metric
ktx sl describe revenue_total

# Validate semantic layer definitions
ktx sl validate

# Export semantic layer to JSON
ktx sl export --output ./semantic-layer.json
```

## Working with Database Connections

### PostgreSQL

```typescript
// In ktx.yaml
databases:
  warehouse:
    type: postgres
    host: "${POSTGRES_HOST}"
    port: 5432
    database: "${POSTGRES_DB}"
    user: "${POSTGRES_USER}"
    password: "${POSTGRES_PASSWORD}"
    schema: public  # Optional: default schema
    readOnly: true
```

### Snowflake

```yaml
databases:
  snowflake_prod:
    type: snowflake
    account: "${SNOWFLAKE_ACCOUNT}"
    warehouse: "${SNOWFLAKE_WAREHOUSE}"
    database: "${SNOWFLAKE_DATABASE}"
    schema: "${SNOWFLAKE_SCHEMA}"
    user: "${SNOWFLAKE_USER}"
    password: "${SNOWFLAKE_PASSWORD}"
    role: ANALYST  # Optional
    readOnly: true
```

### BigQuery

```yaml
databases:
  bigquery_prod:
    type: bigquery
    projectId: "${GCP_PROJECT_ID}"
    datasetId: "${BQ_DATASET_ID}"
    keyFilePath: "${GOOGLE_APPLICATION_CREDENTIALS}"
    readOnly: true
```

## Context Source Integration

### dbt

```yaml
contextSources:
  dbt_main:
    type: dbt
    manifestPath: ./target/manifest.json
    catalogPath: ./target/catalog.json
    # Optional: filter to specific models
    includePatterns:
      - "mart_*"
      - "dim_*"
      - "fct_*"
```

Before ingesting, ensure dbt artifacts are up to date:

```bash
dbt compile
dbt docs generate
ktx ingest --connection dbt_main
```

### Looker / LookML

```yaml
contextSources:
  looker_metrics:
    type: lookml
    projectPath: ./looker-project
    # Optional: specific view files
    includePatterns:
      - "views/metrics/*.lkml"
```

### Metabase

```yaml
contextSources:
  metabase_metrics:
    type: metabase
    apiUrl: "${METABASE_URL}"
    apiKey: "${METABASE_API_KEY}"
    # Optional: filter to specific collections
    collectionIds:
      - 42
      - 57
```

### Notion

```yaml
contextSources:
  company_wiki:
    type: notion
    apiKey: "${NOTION_API_KEY}"
    databaseId: "${NOTION_DATABASE_ID}"
    # Optional: filter by tags or properties
    filters:
      tags:
        - analytics
        - data-dictionary
```

## Semantic Layer Definitions

### Basic metric

```yaml
# semantic-layer/warehouse/revenue_total.yaml
apiVersion: v1
kind: Metric
metadata:
  name: revenue_total
  display_name: Total Revenue
  description: Sum of all completed order amounts
spec:
  type: measure
  aggregation: sum
  sql: "{{source.orders}}.amount"
  filters:
    - "{{source.orders}}.status = 'completed'"
  dimensions:
    - date
    - customer_id
    - product_id
```

### Derived metric

```yaml
# semantic-layer/warehouse/revenue_per_customer.yaml
apiVersion: v1
kind: Metric
metadata:
  name: revenue_per_customer
  display_name: Revenue per Customer
  description: Average revenue per unique customer
spec:
  type: ratio
  numerator: revenue_total
  denominator: customers_count
  dimensions:
    - date
    - region
```

### Dimension

```yaml
# semantic-layer/warehouse/customer_segment.yaml
apiVersion: v1
kind: Dimension
metadata:
  name: customer_segment
  display_name: Customer Segment
  description: Customer categorization by lifetime value
spec:
  type: categorical
  sql: |
    CASE
      WHEN {{source.customers}}.lifetime_value > 10000 THEN 'enterprise'
      WHEN {{source.customers}}.lifetime_value > 1000 THEN 'mid-market'
      ELSE 'smb'
    END
  source: customers
```

### Source table

```yaml
# semantic-layer/warehouse/orders.yaml
apiVersion: v1
kind: Source
metadata:
  name: orders
  display_name: Orders
  description: All customer orders
spec:
  table: public.orders
  primaryKey: id
  joins:
    - name: customers
      type: left
      sql: "{{source.orders}}.customer_id = {{source.customers}}.id"
    - name: products
      type: left
      sql: "{{source.orders}}.product_id = {{source.products}}.id"
  columns:
    - name: id
      type: integer
      primaryKey: true
    - name: customer_id
      type: integer
    - name: product_id
      type: integer
    - name: amount
      type: decimal
    - name: status
      type: string
    - name: created_at
      type: timestamp
```

## Agent Integration

### Claude Code

After running `ktx setup`, ktx installs itself as an MCP server. In Claude Code:

```bash
# From your project directory
ktx mcp start --project-dir .
```

Then in Claude Code, verify MCP server is connected:

```
/mcp-servers
```

You should see `ktx` listed. Now ask:

```
Use ktx to show me total revenue by month for the last quarter
```

### Codex

```bash
# Install ktx skill in Codex
npx skills add Kaelio/ktx --skill ktx

# Verify installation
npx skills list
```

Then in Codex:

```
@ktx search for customer churn metrics
```

### Cursor / OpenCode

Configure MCP server in settings:

```json
{
  "mcpServers": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "${workspaceFolder}"]
    }
  }
}
```

## MCP Tools

When the MCP server is running, agents can call:

### `ktx_search_semantic_layer`

```typescript
// Agent calls this tool
{
  "tool": "ktx_search_semantic_layer",
  "arguments": {
    "query": "revenue by customer segment",
    "limit": 10
  }
}
```

Returns ranked results with metric definitions, SQL, dimensions, and filters.

### `ktx_search_wiki`

```typescript
{
  "tool": "ktx_search_wiki",
  "arguments": {
    "query": "refund policy",
    "limit": 5
  }
}
```

Returns wiki pages with full-text and semantic search.

### `ktx_get_metric`

```typescript
{
  "tool": "ktx_get_metric",
  "arguments": {
    "metricName": "revenue_total"
  }
}
```

Returns full metric definition, SQL, join graph, and usage examples.

### `ktx_list_sources`

```typescript
{
  "tool": "ktx_list_sources",
  "arguments": {
    "connectionId": "warehouse"
  }
}
```

Returns all semantic sources (tables) for a connection.

### `ktx_validate_query`

```typescript
{
  "tool": "ktx_validate_query",
  "arguments": {
    "sql": "SELECT revenue_total FROM metrics WHERE date > '2024-01-01'",
    "connectionId": "warehouse"
  }
}
```

Validates SQL against semantic layer and returns suggestions.

## Common Patterns

### Initial setup for a new project

```bash
# Navigate to your analytics project
cd my-analytics-project

# Initialize ktx
ktx setup

# Configure LLM provider (follow prompts)
# - Choose anthropic, google-vertex, or ai-gateway
# - Enter API key (stored in .ktx/env)

# Configure database connection
# - Choose your warehouse type
# - Enter connection details
# - Test connection

# Add context sources
# - Select dbt, lookml, metabase, or notion
# - Provide paths or API credentials

# Build initial context
ktx ingest

# Verify everything is ready
ktx status
```

### Daily workflow

```bash
# Update context after schema changes
ktx ingest

# Search for a metric
ktx sl "active users"

# Get metric SQL
ktx sl describe active_users_monthly

# Start MCP server for agent work
ktx mcp start
```

### Adding a new metric

```bash
# Create YAML in semantic-layer/<connection-id>/
cat > semantic-layer/warehouse/new_metric.yaml <<EOF
apiVersion: v1
kind: Metric
metadata:
  name: new_metric
  display_name: New Metric
  description: Description of what this measures
spec:
  type: measure
  aggregation: sum
  sql: "{{source.table}}.column"
  dimensions:
    - date
EOF

# Validate
ktx sl validate

# Re-ingest to make available
ktx ingest
```

### Troubleshooting connection issues

```bash
# Test database connection
ktx validate --connection warehouse

# Check logs
cat .ktx/logs/latest.log

# Re-run setup to update credentials
ktx setup

# Force re-ingestion with verbose output
ktx ingest --force --verbose
```

### Working with multiple environments

```yaml
# ktx.yaml
databases:
  prod:
    type: postgres
    host: "${PROD_DB_HOST}"
    database: production
    readOnly: true
  
  staging:
    type: postgres
    host: "${STAGING_DB_HOST}"
    database: staging
    readOnly: true
```

```bash
# Ingest specific environment
ktx ingest --connection prod
ktx ingest --connection staging

# Search within specific connection
ktx sl "revenue" --connection prod
```

## Troubleshooting

### "LLM provider not configured"

```bash
# Re-run setup
ktx setup

# Or manually set in .ktx/env
echo "ANTHROPIC_API_KEY=sk-ant-..." >> .ktx/env
```

### "Database connection failed"

```bash
# Validate connection details
ktx validate --connection warehouse

# Check read-only user has SELECT permissions
# Check network connectivity and firewall rules
# Verify credentials in .ktx/env or environment
```

### "Context ingestion failed"

```bash
# Check specific error in logs
cat .ktx/logs/latest.log

# Common issues:
# - dbt artifacts not generated: run `dbt compile && dbt docs generate`
# - Notion API key expired: regenerate in Notion integrations
# - Looker project path incorrect: verify path in ktx.yaml

# Force re-ingestion
ktx ingest --force --verbose
```

### "MCP server not starting"

```bash
# Check if port is already in use
ktx mcp status

# Stop existing server
ktx mcp stop

# Start with explicit project directory
ktx mcp start --project-dir /full/path/to/project

# Check MCP logs
cat .ktx/logs/mcp-server.log
```

### "Semantic layer validation errors"

```bash
# Validate all definitions
ktx sl validate

# Common issues:
# - Invalid YAML syntax: check indentation
# - Missing source references: ensure source exists in semantic-layer/
# - Circular joins: review join graph in raw-sources/

# Fix and re-validate
ktx sl validate
ktx ingest --force
```

### "Agent can't find ktx tools"

```bash
# Ensure MCP server is running
ktx mcp status

# Restart MCP server
ktx mcp stop
ktx mcp start

# In Claude Code, verify MCP connection:
# /mcp-servers should list ktx

# In Cursor/OpenCode, check MCP settings
```

## Advanced Usage

### Custom LLM provider

```yaml
# ktx.yaml
llmProvider: ai-gateway
llmConfig:
  endpoint: "${AI_GATEWAY_ENDPOINT}"
  apiKey: "${AI_GATEWAY_API_KEY}"
  model: claude-3-5-sonnet-20241022
```

### Python API for custom workflows

```python
# Use ktx semantic layer in Python
from ktx_sl import SemanticLayer

sl = SemanticLayer.from_project_dir("./my-project")

# Query metrics
result = sl.query(
    metrics=["revenue_total", "customers_count"],
    dimensions=["date"],
    filters={"date": {"gte": "2024-01-01"}},
    granularity="month"
)

print(result.to_dataframe())
```

### CI/CD integration

```bash
# Validate in CI
ktx validate --strict

# Export semantic layer for documentation
ktx sl export --output ./docs/metrics.json

# Test queries against semantic layer
ktx sl validate --test-queries ./tests/queries.sql
```

## Resources

- [Documentation](https://docs.kaelio.com/ktx/docs/)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
