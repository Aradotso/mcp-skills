---
name: ktx-context-layer-data-agents
description: Build and query a context layer for AI data agents with ktx - auto-learning semantic layer, wiki, and MCP integration
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer
  - ingest database context with ktx
  - connect AI agent to warehouse using ktx
  - search ktx wiki and metrics
  - configure ktx MCP server
  - build ktx context from dbt
  - troubleshoot ktx agent integration
---

# ktx Context Layer for Data Agents

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is an executable context layer that teaches AI agents how to query data warehouses accurately. It automatically builds and maintains:

- **Semantic layer** with approved metric definitions, join graphs, and automatic fan/chasm trap resolution
- **Wiki** from company knowledge (dbt, Looker, Notion) with deduplication and contradiction detection
- **MCP server** exposing tools for agent execution via CLI or Model Context Protocol

Agents get one searchable surface instead of reinventing SQL on every prompt.

## Installation

```bash
# Install globally
npm install -g @kaelio/ktx

# Or use npx
npx @kaelio/ktx --help
```

**Requirements:**
- Node.js 18+
- SQL warehouse (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, or SQLite)
- LLM provider API key (Anthropic, Google Vertex AI, or AI Gateway)

## Quick Setup

```bash
# Interactive setup - creates ktx.yaml, configures providers, builds context
ktx setup

# Check project status
ktx status

# Build context from configured sources
ktx ingest

# Start MCP server for agent integration
ktx mcp start
```

Example `ktx status` output:
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
├── ktx.yaml                         # Main configuration
├── semantic-layer/
│   └── warehouse/                   # Per-connection YAML semantic sources
│       ├── customers.yaml
│       └── revenue.yaml
├── wiki/
│   ├── global/                      # Shared business context
│   │   └── refund-policy.md
│   └── user/<user-id>/              # User-scoped notes
├── raw-sources/
│   └── warehouse/                   # Ingest artifacts and reports
└── .ktx/                            # Local state, git-ignored
```

**Commit:** `ktx.yaml`, `semantic-layer/`, `wiki/`  
**Ignore:** `.ktx/`

## Configuration (ktx.yaml)

### Minimal Configuration

```yaml
version: 1
project:
  id: analytics-warehouse
  name: Analytics Project

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  apiKeyEnv: ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  apiKeyEnv: OPENAI_API_KEY

databases:
  warehouse:
    type: postgres
    host: db.example.com
    port: 5432
    database: analytics
    user: ktx_reader
    passwordEnv: WAREHOUSE_PASSWORD
    ssl: true

contextSources:
  - id: dbt_main
    type: dbt
    path: ./dbt/target/manifest.json
    database: warehouse
```

### Full Configuration with Multiple Sources

```yaml
version: 1
project:
  id: multi-source-analytics
  name: Multi-Source Analytics

llm:
  provider: vertex
  model: claude-sonnet-4-6
  region: us-central1
  projectId: my-gcp-project
  credentialsEnv: GOOGLE_APPLICATION_CREDENTIALS

embeddings:
  provider: openai
  model: text-embedding-3-small
  apiKeyEnv: OPENAI_API_KEY

databases:
  warehouse:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: KTX_USER
    passwordEnv: SNOWFLAKE_PASSWORD
    role: ANALYST

  clickhouse:
    type: clickhouse
    host: clickhouse.example.com
    port: 8123
    database: events
    user: readonly
    passwordEnv: CLICKHOUSE_PASSWORD

contextSources:
  - id: dbt_main
    type: dbt
    path: ./dbt/target/manifest.json
    database: warehouse
    enabled: true

  - id: looker_lookml
    type: lookml
    path: ./looker-models
    database: warehouse
    enabled: true

  - id: metabase_exports
    type: metabase
    path: ./metabase-export.json
    database: warehouse
    enabled: true

  - id: company_wiki
    type: notion
    apiKeyEnv: NOTION_API_KEY
    databaseId: a1b2c3d4e5f6
    enabled: true

