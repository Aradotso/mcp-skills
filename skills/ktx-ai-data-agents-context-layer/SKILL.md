---
name: ktx-ai-data-agents-context-layer
description: An executable context layer that teaches AI agents to query data warehouses accurately using MCP, skills, and memory
triggers:
  - set up ktx for my data warehouse
  - configure ktx to work with my database
  - help me install and configure ktx
  - use ktx to query my data warehouse
  - build semantic layer context with ktx
  - search ktx wiki for business knowledge
  - integrate ktx with Claude Code
  - create ktx semantic sources for my metrics
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

ktx is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It automatically builds context from your databases, dbt models, wiki content, and business intelligence tools, then exposes this knowledge through MCP tools and CLI commands. Agents get approved metric definitions, joinable columns, and business knowledge instead of re-exploring your warehouse on every question.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, dbt docs, Looker/Metabase metadata, and organizes it automatically
- **Maps the data stack**: Samples tables, detects joinable columns, captures usage patterns
- **Builds a semantic layer**: Combines raw tables and metrics with automatic join resolution (chasm/fan trap detection)
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search across wiki and semantic sources
- **Read-only by design**: Never writes to your warehouse

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Local Installation

```bash
npm install @kaelio/ktx
npx ktx setup
```

### Via Agent Skill (Recommended)

From Claude Code, Cursor, Codex, or OpenCode:

```text
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install and configure ktx in this project.
```

## Initial Setup

The `ktx setup` command is interactive and handles:
- Project creation or resumption
- LLM provider configuration (Anthropic, Google Vertex, AI Gateway, Claude Code SDK)
- Database connection setup (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite)
- Context source configuration (dbt, MetricFlow, LookML, Looker, Metabase, Notion)
- Initial context ingestion
- Agent integration installation

```bash
ktx setup
```

After setup, verify project status:

```bash
ktx status
```

Expected output when ready:

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
├── semantic-layer/<connection-id>/  # YAML semantic sources
│   ├── metrics.yaml
│   └── entities.yaml
├── wiki/global/                     # Shared business context
│   └── refund-policy.md
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
│   ├── metadata.json
│   └── column-samples.json
└── .ktx/                            # Local state and secrets (git-ignored)
    ├── config.json
    └── embeddings.db
```

**Commit to version control**: `ktx.yaml`, `semantic-layer/`, `wiki/`  
**Keep local**: `.ktx/`

## Configuration

### ktx.yaml Example

```yaml
version: 1
project:
  name: my-analytics-project
  description: "Data warehouse context for agents"

llm:
  provider: anthropic
  model: claude-sonnet-4-20250514
  # API key set in .ktx/config.json or KTX_ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key set in .ktx/config.json or KTX_OPENAI_API_KEY

databases:
  warehouse:
    type: postgresql
    host: localhost
    port: 5432
    database: analytics
    schema: public
    # Credentials in .ktx/config.json or env vars
    read_only: true

context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./dbt/target/manifest.json
    catalog_path: ./dbt/target/catalog.json
  
  notion_docs:
    type: notion
    # NOTION_API_KEY in .ktx/config.json
    database_id: "your-notion-database-id"
```

### Environment Variables

```bash
# LLM Provider
export KTX_ANTHROPIC_API_KEY=your_anthropic_key
export KTX_OPENAI_API_KEY=your_openai_key
export KTX_GOOGLE_CLOUD_PROJECT=your_gcp_project

# Database Connections
export KTX_WAREHOUSE_USER=readonly_user
export KTX_WAREHOUSE_PASSWORD=your_password
export KTX_WAREHOUSE_HOST=db.example.com

# Notion Integration
export NOTION_API_KEY=your_notion_integration_key

# Project Directory
export KTX_PROJECT_DIR=/path/to/project
```

## Key Commands

### Status and Health

```bash
# Check project readiness
ktx status

# List all connections
ktx connections list

# Test a specific connection
ktx connections test warehouse
```

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific context source
ktx ingest --source dbt_main

# Force re-ingestion (ignore cache)
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer (metrics, entities)
ktx sl "revenue"
ktx sl "customer lifetime value"

# Search wiki content
ktx wiki "refund policy"
ktx wiki "how do we calculate churn"

# Search with JSON output for parsing
ktx sl "monthly recurring revenue" --json
```

### Semantic Layer Management

```bash
# List all semantic sources
ktx sl list

# Validate semantic layer definitions
ktx sl validate

# Show semantic source details
ktx sl show revenue_metric
```

