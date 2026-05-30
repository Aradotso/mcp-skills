---
name: ktx-data-agent-context-layer
description: Use ktx to build a self-improving context layer that teaches AI agents how to query data warehouses accurately with approved metrics, semantic layers, and business knowledge
triggers:
  - set up ktx for data analytics
  - configure ktx semantic layer
  - teach the agent about our data warehouse
  - use ktx to query our database
  - build context for data agents with ktx
  - integrate ktx with our analytics stack
  - add ktx skills to query data
  - configure ktx for business intelligence
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

ktx is an executable context layer that teaches AI agents how to query data warehouses accurately. It automatically builds and maintains approved metric definitions, discovers joinable columns, ingests business knowledge from wikis and semantic layers, and exposes everything through CLI and MCP tools for agent execution.

## What ktx Does

- **Learns from company knowledge**: Ingests dbt, Looker, Metabase, Notion content and organizes it
- **Maps the data stack**: Samples tables, detects joinable columns, annotates sources
- **Builds semantic layer**: Combines raw tables and metrics through a join graph that resolves chasm and fan traps
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite.

## Installation

```bash
npm install -g @kaelio/ktx
```

Or use locally without global install:

```bash
npx @kaelio/ktx setup
```

## Core Commands

### Initial Setup

```bash
# Interactive setup - creates or resumes a ktx project
ktx setup

# Check project readiness
ktx status

# Manual ingestion (usually automatic during setup)
ktx ingest

# Start MCP server for agent clients
ktx mcp start
```

### Search and Query

```bash
# Search semantic layer sources
ktx sl "revenue"
ktx sl "customer churn rate"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "data quality standards"

# Explain a semantic source in detail
ktx sl explain revenue_daily
```

### Advanced Commands

```bash
# Validate semantic layer YAML
ktx sl validate

# Refresh specific connection
ktx ingest --connection warehouse

# Export context for debugging
ktx export --format json --output context.json
```

## Configuration

### Project Structure

```text
my-project/
├── ktx.yaml                         # Main configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts
└── .ktx/                            # Local state (git-ignored)
```

### ktx.yaml Example

```yaml
version: 1
project:
  name: analytics
  description: Company analytics warehouse
  
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  api_key_env: ANTHROPIC_API_KEY
  
embeddings:
  provider: openai
  model: text-embedding-3-small
  api_key_env: OPENAI_API_KEY

databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    username: readonly_user
    password_env: DB_PASSWORD
    schema: public
    
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: PROD
    schema: PUBLIC
    username: service_account
    password_env: SNOWFLAKE_PASSWORD

context_sources:
  dbt_main:
    type: dbt
    profiles_dir: ~/.dbt
    project_dir: ./dbt
    target: prod
    
  looker_metrics:
    type: lookml
    project_dir: ./looker
    
  notion_wiki:
    type: notion
    token_env: NOTION_TOKEN
    database_id: a1b2c3d4e5f6
```

## Semantic Layer YAML

### Define a Metric Source

```yaml
# semantic-layer/warehouse/revenue_daily.yaml
version: 1
kind: metric_source
metadata:
  name: revenue_daily
  display_name: Daily Revenue
  description: Daily gross revenue aggregated from transactions
  owners: [data-team]
  tags: [finance, revenue]

source:
  connection: warehouse
  type: sql
  sql: |
    SELECT
      DATE(created_at) as date,
      product_id,
      region_id,
      SUM(amount) as revenue,
      COUNT(DISTINCT order_id) as order_count
    FROM transactions
    WHERE status = 'completed'
    GROUP BY 1, 2, 3

dimensions:
  - name: date
    type: date
    primary_key: true
  - name: product_id
    type: string
  - name: region_id
    type: string

metrics:
  - name: revenue
    type: sum
    sql: revenue
    format: currency
  - name: order_count
    type: sum
    sql: order_count
    format: number

joins:
  - to: products
    type: left
    on: product_id = products.id
  - to: regions
    type: left
    on: region_id = regions.id
```

### Define a Dimension Table

```yaml
# semantic-layer/warehouse/products.yaml
version: 1
kind: dimension_source
metadata:
  name: products
  display_name: Products
  description: Product catalog with categories

source:
  connection: warehouse
  type: table
  table: products

dimensions:
  - name: id
    type: string
    primary_key: true
  - name: name
    type: string
  - name: category
    type: string
  - name: price
    type: number
```