ingestion:
  tableSampleSize: 1000
  enableJoinDiscovery: true
  enableColumnProfiling: true
```

## Core Commands

### Setup and Status

```bash
# Create or update project
ktx setup

# Check project health
ktx status

# Validate configuration
ktx config validate

# Show current config
ktx config show
```

### Context Building

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest with verbose output
ktx ingest --verbose

# Force re-ingest (ignores cache)
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer
ktx sl "revenue"
ktx sl "active users" --limit 10

# Search wiki
ktx wiki "refund policy"
ktx wiki "data retention" --connection warehouse

# Describe a specific semantic source
ktx describe semantic customers
ktx describe semantic revenue --connection warehouse
```

### MCP Server

```bash
# Start MCP server (required for agent integration)
ktx mcp start

# Start with custom project directory
ktx mcp start --project-dir /path/to/project

# Check if MCP is running
ktx status | grep "Agent integration"
```

## Agent Integration

### Claude Code / Codex

From your project directory, tell the agent:

```text
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install
and configure ktx in this project.
```

Or manually add to `claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\config.json` (Windows):

```json
{
  "mcpServers": {
    "ktx": {
      "command": "ktx",
      "args": ["mcp", "start", "--project-dir", "/absolute/path/to/project"]
    }
  }
}
```

### Cursor / OpenCode

Add to MCP settings:

```json
{
  "mcpServers": {
    "ktx": {
      "command": "npx",
      "args": ["@kaelio/ktx", "mcp", "start", "--project-dir", "/absolute/path/to/project"]
    }
  }
}
```

## TypeScript API Usage

```typescript
import { KtxClient } from '@kaelio/ktx';

// Initialize client
const ktx = new KtxClient({
  projectDir: '/path/to/project',
});

// Search semantic layer
const metrics = await ktx.searchSemanticLayer({
  query: 'revenue',
  limit: 5,
});

console.log(metrics);
// [
//   {
//     id: 'revenue',
//     type: 'metric',
//     sql: 'SUM(amount)',
//     description: 'Total revenue from orders',
//     connection: 'warehouse',
//   }
// ]

// Search wiki
const wikiPages = await ktx.searchWiki({
  query: 'refund policy',
  limit: 3,
});

// Get semantic source details
const customerMetric = await ktx.getSemanticSource({
  connection: 'warehouse',
  sourceId: 'customers',
});

// Execute query with context
const result = await ktx.query({
  connection: 'warehouse',
  sql: 'SELECT * FROM semantic.revenue WHERE date >= CURRENT_DATE - 7',
  useSemanticLayer: true,
});
```

## Semantic Layer Definition

### Creating a Metric (YAML)

`semantic-layer/warehouse/revenue.yaml`:

```yaml
type: metric
id: total_revenue
name: Total Revenue
description: Sum of all order amounts excluding refunds
sql: |
  SUM(CASE 
    WHEN status != 'refunded' 
    THEN amount 
    ELSE 0 
  END)
baseTable: orders
tags:
  - finance
  - core
filters:
  - dimension: status
    operator: not_in
    values: ['cancelled', 'fraud']
aggregation: sum
```

### Creating a Dimension

`semantic-layer/warehouse/customers.yaml`:

```yaml
type: dimension
id: customer_segment
name: Customer Segment
description: Customer lifecycle segment based on LTV
sql: |
  CASE
    WHEN lifetime_value > 10000 THEN 'enterprise'
    WHEN lifetime_value > 1000 THEN 'mid-market'
    ELSE 'smb'
  END
baseTable: customers
dataType: string
tags:
  - segmentation
```

### Join Definition

```yaml
type: join
id: orders_to_customers
from: orders
to: customers
relationship: many_to_one
sql: orders.customer_id = customers.id
requiredFilters: []
```

## Wiki Content

### Adding Business Context

`wiki/global/refund-policy.md`:

```markdown
# Refund Policy

## Definition

A refund is issued when:
- Customer requests within 30 days
- Product is defective
- Service was not delivered

## Metrics Impact

- `refunded_revenue`: Revenue from orders with status = 'refunded'
- `net_revenue`: Total revenue excluding refunds
- `refund_rate`: refunded_revenue / total_revenue

## Related Tables

- `orders.status`: Use 'refunded' for refund detection
- `refunds`: Detailed refund records with reason codes
```

### User-Scoped Notes

`wiki/user/alice@example.com/weekly-metrics.md`:

```markdown
# Weekly Metrics Review Notes

## Active Users Definition

Per discussion with Product (2024-05-15):
- Use `events.user_id` not `users.id`
- Filter to `event_type IN ('page_view', 'feature_used')`
- 7-day rolling window

Query stored in: `semantic-layer/warehouse/active_users.yaml`
```

## Common Patterns

### Setting Up a New Project

```bash
# 1. Initialize project
mkdir analytics-project && cd analytics-project
ktx setup

# 2. Configure in ktx.yaml (edit manually or via setup prompts)

# 3. Add dbt manifest as context source
cat >> ktx.yaml <<EOF
contextSources:
  - id: dbt_main
    type: dbt
    path: ../dbt/target/manifest.json
    database: warehouse
EOF

# 4. Build context
ktx ingest

# 5. Verify
ktx sl "customers"
ktx wiki "refund"

# 6. Start MCP for agents
ktx mcp start
```

### Connecting to Snowflake

```yaml
databases:
  warehouse:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: KTX_USER
    passwordEnv: SNOWFLAKE_PASSWORD
    role: ANALYST
    authenticator: snowflake  # or 'externalbrowser' for SSO
```

```bash
# Set password
export SNOWFLAKE_PASSWORD='your-password'

# Test connection
ktx ingest --connection warehouse
```

### Connecting to BigQuery

```yaml
databases:
  warehouse:
    type: bigquery
    projectId: my-gcp-project
    dataset: analytics
    credentialsEnv: GOOGLE_APPLICATION_CREDENTIALS
    location: US
```

```bash
# Set credentials path
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Test
ktx ingest --connection warehouse
```

### Using Local LLM via AI Gateway

```yaml
llm:
  provider: ai-gateway
  baseUrl: http://localhost:8080/v1
  model: llama-3-70b
  apiKeyEnv: AI_GATEWAY_API_KEY  # optional
```

### Ingesting Notion as Wiki Source

```yaml
contextSources:
  - id: company_wiki
    type: notion
    apiKeyEnv: NOTION_API_KEY
    databaseId: a1b2c3d4e5f6
    enabled: true
```

```bash
export NOTION_API_KEY='secret_...'
ktx ingest
```

## Troubleshooting

### "Project ready: no"

```bash
# Check what's missing
ktx status

# Common fixes:
# - Missing LLM API key
export ANTHROPIC_API_KEY='sk-ant-...'

# - Missing database password
export WAREHOUSE_PASSWORD='...'

# - Invalid ktx.yaml
ktx config validate
```

### "Agent integration ready: no"

```bash
# Check status output for MCP start command
ktx status

# Run the printed command, e.g.:
ktx mcp start --project-dir /home/user/analytics

# Verify MCP config in Claude Desktop:
# macOS: ~/Library/Application Support/Claude/claude_desktop_config.json
# Windows: %APPDATA%\Claude\config.json
```

### Database Connection Errors

```bash
# Test connection explicitly
ktx ingest --connection warehouse --verbose

# Common issues:
# - SSL required: Add ssl: true to database config
# - Firewall: Verify host/port accessibility
# - Credentials: Check passwordEnv variable is set
# - Permissions: Ensure user has SELECT grants on target schema
```

### Ingestion Fails with "No tables found"

