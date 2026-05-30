---
name: ktx-ai-data-agents-context-layer
description: Context layer for AI data agents - teach Claude, Codex, and agents to query warehouses accurately through MCP with skills, memory, and semantic layers
triggers:
  - set up ktx for data agent queries
  - configure ktx context layer for analytics
  - build semantic layer with ktx
  - integrate ktx with Claude Code
  - teach agent to query warehouse with ktx
  - configure ktx MCP server for data analysis
  - set up ktx for approved metrics
  - use ktx to access warehouse data
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

ktx is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It automatically builds and maintains approved metric definitions, discovers joinable columns, resolves fan and chasm traps, and absorbs business knowledge from wikis, dbt, Looker, and other sources. Agents get one searchable surface combining semantic-layer metrics, raw-table introspection, and company knowledge—with contradictions flagged for human review.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds a semantic layer**: Combines raw tables and metrics through a join graph that resolves traps automatically
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search across wiki and semantic-layer entities

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite. Integrates with dbt, MetricFlow, LookML, Looker, Metabase, Notion.

## Installation

```bash
# Install globally
npm install -g @kaelio/ktx

# Or use as project dependency
npm install @kaelio/ktx

# Initialize ktx project
ktx setup
```

`ktx setup` creates or resumes a local ktx project, configures providers and connections, builds context, and installs agent integration.

## Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local and git-ignored.

## Configuration

### Basic ktx.yaml

```yaml
# Project identification
project:
  id: "analytics-warehouse"
  name: "Analytics Warehouse Context"

# LLM provider for context building
llm:
  provider: "anthropic"
  model: "claude-sonnet-4-6"
  # API key stored in .ktx/secrets.yaml or env var ANTHROPIC_API_KEY

# Embedding provider for semantic search
embeddings:
  provider: "openai"
  model: "text-embedding-3-small"
  # API key stored in .ktx/secrets.yaml or env var OPENAI_API_KEY

# Database connections
databases:
  warehouse:
    type: "postgres"
    # Connection details in .ktx/secrets.yaml or env vars
    # PG_HOST, PG_PORT, PG_DATABASE, PG_USER, PG_PASSWORD

# Context sources to ingest
contextSources:
  dbt_main:
    type: "dbt"
    manifestPath: "./target/manifest.json"
  
  notion_wiki:
    type: "notion"
    # NOTION_API_KEY in env or secrets

# Agent integration
agentIntegration:
  type: "codex"  # or "claude-code", "cursor", "opencode"
  scope: "project"
```

### Secrets Management

Secrets live in `.ktx/secrets.yaml` (git-ignored) or environment variables:

```yaml
# .ktx/secrets.yaml
llm:
  apiKey: "${ANTHROPIC_API_KEY}"

embeddings:
  apiKey: "${OPENAI_API_KEY}"

databases:
  warehouse:
    host: "${PG_HOST}"
    port: 5432
    database: "${PG_DATABASE}"
    user: "${PG_USER}"
    password: "${PG_PASSWORD}"

contextSources:
  notion_wiki:
    apiKey: "${NOTION_API_KEY}"
```

## Key Commands

### Setup and Status

```bash
# Create or resume ktx project
ktx setup

# Check project readiness
ktx status

# Output:
# ktx project: /home/user/analytics
# Project ready: yes
# LLM ready: yes (claude-sonnet-4-6)
# Embeddings ready: yes (text-embedding-3-small)
# Databases configured: yes (warehouse)
# Context sources configured: yes (dbt_main)
# ktx context built: yes
# Agent integration ready: yes (codex:project)
```

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific context source
ktx ingest --source dbt_main

# Force rebuild
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, sources)
ktx sl "revenue"
ktx sl "monthly active users"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "customer segmentation"

# Search with JSON output
ktx sl "revenue" --json
```

### MCP Server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with custom project directory
ktx mcp start --project-dir /path/to/project

# Start on specific port
ktx mcp start --port 8080
```

### Project Management

```bash
# Validate project configuration
ktx validate

# Clean build artifacts
ktx clean

# Export semantic layer
ktx export --format yaml --output ./export
```

## Using ktx in Code

### TypeScript/JavaScript Integration

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load project from directory
const project = await KtxProject.load('/path/to/project');

// Search semantic layer
const slResults = await project.searchSemanticLayer('revenue', {
  limit: 5,
  threshold: 0.7
});

console.log(slResults.map(r => r.entity.name));

// Search wiki
const wikiResults = await project.searchWiki('customer lifecycle', {
  limit: 10
});

wikiResults.forEach(r => {
  console.log(`${r.entity.title}: ${r.snippet}`);
});

