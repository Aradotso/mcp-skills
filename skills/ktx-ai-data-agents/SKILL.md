---
name: ktx-ai-data-agents
description: Configure and use ktx, the executable context layer for data and analytics agents with MCP, semantic layer, and warehouse integration
triggers:
  - "set up ktx for data analytics"
  - "configure ktx with my warehouse"
  - "create a semantic layer with ktx"
  - "integrate ktx with Claude Code"
  - "query my warehouse using ktx"
  - "build context for data agents"
  - "search ktx semantic layer"
  - "configure ktx MCP server"
---

# ktx AI Data Agents Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What ktx Does

ktx is an executable context layer that teaches AI agents how to query your data warehouse accurately. It:

- **Automatically builds warehouse context** by sampling tables, capturing metadata, and detecting joinable columns
- **Ingests company knowledge** from dbt, Looker, Metabase, Notion, and team wikis
- **Creates a semantic layer** with approved metric definitions and automatic join resolution
- **Serves agents via MCP** with combined full-text and semantic search across all context
- **Maintains itself** by flagging contradictions and organizing duplicates

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, and SQLite.

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
```

## Initial Setup

### Interactive Setup

```bash
ktx setup
```

This command:
1. Creates or resumes a local ktx project
2. Configures LLM and embedding providers
3. Sets up database connections
4. Configures context sources (dbt, Looker, etc.)
5. Builds initial context
6. Installs agent integration

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
version: 1
project:
  name: my-analytics
  
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  
embeddings:
  provider: openai
  model: text-embedding-3-small
  
connections:
  warehouse:
    type: postgres
    host: ${POSTGRES_HOST}
    port: 5432
    database: ${POSTGRES_DB}
    user: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}
    schema: public
    
sources:
  dbt_main:
    type: dbt
    profiles_dir: ~/.dbt
    project_dir: ./dbt
    target: prod
```

### Environment Variables

```bash
# LLM Configuration
export ANTHROPIC_API_KEY=your_api_key_here
export OPENAI_API_KEY=your_api_key_here

# Database Connections
export POSTGRES_HOST=localhost
export POSTGRES_DB=analytics
export POSTGRES_USER=readonly_user
export POSTGRES_PASSWORD=secure_password

# Project Directory (optional)
export KTX_PROJECT_DIR=/path/to/project
```

## Key Commands

### Build Context

Ingest and build context from all configured sources:

```bash
ktx ingest
```

For a specific connection:

```bash
ktx ingest --connection warehouse
```

### Search Semantic Layer

```bash
ktx sl "revenue"
ktx sl "customer lifetime value"
ktx sl "monthly active users"
```

### Search Wiki

```bash
ktx wiki "refund policy"
ktx wiki "customer segmentation"
```

### MCP Server

Start the MCP server for agent clients:

```bash
ktx mcp start
```

With custom project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

### Project Management

Initialize a new project:

```bash
ktx init
```

Validate configuration:

```bash
ktx validate
```

## Database Connections

### PostgreSQL

```yaml
connections:
  postgres_warehouse:
    type: postgres
    host: ${POSTGRES_HOST}
    port: 5432
    database: analytics
    user: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}
    schema: public
    ssl: true
```

### Snowflake

```yaml
connections:
  snowflake_prod:
    type: snowflake
    account: ${SNOWFLAKE_ACCOUNT}
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
```

### BigQuery

```yaml
connections:
  bigquery_prod:
    type: bigquery
    project_id: ${GCP_PROJECT_ID}
    dataset: analytics
    credentials_file: ${GOOGLE_APPLICATION_CREDENTIALS}
```

### SQLite (for testing)

```yaml
connections:
  local_db:
    type: sqlite
    path: ./data/local.db
```

## Context Sources

### dbt Integration

```yaml
sources:
  dbt_main:
    type: dbt
    profiles_dir: ~/.dbt
    project_dir: ./dbt
    target: prod
```

ktx will:
- Ingest model definitions
- Extract metric logic
- Map dependencies
- Preserve documentation