```bash
# Check database config schema/dataset
cat ktx.yaml | grep -A 10 databases

# For Snowflake: Verify warehouse, database, schema, role
# For BigQuery: Verify projectId, dataset
# For Postgres: Verify database, schema (defaults to 'public')

# List accessible tables manually:
# PostgreSQL: SELECT tablename FROM pg_tables WHERE schemaname = 'public';
# Snowflake: SHOW TABLES IN SCHEMA analytics.public;
```

### Semantic Layer Not Resolving Joins

```bash
# Check join definitions
ls semantic-layer/warehouse/

# Ensure joins are defined:
# type: join
# relationship: many_to_one | one_to_many | one_to_one

# Run with verbose logging
ktx ingest --verbose

# Check for fan trap warnings in output
```

### MCP Tools Not Available in Agent

```bash
# 1. Ensure MCP server is running
ktx mcp start --project-dir /absolute/path

# 2. Restart agent client completely (not just reload)

# 3. Check MCP server logs
# Look for connection attempts and errors

# 4. Verify absolute path in MCP config (no ~, no relative paths)
```

### Slow Ingestion

```bash
# Reduce table sample size
# In ktx.yaml:
ingestion:
  tableSampleSize: 100  # default 1000
  enableJoinDiscovery: false  # if not needed
  enableColumnProfiling: false  # if not needed

# Ingest specific connection only
ktx ingest --connection warehouse
```

### Contradictions Flagged During Ingest

```bash
# Review flagged contradictions in raw-sources/<connection>/reports/
cat raw-sources/warehouse/reports/contradictions.json

# Example:
# {
#   "metric": "active_users",
#   "sources": ["dbt", "looker"],
#   "conflict": "Different SQL definitions"
# }

# Resolve by:
# 1. Choose canonical source
# 2. Update semantic-layer/<connection>/<metric>.yaml
# 3. Document decision in wiki/global/<metric>-definition.md
```

### Type Errors with TypeScript API

```typescript
// Ensure @kaelio/ktx is installed as dependency, not devDependency
// npm install @kaelio/ktx

// Import types explicitly
import type { SemanticSource, WikiPage } from '@kaelio/ktx';

const metric: SemanticSource = await ktx.getSemanticSource({
  connection: 'warehouse',
  sourceId: 'revenue',
});
```

## Advanced Usage

### Programmatic Project Setup

```typescript
import { KtxProject } from '@kaelio/ktx';

const project = await KtxProject.create({
  projectDir: '/path/to/project',
  config: {
    version: 1,
    project: {
      id: 'analytics',
      name: 'Analytics Project',
    },
    llm: {
      provider: 'anthropic',
      model: 'claude-sonnet-4-6',
      apiKeyEnv: 'ANTHROPIC_API_KEY',
    },
    embeddings: {
      provider: 'openai',
      model: 'text-embedding-3-small',
      apiKeyEnv: 'OPENAI_API_KEY',
    },
    databases: {
      warehouse: {
        type: 'postgres',
        host: 'localhost',
        port: 5432,
        database: 'analytics',
        user: 'ktx',
        passwordEnv: 'DB_PASSWORD',
      },
    },
    contextSources: [],
  },
});

await project.ingest();
```

### Custom Embedding Search

```typescript
const results = await ktx.searchSemanticLayer({
  query: 'monthly recurring revenue',
  limit: 5,
  threshold: 0.7,  // Similarity threshold (0-1)
  filters: {
    tags: ['finance'],
    connection: 'warehouse',
  },
});
```

### Exporting Context for External Use

```bash
# Export semantic layer as JSON
ktx export semantic --connection warehouse > semantic-layer.json

# Export wiki as markdown archive
ktx export wiki > wiki-export.tar.gz
```

## Resources

- **Documentation:** https://docs.kaelio.com/ktx/docs/
- **CLI Reference:** https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Slack Community:** https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues:** https://github.com/Kaelio/ktx/issues
- **Agent Quickstart:** https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