// Get semantic source by ID
const source = await project.getSemanticSource('warehouse', 'orders');

// List all sources
const sources = await project.listSemanticSources('warehouse');
```

### Programmatic Ingestion

```typescript
import { ingestConnection, ingestSource } from '@kaelio/ktx';

// Ingest database connection
await ingestConnection(project, 'warehouse', {
  force: false,
  sampleSize: 1000
});

// Ingest context source
await ingestSource(project, 'dbt_main', {
  force: true
});

// Get ingestion status
const status = await project.getIngestionStatus('warehouse');
console.log(status.tablesScanned, status.metricsExtracted);
```

### Querying Semantic Layer

```typescript
import { querySemanticLayer } from '@kaelio/ktx';

// Declarative metric query
const result = await querySemanticLayer(project, {
  metrics: ['revenue', 'order_count'],
  dimensions: ['customer_segment', 'date_month'],
  filters: {
    date_year: { eq: 2024 },
    customer_segment: { in: ['enterprise', 'mid-market'] }
  },
  orderBy: [{ metric: 'revenue', direction: 'desc' }],
  limit: 100
});

console.log(result.sql);  // Generated SQL
console.log(result.rows);  // Query results
```

## Agent Integration Patterns

### Claude Code / Codex Integration

When ktx is configured for Codex, it exposes MCP tools:

```bash
# In your project directory, agent can use:
ktx_search_semantic_layer(query: "revenue")
ktx_search_wiki(query: "customer lifecycle")
ktx_get_metric(connection: "warehouse", metric: "mrr")
ktx_query_metrics(metrics: ["revenue", "cost"], dimensions: ["month"])
```

Example agent prompt:

```
Use ktx to find the definition of monthly recurring revenue,
then write a query to calculate MRR by customer segment for Q1 2024.
```

### Cursor Integration

```json
// .cursor/mcp.json
{
  "mcpServers": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "${workspaceFolder}"]
    }
  }
}
```

### Skills Integration

ktx provides a skill for self-installation:

```bash
# In agent, run:
npx skills add Kaelio/ktx --skill ktx
```

Then prompt:

```
Use the ktx skill to set up ktx in this project and configure it
for my Snowflake warehouse.
```

## Working with Semantic Sources

### Creating Semantic Sources (YAML)

```yaml
# semantic-layer/warehouse/orders.yaml
type: source
name: orders
description: Core orders table with transaction details
table: public.orders

columns:
  - name: order_id
    type: integer
    primaryKey: true
    
  - name: customer_id
    type: integer
    foreignKey: customers.customer_id
    
  - name: order_date
    type: date
    
  - name: amount
    type: decimal

metrics:
  - name: revenue
    description: Total order revenue
    type: sum
    sql: amount
    
  - name: order_count
    description: Number of orders
    type: count
    sql: order_id

dimensions:
  - name: order_month
    type: time
    sql: date_trunc('month', order_date)
```

### Defining Metrics

```yaml
# semantic-layer/warehouse/metrics.yaml
metrics:
  - name: mrr
    description: Monthly recurring revenue
    type: derived
    sql: |
      SELECT
        date_trunc('month', subscription_start) as month,
        sum(monthly_value) as value
      FROM subscriptions
      WHERE status = 'active'
      GROUP BY 1
    
  - name: customer_ltv
    description: Customer lifetime value
    type: derived
    sql: |
      SELECT
        customer_id,
        sum(order_amount) as value
      FROM orders
      WHERE order_date >= customer_first_order
      GROUP BY 1
```

## Wiki Integration

### Adding Wiki Pages

```bash
# Create global wiki page
cat > wiki/global/customer-lifecycle.md << EOF
# Customer Lifecycle

## Stages
1. **Lead**: Initial contact, not yet customer
2. **Trial**: Active trial user
3. **Customer**: Paid subscription
4. **Churned**: Cancelled subscription

## Metrics
- Trial conversion rate: trials -> customers / trials
- Churn rate: churned / (customers + churned) in period
EOF

# Ingest wiki
ktx ingest
```

### Notion Integration

```yaml
# ktx.yaml
contextSources:
  company_wiki:
    type: "notion"
    databaseId: "abc123..."  # Notion database ID
    # NOTION_API_KEY in env or .ktx/secrets.yaml
```

## Common Workflows

### Initial Setup Flow

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Initialize project
cd my-analytics-project
ktx setup
# Follow interactive prompts to configure LLM, embeddings, database

# 3. Verify configuration
ktx status

# 4. Build context
ktx ingest

# 5. Test search
ktx sl "revenue"
ktx wiki "customer"

# 6. Start MCP server for agents
ktx mcp start
```

