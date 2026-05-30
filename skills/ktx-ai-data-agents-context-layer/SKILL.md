---
name: ktx-ai-data-agents-context-layer
description: Teach AI agents to query data warehouses accurately using ktx, a self-improving context layer with approved metrics, semantic layer, and wiki knowledge
triggers:
  - set up ktx for data warehouse access
  - configure ktx context layer for analytics
  - help me query the warehouse with ktx
  - add ktx to enable data agent queries
  - install ktx for semantic layer
  - configure ktx with dbt and warehouse
  - use ktx to build data context
  - integrate ktx with Claude Code for data queries
---

# ktx AI Data Agents Context Layer Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It ingests company knowledge (dbt, Looker, Metabase, Notion), builds a semantic layer with approved metric definitions, detects joinable columns, resolves fan/chasm traps, and exposes everything through CLI and MCP tools for agent execution.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds semantic layer**: Combines raw tables and metrics through a join graph that auto-resolves traps
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search across wiki and semantic entities
- **Read-only by design**: Never writes to your warehouse

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite. Integrates with dbt, MetricFlow, LookML, Looker, Metabase, Notion.

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Project-level Installation

```bash
npm install @kaelio/ktx
# or
pnpm add @kaelio/ktx
```

## Initial Setup

### Interactive Setup Wizard

```bash
ktx setup
```

The setup wizard will:
1. Create or resume a ktx project
2. Configure LLM provider (Anthropic, Google Vertex AI, AI Gateway, or Claude Code session)
3. Configure embedding provider (OpenAI, Google Vertex AI, Claude Agent SDK)
4. Add database connections
5. Add context sources (dbt, Looker, Metabase, Notion, raw warehouse)
6. Build initial context
7. Install agent integration (Codex, Claude Code)

### Check Project Status

```bash
ktx status
```

Example output:
```
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

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` in `.gitignore`.

## Configuration

### ktx.yaml Example

```yaml
version: 1
project:
  name: analytics
  default_connection: warehouse

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics_db
    schema: public
    # Credentials in .ktx/secrets.yaml

context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./dbt/target/manifest.json
    catalog_path: ./dbt/target/catalog.json
  
  warehouse_raw:
    type: warehouse
    connection: warehouse
    schemas:
      - public
      - staging
    sample_size: 1000

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key in .ktx/secrets.yaml or ANTHROPIC_API_KEY env var

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key in .ktx/secrets.yaml or OPENAI_API_KEY env var
```

### Environment Variables

```bash
# LLM providers
export ANTHROPIC_API_KEY=your_key_here
export GOOGLE_CLOUD_PROJECT=your_project
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Embedding providers
export OPENAI_API_KEY=your_key_here

# Database connections
export POSTGRES_PASSWORD=your_password
export SNOWFLAKE_PASSWORD=your_password

# Project location (optional)
export KTX_PROJECT_DIR=/path/to/project
```

### Adding Database Connections

```bash
# Interactive
ktx connection add

# Programmatic
ktx connection add \
  --id warehouse \
  --type postgres \
  --host localhost \
  --port 5432 \
  --database analytics_db \
  --user analytics_user
```

### Adding Context Sources

```bash
# dbt project
ktx source add \
  --id dbt_main \
  --type dbt \
  --manifest-path ./dbt/target/manifest.json \
  --catalog-path ./dbt/target/catalog.json

# Warehouse introspection
ktx source add \
  --id warehouse_raw \
  --type warehouse \
  --connection warehouse \
  --schemas public,staging

# Looker
ktx source add \
  --id looker \
  --type looker \
  --base-url https://company.looker.com \
  --client-id $LOOKER_CLIENT_ID \
  --client-secret $LOOKER_CLIENT_SECRET

# Notion workspace
ktx source add \
  --id notion \
  --type notion \
  --token $NOTION_INTEGRATION_TOKEN \
  --database-id $NOTION_DATABASE_ID
```

## Key Commands

### Building Context

```bash
# Build context from all configured sources
ktx ingest

# Build from specific source
ktx ingest --source dbt_main

# Rebuild everything from scratch
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, tables)
ktx sl "monthly recurring revenue"
ktx sl "customer lifetime value"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "how to calculate churn"

# Combined search (semantic + wiki)
ktx search "revenue metrics"
```

### MCP Server

```bash
# Start MCP server for agent integration
ktx mcp start

# Start with specific project
ktx mcp start --project-dir /path/to/project

# Check MCP status
ktx mcp status
```