### Wiki Management

```bash
# List wiki pages
ktx wiki list

# Create a new wiki page
ktx wiki create "Business Metrics Guide" \
  --content "Revenue is calculated as..."

# Edit existing page
ktx wiki edit revenue-definitions
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

## Semantic Layer Definition

### Defining Metrics

Create `semantic-layer/<connection-id>/metrics.yaml`:

```yaml
metrics:
  - name: monthly_recurring_revenue
    description: "Total monthly recurring revenue from active subscriptions"
    sql: |
      SELECT 
        DATE_TRUNC('month', subscription_start_date) as month,
        SUM(monthly_amount) as mrr
      FROM subscriptions
      WHERE status = 'active'
      GROUP BY 1
    dimensions:
      - month
    measures:
      - mrr
    filters:
      default:
        status: 'active'
    
  - name: customer_lifetime_value
    description: "Average revenue per customer over their lifetime"
    sql: |
      SELECT 
        customer_id,
        SUM(order_total) / COUNT(DISTINCT customer_id) as ltv
      FROM orders
      GROUP BY customer_id
    dimensions:
      - customer_id
    measures:
      - ltv
```

### Defining Entities

Create `semantic-layer/<connection-id>/entities.yaml`:

```yaml
entities:
  - name: customers
    description: "Customer master table"
    table: public.customers
    primary_key: customer_id
    attributes:
      - name: customer_id
        type: integer
        description: "Unique customer identifier"
      - name: email
        type: string
        description: "Customer email address"
      - name: created_at
        type: timestamp
        description: "Account creation timestamp"
    
  - name: orders
    description: "Order transactions"
    table: public.orders
    primary_key: order_id
    foreign_keys:
      - column: customer_id
        references: customers.customer_id
    attributes:
      - name: order_id
        type: integer
      - name: customer_id
        type: integer
      - name: order_total
        type: decimal
      - name: order_date
        type: date
```

## Using ktx from TypeScript

While ktx is primarily a CLI tool, you can use it programmatically:

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function searchSemanticLayer(query: string): Promise<any> {
  const { stdout } = await execAsync(`ktx sl "${query}" --json`);
  return JSON.parse(stdout);
}

async function searchWiki(query: string): Promise<any> {
  const { stdout } = await execAsync(`ktx wiki "${query}" --json`);
  return JSON.parse(stdout);
}

// Example usage
async function getRevenueContext() {
  const semanticResults = await searchSemanticLayer('revenue');
  const wikiResults = await searchWiki('revenue calculation');
  
  return {
    metrics: semanticResults.results,
    documentation: wikiResults.results
  };
}
```

## Agent Integration Patterns

### Using ktx with MCP Tools

When ktx MCP server is running, agents can call:

```typescript
// Agent calls ktx_search_semantic_layer tool
{
  "tool": "ktx_search_semantic_layer",
  "arguments": {
    "query": "customer churn rate",
    "limit": 5
  }
}

// Agent calls ktx_search_wiki tool
{
  "tool": "ktx_search_wiki",
  "arguments": {
    "query": "how we define active users",
    "limit": 3
  }
}

// Agent calls ktx_get_metric_definition tool
{
  "tool": "ktx_get_metric_definition",
  "arguments": {
    "metric_name": "monthly_recurring_revenue"
  }
}
```

### Agent Workflow Example

```text
User: What was our MRR last month?

Agent thinks:
1. Search ktx semantic layer for "MRR" definition
2. Use the canonical SQL from ktx metric
3. Execute query against warehouse
4. Return result with proper metric context

Agent executes:
- ktx_search_semantic_layer("MRR")
- Gets approved metric definition with SQL
- Runs query: SELECT month, mrr FROM ... WHERE month = '2025-04-01'
- Returns: "Based on the ktx metric definition, MRR in April 2025 was $X"
```

## Common Patterns

### Setting Up for a New Project

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Navigate to project
cd /path/to/analytics-project

# 3. Run interactive setup
ktx setup
# Follow prompts to configure LLM, database, and sources

# 4. Build initial context
ktx ingest

# 5. Verify readiness
ktx status

# 6. Start MCP server for agents
ktx mcp start
```

### Adding a New Data Source

```bash
# 1. Update ktx.yaml to add database connection
# Edit ktx.yaml:
# databases:
#   new_warehouse:
#     type: snowflake
#     account: abc123
#     ...

# 2. Test connection
ktx connections test new_warehouse

