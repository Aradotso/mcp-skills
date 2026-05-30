---
name: ktx-ai-data-agents-mcp-context-skills
description: Executable context layer for data and analytics agents with skills, memory, and semantic layer for accurate warehouse queries
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer
  - help me build context with ktx
  - integrate ktx with Claude Code
  - query warehouse using ktx context
  - create ktx semantic layer for metrics
  - ingest dbt models into ktx
  - set up ktx MCP server
---

# ktx AI Data Agents MCP Context Skills

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It combines approved metric definitions, joinable columns, wiki knowledge, and semantic layer modeling to give agents like Claude Code, Codex, and Cursor a unified understanding of your data stack.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content (Notion, Confluence), organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds semantic layer**: Combines raw tables and metrics through a join graph that resolves chasm and fan traps
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search across all context

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install --save-dev @kaelio/ktx
```

## Quick Setup

### Interactive Setup

The simplest way to get started:

```bash
ktx setup
```

This command:
1. Creates or resumes a ktx project
2. Configures LLM provider (Anthropic, Google Vertex AI, AI Gateway, or Claude Agent SDK)
3. Configures embeddings provider
4. Sets up database connections
5. Configures context sources (dbt, MetricFlow, LookML, Looker, Metabase, Notion)
6. Builds initial context
7. Installs agent integration

### Check Project Status

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

## Project Configuration

### ktx.yaml Structure

```yaml
version: "1"
providers:
  llm:
    type: anthropic
    model: claude-sonnet-4-6
  embeddings:
    type: openai
    model: text-embedding-3-small

databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    user: analytics_user
    # Password stored in .ktx/secrets.yaml

context-sources:
  dbt_main:
    type: dbt
    project-dir: ./dbt
    profiles-dir: ~/.dbt
    target: dev

  notion_docs:
    type: notion
    # Token stored in .ktx/secrets.yaml
    database-ids:
      - "abc123def456"

agent-integration:
  type: codex
  scope: project
```

### Environment Variables

Store secrets in environment variables:

```bash
export ANTHROPIC_API_KEY=your-api-key
export OPENAI_API_KEY=your-api-key
export NOTION_TOKEN=your-notion-token
```

## Key Commands

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific context source
ktx ingest --source dbt_main

# Validate semantic layer without ingesting
ktx validate-sl
```

### Searching Context

```bash
# Search semantic layer
ktx sl "monthly recurring revenue"

# Search wiki
ktx wiki "refund policy"

# Search with JSON output
ktx sl "revenue" --json
```

### MCP Server

```bash
# Start MCP server for agent integration
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status
```

### Managing Wiki

```bash
# Add wiki page
ktx wiki add "Customer Segmentation" --content "Enterprise customers..."

# Edit wiki page
ktx wiki edit "Customer Segmentation"

# List all wiki pages
ktx wiki list

# Remove wiki page
ktx wiki remove "Customer Segmentation"
```

## Database Connectors

### PostgreSQL

```yaml
databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    user: ${POSTGRES_USER}
    ssl: true
```

### Snowflake

```yaml
databases:
  snowflake:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: ${SNOWFLAKE_USER}
```

### BigQuery

```yaml
databases:
  bigquery:
    type: bigquery
    project: my-gcp-project
    dataset: analytics
    credentials-path: ${GOOGLE_APPLICATION_CREDENTIALS}
```

### ClickHouse

```yaml
databases:
  clickhouse:
    type: clickhouse
    host: localhost
    port: 8123
    database: analytics
    user: ${CLICKHOUSE_USER}
```

## Context Sources

### dbt

```yaml
context-sources:
  dbt_main:
    type: dbt
    project-dir: ./dbt
    profiles-dir: ~/.dbt
    target: dev
    exclude-patterns:
      - "staging.*"
      - "temp_*"
```

### MetricFlow

```yaml
context-sources:
  metrics:
    type: metricflow
    project-dir: ./dbt
    profiles-dir: ~/.dbt
```

### LookML

```yaml
context-sources:
  lookml:
    type: lookml
    project-dir: ./lookml
    models:
      - analytics
      - marketing
```