### Semantic Layer Operations

```bash
# List all metrics
ktx sl list --type metric

# Show metric details
ktx sl get metric.mrr

# Validate semantic layer
ktx sl validate

# Generate SQL for metric query
ktx sl query --metric mrr --dimensions plan_type --filters "status = 'active'"
```

### Wiki Management

```bash
# Create wiki page
ktx wiki create "Refund Policy" --content "Our refund policy is..."

# Edit existing page
ktx wiki edit "Refund Policy"

# List all pages
ktx wiki list

# Search pages
ktx wiki search "refund"
```

## Real Usage Examples

### TypeScript: Using ktx Programmatically

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load project
const project = await KtxProject.load({
  projectDir: process.env.KTX_PROJECT_DIR || '.'
});

// Search semantic layer
const metrics = await project.semanticLayer.search('revenue', {
  type: 'metric',
  limit: 10
});

for (const metric of metrics) {
  console.log(`${metric.name}: ${metric.description}`);
  console.log(`SQL: ${metric.sql}`);
}

// Search wiki
const pages = await project.wiki.search('refund policy');
for (const page of pages) {
  console.log(`${page.title}: ${page.excerpt}`);
}

// Query with semantic layer
const result = await project.semanticLayer.query({
  metrics: ['mrr', 'arr'],
  dimensions: ['plan_type', 'region'],
  filters: {
    status: 'active',
    created_at: { gte: '2024-01-01' }
  }
});

console.log(result.sql);
console.log(result.data);
```

### Agent Integration: MCP Tools

When ktx MCP server is running, agents get these tools:

```typescript
// Tool: search_semantic_layer
{
  "query": "monthly recurring revenue",
  "type": "metric",
  "limit": 5
}

// Tool: search_wiki
{
  "query": "refund policy",
  "limit": 5
}

// Tool: get_metric_definition
{
  "metric_id": "mrr"
}

// Tool: generate_metric_query
{
  "metrics": ["mrr", "customer_count"],
  "dimensions": ["plan_type"],
  "filters": {
    "status": "active"
  }
}

// Tool: execute_query
{
  "connection_id": "warehouse",
  "sql": "SELECT plan_type, SUM(amount) FROM subscriptions GROUP BY 1"
}
```

### Example Agent Workflow

```typescript
// User asks: "What's our MRR by plan type?"

// 1. Agent uses search_semantic_layer
const metrics = await mcp.call('search_semantic_layer', {
  query: 'monthly recurring revenue',
  type: 'metric'
});
// Returns: metric.mrr definition

// 2. Agent uses generate_metric_query
const query = await mcp.call('generate_metric_query', {
  metrics: ['mrr'],
  dimensions: ['plan_type'],
  filters: { status: 'active' }
});
// Returns: { sql: "SELECT ...", joins: [...] }

// 3. Agent uses execute_query
const result = await mcp.call('execute_query', {
  connection_id: 'warehouse',
  sql: query.sql
});
// Returns: { rows: [...], columns: [...] }
```

## Common Patterns

### Pattern 1: Setting up ktx in Existing dbt Project

```bash
# Navigate to dbt project root
cd /path/to/dbt-project

# Initialize ktx
ktx setup

# During setup:
# - Choose existing database connection credentials
# - Add dbt as context source: ./target/manifest.json, ./target/catalog.json
# - Add warehouse introspection for raw tables
# - Build initial context

# Verify
ktx status
ktx sl list --type metric
```

### Pattern 2: Adding Custom Metrics

Create `semantic-layer/warehouse/metrics/custom.yaml`:

```yaml
metrics:
  - id: customer_lifetime_value
    name: Customer Lifetime Value
    description: Average revenue per customer over their lifetime
    type: derived
    sql: |
      AVG(total_revenue) OVER (
        PARTITION BY customer_id
        ORDER BY created_at
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
      )
    base_table: customers
    dimensions:
      - cohort_month
      - acquisition_channel
    filters:
      status: active
    
  - id: net_revenue_retention
    name: Net Revenue Retention
    description: Percentage of revenue retained from existing customers
    type: ratio
    numerator: expansion_revenue + renewal_revenue - churn_revenue
    denominator: prior_period_revenue
    base_table: revenue_summary
    dimensions:
      - period_month
```

Then rebuild context:

```bash
ktx ingest --source warehouse_raw
```

### Pattern 3: Syncing Wiki from Notion

```bash
# Add Notion as context source
ktx source add \
  --id notion_docs \
  --type notion \
  --token $NOTION_INTEGRATION_TOKEN \
  --database-id $NOTION_DATABASE_ID