### Looker/LookML Integration

```yaml
sources:
  looker_prod:
    type: lookml
    project_dir: ./looker
    models:
      - analytics
      - marketing
```

### Metabase Integration

```yaml
sources:
  metabase_prod:
    type: metabase
    url: https://metabase.company.com
    api_key: ${METABASE_API_KEY}
```

### Notion Integration

```yaml
sources:
  notion_wiki:
    type: notion
    api_key: ${NOTION_API_KEY}
    database_id: ${NOTION_DATABASE_ID}
```

## Semantic Layer

### Defining Metrics

Create files in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/revenue.yaml
name: revenue
description: Total revenue across all orders
type: metric
sql: |
  SELECT 
    DATE_TRUNC('month', order_date) as period,
    SUM(order_total) as revenue
  FROM orders
  WHERE order_status = 'completed'
  GROUP BY period
dimensions:
  - period
measures:
  - revenue
tags:
  - finance
  - core-metric
```

### Defining Dimensions

```yaml
# semantic-layer/warehouse/customer.yaml
name: customer
description: Customer dimension
type: dimension
table: customers
primary_key: customer_id
columns:
  customer_id:
    type: string
    description: Unique customer identifier
  customer_segment:
    type: string
    description: Customer segment (enterprise, smb, individual)
  signup_date:
    type: date
    description: Date customer signed up
```

### Join Definitions

```yaml
# semantic-layer/warehouse/joins.yaml
joins:
  - left: orders
    right: customers
    on: orders.customer_id = customers.customer_id
    type: left
    
  - left: orders
    right: products
    on: orders.product_id = products.product_id
    type: left
```

## Wiki Management

### Creating Wiki Pages

```bash
# Global wiki (shared across team)
echo "# Refund Policy

Customers can request refunds within 30 days.
Approval requires manager review for amounts > $100." > wiki/global/refund-policy.md
```

### User-Scoped Notes

```bash
# User-specific notes
echo "# Q4 Analysis Notes

Focus on customer segments with >10% churn." > wiki/user/myuser/q4-notes.md
```

### Organizing Content

ktx automatically:
- Removes duplicate content across sources
- Flags contradictions for human review
- Organizes content by topic
- Maintains metadata for search

## Agent Integration

### Claude Code Integration

ktx installs automatically during setup. Verify with:

```bash
ktx status
```

Look for: `Agent integration ready: yes (codex:project)`

### Manual MCP Configuration

If needed, add to your MCP client config (e.g., `~/Library/Application Support/Claude/claude_desktop_config.json`):

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

### Using ktx in Agent Prompts

Example prompts:

```text
"Use ktx to search for revenue metrics"

"Search the ktx wiki for our refund policy"

"Query the warehouse using ktx semantic layer for monthly active users"

"Show me all ktx metrics related to customer retention"
```

## Common Patterns

### Setting Up a New Analytics Project

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Navigate to project
cd ~/analytics-project

# 3. Run interactive setup
ktx setup

# 4. Verify configuration
ktx status

# 5. Build context
ktx ingest

# 6. Search to verify
ktx sl "revenue"
```

### Adding a New Database Connection

```bash
# 1. Edit ktx.yaml, add connection
# 2. Set environment variables
export NEW_DB_PASSWORD=secure_password

# 3. Resume setup to configure
ktx setup

# 4. Ingest new connection
ktx ingest --connection new_db

# 5. Verify
ktx status
```

### Updating Context After Schema Changes

```bash
# Full re-ingestion
ktx ingest

# Or specific connection
ktx ingest --connection warehouse

# Verify updated context
ktx sl "your_new_metric"
```

### Creating a Standard Metric

```yaml
# semantic-layer/warehouse/arr.yaml
name: annual_recurring_revenue
description: ARR calculated from active subscriptions
type: metric
sql: |
  SELECT 
    DATE_TRUNC('month', period_start) as period,
    SUM(mrr * 12) as arr
  FROM subscriptions
  WHERE status = 'active'
  GROUP BY period
dimensions:
  - period
measures:
  - arr
tags:
  - finance
  - saas
owner: finance-team
```

