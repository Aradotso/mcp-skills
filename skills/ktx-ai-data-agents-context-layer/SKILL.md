---
name: ktx-ai-data-agents-context-layer
description: Executable context layer for data and analytics agents with semantic layer, wiki knowledge, and MCP integration for accurate warehouse querying
triggers:
  - set up ktx for data agents
  - configure ktx context layer
  - build semantic layer with ktx
  - integrate ktx with claude code
  - query warehouse using ktx
  - ingest data context into ktx
  - search ktx semantic layer
  - configure ktx database connections
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It automatically builds and maintains:

- **Semantic layer** with approved metric definitions, joinable columns, and automatic fan/chasm trap resolution
- **Wiki knowledge** from Notion, dbt docs, Looker, Metabase, and other sources
- **Database metadata** through introspection and usage pattern detection
- **MCP server** exposing tools for Claude Code, Codex, Cursor, and other agents

Unlike traditional semantic layers, ktx absorbs company knowledge automatically, detects contradictions, and serves agents through CLI and MCP interfaces.

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install --save-dev @kaelio/ktx
```

### Using with Agent Skills

In Claude Code, Codex, Cursor, or OpenCode:

```bash
npx skills add Kaelio/ktx --skill ktx
```

## Setup and Configuration

### Initial Setup

Run the interactive setup wizard:

```bash
ktx setup
```

This creates:
- `ktx.yaml` - project configuration
- `semantic-layer/` - YAML semantic sources
- `wiki/global/` - shared business context
- `.ktx/` - local state and secrets (git-ignored)

### Manual Configuration Example

```yaml
# ktx.yaml
version: 1

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key from environment: ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key from environment: OPENAI_API_KEY

databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    # Credentials from environment: POSTGRES_USER, POSTGRES_PASSWORD
    read_only: true

context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./dbt/target/manifest.json
    catalog_path: ./dbt/target/catalog.json
  
  looker_metrics:
    type: lookml
    project_dir: ./looker
  
  notion_wiki:
    type: notion
    # Token from environment: NOTION_TOKEN
    database_ids:
      - abc123def456
```

### LLM Provider Configuration

#### Anthropic API

```bash
export ANTHROPIC_API_KEY=your_key_here
```

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
```

#### Google Vertex AI

```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
```

```yaml
llm:
  provider: vertex
  model: claude-sonnet-4-6
  project: my-gcp-project
  region: us-central1
```

#### Claude Code Session (local)

```yaml
llm:
  provider: claude-agent-sdk
```

### Database Connection Examples

#### PostgreSQL

```yaml
databases:
  analytics:
    type: postgres
    host: db.example.com
    port: 5432
    database: warehouse
    schema: public
    user: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}
    ssl: true
    read_only: true
```

#### Snowflake

```yaml
databases:
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}
    role: ANALYST_ROLE
```

#### BigQuery

```yaml
databases:
  bigquery_prod:
    type: bigquery
    project: my-gcp-project
    dataset: analytics
    credentials_path: ${GOOGLE_APPLICATION_CREDENTIALS}
```

## Key Commands

### Check Project Status

```bash
ktx status
```

Output example:
```
ktx project: /home/user/analytics
Project ready: yes
LLM ready: yes (claude-sonnet-4-6)
Embeddings ready: yes (text-embedding-3-small)
Databases configured: yes (warehouse)
Context sources configured: yes (dbt_main, looker_metrics)
ktx context built: yes
Agent integration ready: yes (codex:project)
```

### Build Context

Ingest metadata and build semantic layer:

```bash
ktx ingest
```

Ingest specific connection:

```bash
ktx ingest --connection warehouse
```

Force rebuild:

```bash
ktx ingest --force
```

### Search Semantic Layer

```bash
ktx sl "revenue"
ktx sl "active users last 30 days"
ktx sl "customer lifetime value"
```

Returns metric definitions, dimensions, and joinable entities.

### Search Wiki

```bash
ktx wiki "refund policy"
ktx wiki "data retention"
ktx wiki "how to calculate MRR"
```

### MCP Server for Agents

Start the MCP server:

```bash
ktx mcp start
```

With specific project directory:

```bash
ktx mcp start --project-dir /path/to/analytics
```

## Semantic Layer Definition

### Creating Semantic Sources

Create YAML files in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/users.yaml
version: 1
name: users
description: Core user entity with signup and activity metrics

table:
  database: analytics
  schema: public
  name: users