## TypeScript Integration

### Using ktx as a Library

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load existing project
const project = await KtxProject.load('/path/to/project');

// Search semantic layer
const slResults = await project.semanticLayer.search('revenue', {
  limit: 5,
  threshold: 0.7
});

for (const result of slResults) {
  console.log(`${result.metadata.display_name}: ${result.metadata.description}`);
  console.log(`Score: ${result.score}`);
}

// Search wiki
const wikiResults = await project.wiki.search('customer retention', {
  scope: 'global',
  limit: 3
});

// Get source details
const revenueSource = await project.semanticLayer.getSource('revenue_daily');
console.log(revenueSource.metrics);
```

### Custom Connector

```typescript
import { DatabaseConnector, ScanResult } from '@kaelio/ktx/connectors';

class CustomConnector implements DatabaseConnector {
  async connect(): Promise<void> {
    // Connection logic
  }

  async scan(): Promise<ScanResult> {
    const tables = await this.getTables();
    const samples = await this.sampleData(tables);
    
    return {
      tables,
      samples,
      relationships: this.detectJoins(tables, samples),
      metadata: this.extractMetadata(tables)
    };
  }

  private detectJoins(tables: Table[], samples: Sample[]): Relationship[] {
    // Join detection logic
    return [];
  }
}
```

## Agent Integration

### MCP Server Setup

```bash
# Start MCP server (usually automatic with agent clients)
ktx mcp start --project-dir /path/to/project

# Start with specific port
ktx mcp start --port 3000

# Check MCP server status
ktx mcp status
```

### Agent Configuration

For **Claude Code**, **Codex**, **Cursor**, or **OpenCode**:

```bash
# From your project directory, ask your agent:
# "Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install and configure ktx in this project."
```

Or configure manually in MCP settings:

```json
{
  "mcpServers": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "/path/to/project"]
    }
  }
}
```

## Common Patterns

### Daily Context Refresh

```bash
#!/bin/bash
# refresh-context.sh

cd /path/to/analytics-project
ktx ingest --connection warehouse
ktx sl validate
```

### Multi-Database Setup

```yaml
# ktx.yaml with multiple warehouses
databases:
  prod_warehouse:
    type: snowflake
    account: prod.us-east-1
    database: PROD
    password_env: PROD_DB_PASSWORD
    
  staging_warehouse:
    type: postgres
    host: staging-db.company.com
    database: staging
    password_env: STAGING_DB_PASSWORD

context_sources:
  prod_dbt:
    type: dbt
    target: prod
    database: prod_warehouse
    
  staging_dbt:
    type: dbt
    target: staging
    database: staging_warehouse
```

### Wiki Organization

```text
wiki/
├── global/
│   ├── metrics/
│   │   ├── revenue-definitions.md
│   │   └── customer-metrics.md
│   ├── data-quality/
│   │   └── validation-rules.md
│   └── onboarding/
│       └── data-stack-overview.md
└── user/
    └── alice/
        └── analysis-notes.md
```

Example wiki page:

```markdown
# Revenue Definitions

## Gross Revenue
Total transaction amount before refunds and discounts.
Source: `revenue_daily.revenue`

## Net Revenue
Gross revenue minus refunds.
Formula: `revenue_daily.revenue - refunds_daily.amount`

## ARR (Annual Recurring Revenue)
Subscription revenue normalized to annual value.
Only applies to subscription products.
```

### Query Pattern with ktx

```typescript
// Agent workflow using ktx context

// 1. Search for relevant metrics
const metrics = await project.semanticLayer.search('monthly recurring revenue');

// 2. Get the semantic source
const mrrSource = metrics[0];

// 3. Agent generates query using source metadata
const query = `
  SELECT 
    date,
    SUM(${mrrSource.metrics.find(m => m.name === 'mrr').sql}) as total_mrr
  FROM ${mrrSource.source.sql}
  WHERE date >= '2024-01-01'
  GROUP BY date
  ORDER BY date
`;