### Adding New Data Source

```bash
# 1. Add to ktx.yaml
# databases:
#   new_warehouse:
#     type: "snowflake"

# 2. Add credentials to .ktx/secrets.yaml
# databases:
#   new_warehouse:
#     account: "${SNOWFLAKE_ACCOUNT}"
#     user: "${SNOWFLAKE_USER}"
#     password: "${SNOWFLAKE_PASSWORD}"

# 3. Ingest
ktx ingest --connection new_warehouse

# 4. Verify
ktx sl "tables from new_warehouse"
```

### Updating Context from dbt

```bash
# 1. Run dbt to generate manifest
cd dbt-project
dbt compile

# 2. Configure ktx to read manifest
# contextSources:
#   dbt_main:
#     type: "dbt"
#     manifestPath: "./dbt-project/target/manifest.json"

# 3. Ingest
ktx ingest --source dbt_main

# 4. Search for dbt models
ktx sl "dbt models"
```

## Troubleshooting

### "Project not found"

```bash
# Check project resolution
ktx status

# Explicitly set project directory
ktx status --project-dir /path/to/project

# Or set environment variable
export KTX_PROJECT_DIR=/path/to/project
ktx status
```

### "LLM provider not configured"

```bash
# Check configuration
ktx validate

# Set API key
export ANTHROPIC_API_KEY=sk-ant-...

# Or add to .ktx/secrets.yaml
echo "llm:\n  apiKey: \"${ANTHROPIC_API_KEY}\"" > .ktx/secrets.yaml

# Verify
ktx status
```

### "Database connection failed"

```bash
# Test connection directly
ktx ingest --connection warehouse --verbose

# Check credentials in .ktx/secrets.yaml
cat .ktx/secrets.yaml

# Verify environment variables
env | grep PG_

# Test with psql/mysql/etc directly
psql -h $PG_HOST -U $PG_USER -d $PG_DATABASE
```

### "MCP server won't start"

```bash
# Check if port is in use
lsof -i :8080

# Start on different port
ktx mcp start --port 8081

# Check logs
ktx mcp start --verbose

# Verify agent integration in ktx.yaml
ktx validate
```

### "Ingestion fails with large tables"

```bash
# Reduce sample size
ktx ingest --connection warehouse --sample-size 100

# Ingest specific schemas only
# In ktx.yaml:
# databases:
#   warehouse:
#     schemas: ["public", "analytics"]

# Skip large tables
# In ktx.yaml:
# databases:
#   warehouse:
#     excludeTables: ["raw_events", "logs"]
```

### "Contradictory metrics detected"

```bash
# Review flagged contradictions
ktx validate --verbose

# Example output:
# Warning: Metric 'revenue' defined differently:
#   - dbt: sum(amount) where status = 'paid'
#   - looker: sum(total) where cancelled_at is null
# Review and resolve in semantic-layer/warehouse/revenue.yaml
```

### "Semantic search returns poor results"

```bash
# Rebuild embeddings
ktx ingest --force

# Check embedding model configuration
ktx status

# Try different query phrasing
ktx sl "total revenue"
ktx sl "sum of order amounts"
ktx sl "sales metrics"

# Use JSON output to inspect scores
ktx sl "revenue" --json | jq '.[] | {name: .entity.name, score: .score}'
```

## Best Practices

1. **Commit semantic-layer and wiki**: These are source-controlled context that improves over time
2. **Keep .ktx/ local**: Contains secrets and local state
3. **Use environment variables for secrets**: Safer than hardcoding in YAML
4. **Ingest regularly**: Run `ktx ingest` after dbt runs or schema changes
5. **Review contradictions**: When ktx flags conflicts, resolve them in YAML
6. **Start specific**: Search for exact metric names before browsing
7. **Test queries locally**: Use `ktx sl` before exposing to agents
8. **Monitor agent usage**: Check `.ktx/logs/` for query patterns
9. **Version control ktx.yaml**: Track configuration changes with your code

## Advanced Configuration

### Custom Embedding Dimensions

```yaml
embeddings:
  provider: "openai"
  model: "text-embedding-3-large"
  dimensions: 1536  # Higher dimensions for better semantic search
```

### Multi-Database Setup

```yaml
databases:
  prod_warehouse:
    type: "snowflake"
    
  analytics_postgres:
    type: "postgres"
    
  clickhouse_events:
    type: "clickhouse"
```

### Selective Ingestion

```yaml
databases:
  warehouse:
    schemas: ["public", "analytics"]
    excludeTables: ["_temp_*", "staging_*"]
    sampleSize: 500
```