primary_key: user_id

dimensions:
  - name: user_id
    type: string
    column: id
    primary: true
  
  - name: signup_date
    type: date
    column: created_at
  
  - name: user_type
    type: string
    column: type
    values:
      - free
      - paid
      - enterprise

metrics:
  - name: total_users
    type: count
    description: Total number of users
  
  - name: paid_users
    type: count
    description: Number of paid users
    filters:
      - dimension: user_type
        operator: in
        values: [paid, enterprise]
  
  - name: signup_rate
    type: derived
    description: Daily user signups
    sql: |
      COUNT(*) / NULLIF(COUNT(DISTINCT DATE(created_at)), 0)
```

### Join Definitions

```yaml
# semantic-layer/warehouse/orders.yaml
version: 1
name: orders
description: Order transactions

table:
  database: analytics
  schema: public
  name: orders

primary_key: order_id

joins:
  - name: user
    to: users
    type: left
    on:
      - left: user_id
        right: user_id

dimensions:
  - name: order_id
    type: string
    column: id
    primary: true
  
  - name: order_date
    type: timestamp
    column: created_at
  
  - name: amount
    type: number
    column: total_amount

metrics:
  - name: total_revenue
    type: sum
    column: amount
    description: Sum of all order amounts
  
  - name: average_order_value
    type: average
    column: amount
    description: Average order amount
```

## Wiki Management

### Adding Wiki Pages

Create markdown files in `wiki/global/`:

```markdown
<!-- wiki/global/revenue-definition.md -->
# Revenue Definition

## Overview
Revenue is recognized when payment is received and service is delivered.

## Calculation
- Include: subscription fees, one-time purchases
- Exclude: refunds, credits, promotional discounts

## Metrics
- MRR: Monthly recurring revenue from active subscriptions
- ARR: Annual recurring revenue (MRR × 12)

## Related Tables
- `payments` - raw payment transactions
- `subscriptions` - active and historical subscriptions
- `refunds` - customer refunds
```

### User-Scoped Notes

Create in `wiki/user/<user-id>/`:

```markdown
<!-- wiki/user/alice/analysis-notes.md -->
# Q1 Revenue Analysis Notes

Working hypothesis: churn spike in March due to pricing change.

TODO:
- Compare March cohort retention vs Q4
- Check support tickets during rollout
```

## Agent Integration Patterns

### Query Warehouse with Context

TypeScript example using ktx from an agent:

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function searchSemanticLayer(query: string): Promise<string> {
  const { stdout } = await execAsync(`ktx sl "${query}"`);
  return stdout;
}

async function searchWiki(query: string): Promise<string> {
  const { stdout } = await execAsync(`ktx wiki "${query}"`);
  return stdout;
}

// Agent workflow
async function analyzeRevenue() {
  // 1. Find relevant metrics
  const revenueMetrics = await searchSemanticLayer('revenue metrics');
  
  // 2. Check business context
  const revenuePolicy = await searchWiki('revenue recognition policy');
  
  // 3. Agent can now generate accurate SQL using approved definitions
  console.log('Metrics:', revenueMetrics);
  console.log('Policy:', revenuePolicy);
}
```

### MCP Tool Usage

When ktx MCP server is running, agents can call tools directly:

```json
{
  "tool": "ktx_search_semantic_layer",
  "arguments": {
    "query": "monthly recurring revenue"
  }
}
```

```json
{
  "tool": "ktx_search_wiki",
  "arguments": {
    "query": "how to calculate customer lifetime value"
  }
}
```

```json
{
  "tool": "ktx_get_metric_definition",
  "arguments": {
    "metric_name": "total_revenue",
    "source": "orders"
  }
}
```

## Common Patterns

### Initial Project Setup for Data Analysis

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Navigate to analytics project
cd ~/projects/analytics

# 3. Run setup wizard
ktx setup

# 4. Build initial context
ktx ingest

# 5. Verify everything works
ktx status

# 6. Test semantic search
ktx sl "revenue"
ktx wiki "metric definitions"

# 7. Start MCP for agents
ktx mcp start
```

### Adding New Data Source

```yaml
# Add to ktx.yaml
databases:
  new_warehouse:
    type: snowflake
    account: ${SNOWFLAKE_ACCOUNT}
    warehouse: ANALYTICS_WH
    database: REPORTING
    schema: PUBLIC
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}
```

```bash
# Ingest new source
ktx ingest --connection new_warehouse