### Looker

```yaml
context-sources:
  looker:
    type: looker
    base-url: https://company.looker.com
    # Client ID and secret in .ktx/secrets.yaml
```

### Metabase

```yaml
context-sources:
  metabase:
    type: metabase
    base-url: https://metabase.company.com
    # Username and password in .ktx/secrets.yaml
```

### Notion

```yaml
context-sources:
  notion:
    type: notion
    database-ids:
      - "abc123"
      - "def456"
```

## Semantic Layer

### Creating Semantic Sources

Create YAML files in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/customers.yaml
name: customers
type: base
description: Core customer dimension
table: public.customers
columns:
  - name: customer_id
    type: dimension
    primary_key: true
  - name: email
    type: dimension
  - name: created_at
    type: dimension
  - name: lifetime_value
    type: measure
    aggregation: sum
```

### Defining Metrics

```yaml
# semantic-layer/warehouse/revenue.yaml
name: monthly_revenue
type: metric
description: Total revenue aggregated by month
sql: |
  SELECT
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS revenue
  FROM public.orders
  WHERE status = 'completed'
  GROUP BY 1
measures:
  - name: revenue
    type: measure
    aggregation: sum
dimensions:
  - name: month
    type: time
    granularity: month
```

### Join Relationships

ktx automatically detects joinable columns, but you can define explicit joins:

```yaml
# semantic-layer/warehouse/orders.yaml
name: orders
type: base
table: public.orders
joins:
  - to: customers
    type: many_to_one
    on:
      - from: customer_id
        to: customer_id
columns:
  - name: order_id
    type: dimension
    primary_key: true
  - name: customer_id
    type: dimension
  - name: amount
    type: measure
    aggregation: sum
```

## Agent Integration

### Codex Integration

```typescript
// Install ktx skill in Codex project
// From Codex: "Run npx skills add Kaelio/ktx --skill ktx"

// Use ktx in agent prompts:
// "Use ktx to find the definition of monthly recurring revenue"
// "Search our wiki for the refund policy using ktx"
// "Query the warehouse using ktx context for customer cohorts"
```

### MCP Tools Available to Agents

When MCP server is running, agents can use:

- `ktx_search_semantic_layer`: Search semantic sources (tables, metrics, columns)
- `ktx_search_wiki`: Search wiki pages
- `ktx_get_table_schema`: Get detailed table schema
- `ktx_get_metric_definition`: Get metric SQL and metadata
- `ktx_validate_query`: Validate SQL against semantic layer

### Using ktx in Claude Code

1. Start MCP server:
```bash
ktx mcp start
```

2. In Claude Code, reference ktx context:
```
Use ktx to find tables related to customer revenue.
Show me the definition of our MRR metric from ktx.
```

## Common Patterns

### Setting Up for a New Project

```bash
# Initialize project
cd /path/to/project
ktx setup

# Configure database
# (ktx setup will prompt)

# Add dbt as context source
# (ktx setup will prompt)

# Build context
ktx ingest

# Start MCP server for agents
ktx mcp start
```

### Adding Company Knowledge

```bash
# Add wiki page from file
ktx wiki add "Data Dictionary" --file ./docs/data-dictionary.md

# Add inline content
ktx wiki add "Revenue Definitions" --content "MRR is calculated as..."

# Ingest from Notion
# First configure in ktx.yaml:
```

```yaml
context-sources:
  notion:
    type: notion
    database-ids:
      - "your-database-id"
```

```bash
# Then ingest
ktx ingest --source notion
```

### Querying Context Programmatically

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function searchSemanticLayer(query: string) {
  const { stdout } = await execAsync(`ktx sl "${query}" --json`);
  return JSON.parse(stdout);
}

async function searchWiki(query: string) {
  const { stdout } = await execAsync(`ktx wiki "${query}" --json`);
  return JSON.parse(stdout);
}

// Usage
const revenueMetrics = await searchSemanticLayer('revenue');
const refundPolicy = await searchWiki('refund policy');
```

### Building Context in CI/CD