# 3. Ingest new source
ktx ingest --connection new_warehouse

# 4. Verify semantic layer
ktx sl list
```

### Creating Business Context

```bash
# Create wiki page for business logic
ktx wiki create "Churn Definition" --content "
# Customer Churn Definition

A customer is considered churned when:
- No active subscription for 30+ days
- No order placed in 90+ days
- Account status is 'cancelled'

SQL reference:
\`\`\`sql
SELECT customer_id
FROM customers
WHERE last_order_date < CURRENT_DATE - INTERVAL '90 days'
  AND subscription_status = 'cancelled'
\`\`\`
"

# Verify it's searchable
ktx wiki "churn definition"
```

### Updating Metrics

```bash
# 1. Edit semantic-layer/warehouse/metrics.yaml
# Update metric SQL or description

# 2. Validate changes
ktx sl validate

# 3. Re-ingest to update embeddings
ktx ingest --source semantic_layer --force

# 4. Test search
ktx sl "revenue"
```

## Troubleshooting

### ktx status shows "LLM ready: no"

```bash
# Check your API key is set
echo $KTX_ANTHROPIC_API_KEY

# Or set it in .ktx/config.json:
# {
#   "llm": {
#     "api_key": "your-key-here"
#   }
# }

# Re-run setup if needed
ktx setup
```

### Database connection fails

```bash
# Test connection explicitly
ktx connections test warehouse

# Check credentials in environment
echo $KTX_WAREHOUSE_USER
echo $KTX_WAREHOUSE_HOST

# Verify read-only access
# ktx will never write, but needs SELECT permissions
```

### MCP server not starting

```bash
# Check if server is already running
ktx mcp status

# Kill existing process if needed
pkill -f "ktx mcp"

# Start with debug output
ktx mcp start --verbose

# Check project directory is correct
ktx mcp start --project-dir /path/to/project
```

### Ingest fails or incomplete

```bash
# Check specific connection
ktx connections test warehouse

# Force clean re-ingest
ktx ingest --force

# Check logs for errors
ktx ingest --verbose

# Ingest one source at a time
ktx ingest --source dbt_main
```

### Search returns no results

```bash
# Verify context was ingested
ktx sl list
ktx wiki list

# Re-ingest to rebuild embeddings
ktx ingest --force

# Try broader search terms
ktx sl "revenue"  # instead of "monthly_recurring_revenue_for_enterprise_customers"
```

### Agent can't find ktx tools

```bash
# Verify MCP server is running
ktx mcp status

# Restart MCP server
ktx mcp start --project-dir $(pwd)

# Check agent configuration includes ktx MCP server
# For Claude Code: ensure ~/.config/claude/mcp.json includes ktx
# For Cursor: check MCP settings in preferences
```

## Advanced Configuration

### Multiple Database Connections

```yaml
databases:
  production:
    type: postgresql
    host: prod-db.example.com
    database: analytics
  
  staging:
    type: postgresql
    host: staging-db.example.com
    database: analytics
  
  snowflake_warehouse:
    type: snowflake
    account: abc123
    warehouse: COMPUTE_WH
    database: ANALYTICS
```

### Custom LLM Configuration

```yaml
llm:
  provider: vertex
  model: claude-3-5-sonnet-v2@20241022
  region: us-central1
  project_id: ${GOOGLE_CLOUD_PROJECT}
  
  # Or use AI Gateway
  # provider: ai-gateway
  # endpoint: https://gateway.example.com/v1
  # model: claude-sonnet-4
```

### Notion Integration

```yaml
context_sources:
  company_wiki:
    type: notion
    database_id: "a1b2c3d4e5f6"
    # Filters specific pages
    filter:
      property: "Tags"
      multi_select:
        contains: "Data"
```

## Best Practices

1. **Version control semantic layer**: Commit `semantic-layer/` and `wiki/` directories to track approved metric definitions
2. **Use descriptive metric names**: `monthly_recurring_revenue` not `mrr_calc`
3. **Document business logic in wiki**: Agents learn from both semantic layer SQL and wiki explanations
4. **Test connections before ingest**: Use `ktx connections test` to verify credentials
5. **Ingest regularly**: Set up cron/CI to run `ktx ingest` when dbt models or BI tools update
6. **Keep .ktx/ local**: Never commit API keys or embeddings database
7. **Use read-only database users**: ktx never writes, but principle of least privilege applies
8. **Start MCP server in project directory**: Ensures agents load the correct ktx project context

## Resources

- [Official Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Setup Guide](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