# Verify
ktx status
```

### Updating Context After Schema Changes

```bash
# Force rebuild context
ktx ingest --force

# Or rebuild specific connection
ktx ingest --connection warehouse --force
```

### Integrating dbt Semantic Layer

```yaml
# ktx.yaml
context_sources:
  dbt_analytics:
    type: dbt
    manifest_path: ./dbt/target/manifest.json
    catalog_path: ./dbt/target/catalog.json
```

```bash
# After dbt runs
cd dbt && dbt compile && dbt docs generate && cd ..

# Ingest updated dbt metadata
ktx ingest --source dbt_analytics
```

### Importing Looker Metrics

```yaml
# ktx.yaml
context_sources:
  looker_dashboards:
    type: lookml
    project_dir: ./looker
    models:
      - users
      - orders
      - analytics
```

```bash
ktx ingest --source looker_dashboards
```

## Troubleshooting

### ktx status shows "ktx context built: no"

**Solution:** Run initial ingestion:

```bash
ktx ingest
```

### MCP server not found by agent

**Solution:** Ensure MCP server is running:

```bash
ktx mcp start --project-dir $(pwd)
```

Check agent configuration includes ktx MCP server path.

### Database connection fails

**Solution:** Verify credentials and network access:

```bash
# Test connection manually
psql -h db.example.com -U analytics_user -d warehouse

# Check environment variables
echo $POSTGRES_USER
echo $POSTGRES_PASSWORD

# Verify read-only mode in ktx.yaml
databases:
  warehouse:
    read_only: true  # Must be true
```

### Semantic layer search returns no results

**Solution:** Ensure semantic sources are defined and ingested:

```bash
# Check semantic-layer directory
ls -la semantic-layer/warehouse/

# Rebuild context
ktx ingest --force

# Test search
ktx sl "revenue"
```

### LLM provider errors

**Solution:** Verify API keys and provider configuration:

```bash
# Check environment
echo $ANTHROPIC_API_KEY
echo $OPENAI_API_KEY

# Test with explicit provider
export ANTHROPIC_API_KEY=your_key
ktx ingest
```

### Wiki pages not found in search

**Solution:** Ensure markdown files are in correct location:

```bash
# Check wiki structure
ls -la wiki/global/

# Add metadata to markdown files
cat > wiki/global/test.md << 'EOF'
# Test Page
This is searchable content.
EOF

# Rebuild context
ktx ingest --force
```

### Join graph resolution fails

**Solution:** Verify join definitions in semantic sources:

```yaml
# Ensure bidirectional joins are defined
# In orders.yaml:
joins:
  - name: user
    to: users
    type: left
    on:
      - left: user_id
        right: user_id

# In users.yaml:
joins:
  - name: orders
    to: orders
    type: left
    on:
      - left: user_id
        right: user_id
```

### Permission denied errors

**Solution:** Ensure database user has SELECT permissions:

```sql
-- PostgreSQL example
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analytics_user;
GRANT USAGE ON SCHEMA public TO analytics_user;
```

## Project Structure Reference

```
my-analytics-project/
├── ktx.yaml                              # Main configuration
├── semantic-layer/                       # Semantic layer definitions
│   └── warehouse/
│       ├── users.yaml
│       ├── orders.yaml
│       └── subscriptions.yaml
├── wiki/                                 # Knowledge base
│   ├── global/
│   │   ├── revenue-definition.md
│   │   ├── metric-catalog.md
│   │   └── data-dictionary.md
│   └── user/
│       └── alice/
│           └── analysis-notes.md
├── raw-sources/                          # Ingest artifacts
│   └── warehouse/
│       ├── metadata.json
│       └── reports/
└── .ktx/                                 # Local state (git-ignored)
    ├── state.db
    └── credentials.json
```

## Environment Variables Reference

```bash
# LLM Providers
export ANTHROPIC_API_KEY=your_anthropic_key
export OPENAI_API_KEY=your_openai_key
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/gcp-credentials.json

# Database Connections
export POSTGRES_USER=analytics_user
export POSTGRES_PASSWORD=secure_password
export SNOWFLAKE_USER=analyst
export SNOWFLAKE_PASSWORD=secure_password
export SNOWFLAKE_ACCOUNT=xy12345.us-east-1

# Integrations
export NOTION_TOKEN=secret_notion_token
export DBT_PROJECT_DIR=/path/to/dbt

# Project
export KTX_PROJECT_DIR=/path/to/analytics
```