# Ingest Notion content
ktx ingest --source notion_docs

# Search synced content
ktx wiki search "onboarding"
```

### Pattern 4: CI/CD Integration

```yaml
# .github/workflows/ktx-validate.yml
name: Validate ktx Context

on:
  pull_request:
    paths:
      - 'semantic-layer/**'
      - 'wiki/**'
      - 'ktx.yaml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install ktx
        run: npm install -g @kaelio/ktx
      
      - name: Validate semantic layer
        run: ktx sl validate
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
      
      - name: Check for contradictions
        run: ktx wiki validate --check-contradictions
```

### Pattern 5: Multi-Warehouse Setup

```yaml
# ktx.yaml
connections:
  prod_warehouse:
    type: snowflake
    account: company.snowflakecomputing.com
    warehouse: ANALYTICS_WH
    database: PROD
    
  dev_warehouse:
    type: postgres
    host: localhost
    database: dev_analytics

context_sources:
  prod_dbt:
    type: dbt
    manifest_path: ./dbt/prod/target/manifest.json
    connection: prod_warehouse
  
  dev_raw:
    type: warehouse
    connection: dev_warehouse
    schemas: [public, staging]
```

Query specific warehouse:

```bash
ktx sl query \
  --connection prod_warehouse \
  --metric mrr \
  --dimensions plan_type
```

## Troubleshooting

### Context Not Building

```bash
# Check configuration
ktx status

# Verify source connectivity
ktx connection test warehouse

# Review ingestion logs
ktx ingest --source dbt_main --verbose

# Force rebuild
ktx ingest --force --source dbt_main
```

### MCP Server Not Starting

```bash
# Check if already running
ktx mcp status

# Kill existing process
pkill -f "ktx mcp"

# Start with debug logs
ktx mcp start --log-level debug

# Verify project readiness
ktx status
```

### LLM Provider Issues

```bash
# Test LLM connection
ktx llm test

# Switch provider
ktx setup --reconfigure-llm

# Use different model
# Edit ktx.yaml:
llm:
  provider: anthropic
  model: claude-sonnet-4-20250514  # Latest model
```

### Semantic Layer Validation Errors

```bash
# Validate with details
ktx sl validate --verbose

# Check specific metric
ktx sl get metric.mrr --validate

# Common issues:
# - Missing base_table reference
# - Invalid SQL syntax in metric definition
# - Circular dependencies in derived metrics
# - Undefined dimensions or filters
```

### Wiki Contradictions

```bash
# Find contradictions
ktx wiki validate --check-contradictions

# Review flagged pages
ktx wiki list --status flagged

# Resolve by editing conflicting pages
ktx wiki edit "Metric Definitions"
```

### Database Connection Issues

```bash
# Test connection
ktx connection test warehouse

# Update credentials
ktx connection update warehouse --password $NEW_PASSWORD

# Check network/firewall
# Ensure database accepts connections from your IP
# Verify SSL/TLS requirements for cloud warehouses
```

### Permission Errors

```bash
# ktx needs read-only access
# Grant minimal permissions:
# PostgreSQL:
GRANT CONNECT ON DATABASE analytics_db TO analytics_user;
GRANT USAGE ON SCHEMA public TO analytics_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analytics_user;

# Snowflake:
GRANT USAGE ON WAREHOUSE ANALYTICS_WH TO ROLE analytics_role;
GRANT USAGE ON DATABASE PROD TO ROLE analytics_role;
GRANT USAGE ON SCHEMA PROD.PUBLIC TO ROLE analytics_role;
GRANT SELECT ON ALL TABLES IN SCHEMA PROD.PUBLIC TO ROLE analytics_role;
```

## Agent Best Practices

When using ktx with AI coding agents:

1. **Always run `ktx status`** before querying to ensure context is built
2. **Use semantic search first**: `ktx sl "revenue"` before writing SQL
3. **Check wiki for business logic**: `ktx wiki "how to calculate churn"`
4. **Prefer semantic layer queries** over raw SQL when approved metrics exist
5. **Update wiki** after discovering new business knowledge
6. **Rebuild context** after schema changes: `ktx ingest --force`
7. **Use MCP tools** when agent integration is configured for automatic context

## Resources

- **Documentation**: https://docs.kaelio.com/ktx
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Setup**: https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
- **Community Slack**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues**: https://github.com/Kaelio/ktx/issues