## Troubleshooting

### Connection Issues

```bash
# Test connection manually
ktx validate --connection warehouse

# Check environment variables
echo $POSTGRES_HOST
echo $POSTGRES_PASSWORD

# Review logs
ktx status --verbose
```

### LLM Configuration Issues

```bash
# Verify API key is set
echo $ANTHROPIC_API_KEY

# Test with different model
ktx setup
# Select different model during interactive setup

# Check provider status
ktx status
```

### MCP Server Not Starting

```bash
# Check if already running
ps aux | grep "ktx mcp"

# Kill existing process
pkill -f "ktx mcp"

# Start with verbose logging
ktx mcp start --verbose

# Verify project directory
ktx mcp start --project-dir $(pwd)
```

### Context Not Building

```bash
# Check source configuration
ktx status

# Validate ktx.yaml
ktx validate

# Re-run setup
ktx setup

# Force fresh ingestion
rm -rf raw-sources/
ktx ingest
```

### Search Not Returning Results

```bash
# Verify context was built
ktx status

# Check if files exist
ls -la semantic-layer/
ls -la wiki/

# Re-build embeddings
ktx ingest

# Try different search terms
ktx sl "metric"
ktx wiki "policy"
```

## Project Layout

```text
my-project/
├── ktx.yaml                         # Main configuration
├── semantic-layer/                  # Semantic layer definitions
│   └── warehouse/
│       ├── revenue.yaml
│       ├── customers.yaml
│       └── joins.yaml
├── wiki/                           # Knowledge base
│   ├── global/                     # Shared documentation
│   │   └── refund-policy.md
│   └── user/                       # User-specific notes
│       └── myuser/
│           └── analysis-notes.md
├── raw-sources/                    # Ingestion artifacts
│   └── warehouse/
│       ├── tables.json
│       └── metadata.json
└── .ktx/                          # Local state (gitignored)
    ├── secrets.json
    └── state.db
```

### What to Commit

**Do commit:**
- `ktx.yaml`
- `semantic-layer/`
- `wiki/global/`

**Don't commit:**
- `.ktx/` (contains secrets and local state)
- `raw-sources/` (regenerated on ingestion)
- `wiki/user/` (personal notes)

## Advanced Configuration

### Custom Project Directory

```bash
# Via environment variable
export KTX_PROJECT_DIR=/path/to/project
ktx status

# Via flag
ktx status --project-dir /path/to/project
```

### Multiple Environments

```yaml
# ktx.yaml for staging
connections:
  warehouse:
    type: postgres
    host: ${STAGING_POSTGRES_HOST}
    database: analytics_staging
    user: ${STAGING_POSTGRES_USER}
    password: ${STAGING_POSTGRES_PASSWORD}
```

```bash
# Separate project directories
ktx setup --project-dir ~/analytics-staging
ktx setup --project-dir ~/analytics-prod
```

### Custom Embedding Models

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-large  # Higher quality
  dimensions: 1536
```

### Read-Only Database Users

Create dedicated read-only users:

```sql
-- PostgreSQL
CREATE ROLE ktx_readonly WITH LOGIN PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE analytics TO ktx_readonly;
GRANT USAGE ON SCHEMA public TO ktx_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ktx_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO ktx_readonly;
```

## Performance Tips

- Use `--connection` flag to ingest incrementally
- Keep semantic layer files focused (one metric per file)
- Use tags to organize metrics and dimensions
- Regularly clean up duplicate wiki content
- Configure appropriate sampling rates for large tables

## Security Best Practices

1. **Never commit secrets** — use environment variables
2. **Use read-only database users** — ktx never writes to your warehouse
3. **Restrict API key permissions** — use minimum required scopes
4. **Keep `.ktx/` local** — add to `.gitignore`
5. **Rotate credentials regularly** — update environment variables and re-run setup