// 4. Execute against database connection
const results = await project.executeQuery('warehouse', query);
```

## Troubleshooting

### Setup Issues

**Problem**: `ktx setup` fails with "LLM provider not configured"

```bash
# Set required environment variables
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...

# Or use Claude Code session (no API key needed)
ktx setup --llm-provider claude-code
```

**Problem**: Database connection fails

```bash
# Test connection separately
export DB_PASSWORD=your_password
ktx setup --test-connection warehouse

# Check credentials in .ktx/secrets.env (git-ignored)
cat .ktx/secrets.env
```

### Ingestion Issues

**Problem**: `ktx ingest` times out on large database

```yaml
# ktx.yaml - limit scope
databases:
  warehouse:
    type: postgres
    # ... connection details ...
    schema: analytics  # Only scan this schema
    sample_size: 100   # Reduce sample size
    max_tables: 50     # Limit tables scanned
```

**Problem**: Duplicate or conflicting metrics

```bash
# Search for conflicts
ktx sl "revenue" --show-all

# Review validation report
ktx sl validate --verbose

# ktx flags contradictions in output - review and consolidate
```

### MCP Server Issues

**Problem**: Agent can't connect to MCP server

```bash
# Check if server is running
ktx mcp status

# Start manually with logging
ktx mcp start --verbose --project-dir $(pwd)

# Verify project path in agent MCP config matches ktx project location
```

**Problem**: "Project not ready" error

```bash
# Check project status
ktx status

# If context not built:
ktx ingest

# If semantic layer invalid:
ktx sl validate
```

### Performance Optimization

```yaml
# ktx.yaml - optimize for large projects
project:
  cache:
    enabled: true
    ttl: 3600
    
embeddings:
  batch_size: 50  # Smaller batches for rate limits
  
databases:
  warehouse:
    connection_pool_size: 5
    query_timeout: 30
```

## Environment Variables

```bash
# Required for non-Claude-Code LLM
export ANTHROPIC_API_KEY=sk-ant-...

# Required for embeddings
export OPENAI_API_KEY=sk-...

# Database credentials (referenced in ktx.yaml)
export DB_PASSWORD=...
export SNOWFLAKE_PASSWORD=...

# Optional: Notion integration
export NOTION_TOKEN=secret_...

# Optional: Project location override
export KTX_PROJECT_DIR=/path/to/project

# Optional: Disable telemetry
export KTX_TELEMETRY_DISABLED=1
```

## Validation

```bash
# Validate all semantic layer YAML
ktx sl validate

# Validate specific source
ktx sl validate revenue_daily

# Check for broken joins
ktx sl validate --check-joins

# Lint wiki markdown
ktx wiki validate
```

## Best Practices

1. **Commit semantic layer and wiki to git**, exclude `.ktx/`
2. **Use environment variables for all secrets**, never hardcode
3. **Run `ktx ingest` regularly** (nightly cron) to keep context fresh
4. **Organize wiki by domain** (metrics/, processes/, data-quality/)
5. **Tag semantic sources** for discoverability
6. **Set owners on metrics** for accountability
7. **Use read-only database credentials** - ktx never writes to your warehouse
8. **Validate before committing** changes to semantic layer YAML

## Example: Complete Setup Flow

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Set credentials
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export DB_PASSWORD=readonly_password

# 3. Initialize project
cd ~/my-analytics-project
ktx setup
# Follow interactive prompts:
# - Select LLM provider: Anthropic
# - Select embeddings: OpenAI
# - Add database connection: PostgreSQL
# - Add context source: dbt
# - Build initial context: Yes

# 4. Verify setup
ktx status

# 5. Create first semantic source
mkdir -p semantic-layer/warehouse
cat > semantic-layer/warehouse/users.yaml <<EOF
version: 1
kind: dimension_source
metadata:
  name: users
  display_name: Users
source:
  connection: warehouse
  type: table
  table: users
dimensions:
  - name: id
    type: string
    primary_key: true
  - name: email
    type: string
  - name: created_at
    type: timestamp
EOF

# 6. Validate and re-ingest
ktx sl validate
ktx ingest

# 7. Test search
ktx sl "users"
ktx wiki "user"

# 8. Start MCP for agents
ktx mcp start
```

Now your AI agent can query your warehouse using approved metrics and business context from ktx.