```yaml
# .github/workflows/ktx-context.yml
name: Build ktx Context
on:
  push:
    branches: [main]
    paths:
      - 'dbt/**'
      - 'semantic-layer/**'
      - 'wiki/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install -g @kaelio/ktx
      - run: ktx ingest
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
```

### Custom LLM Provider Configuration

```yaml
# Using AI Gateway
providers:
  llm:
    type: ai-gateway
    url: https://gateway.company.com/v1
    model: claude-sonnet-4-6
    # API key in .ktx/secrets.yaml

# Using Google Vertex AI
providers:
  llm:
    type: vertex
    project: my-gcp-project
    location: us-central1
    model: claude-sonnet-4-6
```

## Project Layout

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/
│   └── warehouse/                   # Per-connection semantic sources
│       ├── customers.yaml
│       ├── orders.yaml
│       └── revenue.yaml
├── wiki/
│   ├── global/                      # Shared business context
│   │   ├── data-dictionary.md
│   │   └── metric-definitions.md
│   └── user/
│       └── <user-id>/               # User-scoped notes
├── raw-sources/
│   └── warehouse/                   # Ingest artifacts
│       ├── sample-data.json
│       └── introspection-report.json
└── .ktx/                            # Local state (git-ignored)
    ├── secrets.yaml
    └── state.db
```

### Git Recommendations

```gitignore
# .gitignore
.ktx/
raw-sources/
```

Commit:
- `ktx.yaml`
- `semantic-layer/`
- `wiki/global/`

## Troubleshooting

### "LLM not ready" Error

```bash
# Check configured provider
ktx status

# Reconfigure LLM
ktx setup
# Select "Configure LLM provider"

# Verify API key
echo $ANTHROPIC_API_KEY
```

### "No project found" Error

```bash
# Specify project directory
ktx status --project-dir /path/to/project

# Or set environment variable
export KTX_PROJECT_DIR=/path/to/project
ktx status

# Or run from directory containing ktx.yaml
cd /path/to/project
ktx status
```

### MCP Server Won't Start

```bash
# Check if already running
ktx mcp status

# Stop existing server
ktx mcp stop

# Start with verbose logging
ktx mcp start --log-level debug
```

### Context Not Building

```bash
# Validate semantic layer first
ktx validate-sl

# Check database connection
ktx ingest --connection warehouse --dry-run

# Rebuild context
ktx ingest --force
```

### Database Connection Issues

```bash
# Test connection directly
ktx test-connection warehouse

# Check credentials in .ktx/secrets.yaml
cat .ktx/secrets.yaml

# Reconfigure connection
ktx setup
# Select "Configure database connections"
```

### Semantic Layer Validation Errors

```bash
# Validate specific source
ktx validate-sl --source customers

# Check for common issues:
# - Missing primary keys
# - Invalid join relationships
# - Duplicate column names
# - Invalid aggregation types
```

## Advanced Usage

### Custom Embedding Models

```yaml
providers:
  embeddings:
    type: openai
    model: text-embedding-3-large
    dimensions: 3072
```

### Excluding Tables from Ingestion

```yaml
databases:
  warehouse:
    type: postgres
    # ... connection config
    exclude-schemas:
      - temp
      - staging
    exclude-tables:
      - "*_backup"
      - "test_*"
```

### Multi-Database Projects

```yaml
databases:
  prod_warehouse:
    type: snowflake
    # ... config

  analytics_db:
    type: postgres
    # ... config

context-sources:
  dbt_prod:
    type: dbt
    database: prod_warehouse
    # ... config

  dbt_analytics:
    type: dbt
    database: analytics_db
    # ... config
```

### Scripting with ktx

```bash
#!/bin/bash
# sync-context.sh

set -e

echo "Syncing ktx context..."

# Pull latest dbt changes
git pull origin main

# Rebuild context
ktx ingest --source dbt_main

# Restart MCP server
ktx mcp stop
ktx mcp start

echo "Context synced successfully"
```

## Read-Only by Design

ktx never writes to your warehouse. All database connections are read-only. The tool only:
- Samples table data for context
- Reads schema metadata
- Analyzes query patterns

No INSERT, UPDATE, DELETE, or DDL statements are ever executed against your databases.
