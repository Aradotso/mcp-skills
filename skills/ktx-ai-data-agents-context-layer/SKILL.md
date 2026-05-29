---
name: ktx-ai-data-agents-context-layer
description: Executable context layer for data and analytics AI agents - queries data accurately through MCP, skills, and semantic layer
triggers:
  - set up ktx for data agent queries
  - configure ktx context layer for my warehouse
  - add ktx to enable AI agent data queries
  - install ktx semantic layer for Claude
  - connect ktx to my database for agent access
  - build ktx context from dbt and warehouse
  - query my data warehouse through ktx
  - configure ktx MCP server for agents
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What ktx Does

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It:

- **Ingests company knowledge** from wikis, dbt, Looker, Metabase, and Notion
- **Maps your data stack** by sampling tables, detecting joinable columns, and capturing metadata
- **Builds a semantic layer** with approved metric definitions and automatic join resolution
- **Serves agents via MCP** with searchable context across wiki and semantic entities
- **Runs locally** with your own LLM API keys (no hosted service, read-only by design)

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, and SQLite.

## Installation

### Global CLI Installation

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

## Setup and Configuration

### Initial Setup

Run the interactive setup wizard:

```bash
ktx setup
```

This will:
1. Create or resume a local ktx project
2. Configure LLM and embedding providers
3. Set up database connections
4. Configure context sources (dbt, Looker, etc.)
5. Build initial context
6. Install agent integration (MCP)

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

### Manual Configuration

Edit `ktx.yaml` in your project directory:

```yaml
# ktx.yaml
version: 1
project:
  name: my-analytics-project
  
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  apiKey: ${ANTHROPIC_API_KEY}

embeddings:
  provider: openai
  model: text-embedding-3-small
  apiKey: ${OPENAI_API_KEY}

databases:
  warehouse:
    type: postgres
    host: ${DB_HOST}
    port: 5432
    database: analytics
    user: ${DB_USER}
    password: ${DB_PASSWORD}
    ssl: true

contextSources:
  dbt_main:
    type: dbt
    manifestPath: ./target/manifest.json
    catalogPath: ./target/catalog.json
```

### Environment Variables

Create `.env` in your project directory:

```bash
# .env (add to .gitignore)
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
DB_HOST=warehouse.company.com
DB_USER=readonly_user
DB_PASSWORD=secure_password
```

## Project Structure

```
my-project/
├── ktx.yaml                         # Project configuration (commit)
├── semantic-layer/                  # Generated YAML semantic sources (commit)
│   └── warehouse/
│       ├── customers.yaml
│       └── orders.yaml
├── wiki/                            # Business context (commit)
│   ├── global/
│   │   ├── metrics-definitions.md
│   │   └── data-dictionary.md
│   └── user/
│       └── john-doe/
│           └── notes.md
├── raw-sources/                     # Ingest artifacts (commit)
│   └── warehouse/
│       └── scan-report.json
├── .ktx/                            # Local state (gitignore)
│   ├── state.db
│   └── secrets.enc
└── .env                             # Environment variables (gitignore)
```

## Core Commands

### Building Context

Ingest and build context from all configured sources:

```bash
ktx ingest
```

Ingest specific connection only:

```bash
ktx ingest --connection warehouse
```

Force full re-ingest:

```bash
ktx ingest --force
```

### Searching Context

Search semantic layer (metrics, dimensions, tables):

```bash
ktx sl "revenue"
ktx sl "customer lifetime value"
ktx sl "monthly active users"
```

Search wiki content:

```bash
ktx wiki "refund policy"
ktx wiki "data retention"
ktx wiki "how to calculate churn"
```

### MCP Server

Start the MCP server for agent clients:

```bash
ktx mcp start
```

Start with specific project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

Stop the MCP server:

```bash
ktx mcp stop
```

### Managing Connections

List configured database connections:

```bash
ktx db list
```

Test a connection:

```bash
ktx db test warehouse
```

Add a new database connection:

```bash
ktx db add
```

### Managing Context Sources

List configured context sources (dbt, Looker, etc.):

```bash
ktx sources list
```

Add a new context source:

```bash
ktx sources add
```

## Agent Integration

### Claude Code / Codex

From your project directory, tell the agent:

```text
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install and configure ktx in this project.
```

Or manually configure MCP in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "ktx": {
      "command": "npx",
      "args": [
        "-y",
        "@kaelio/ktx",
        "mcp",
        "start",
        "--project-dir",
        "/absolute/path/to/project"
      ]
    }
  }
}
```

### Using ktx in Agent Workflows

Once configured, agents can:

```typescript
// Example agent interaction
// User: "What was our revenue last quarter?"

// Agent uses ktx MCP tools:
// 1. Search semantic layer for "revenue" metric
// 2. Get metric definition with approved SQL
// 3. Execute query against warehouse
// 4. Return results with context
```

## TypeScript/JavaScript Usage

### Programmatic API

```typescript
import { KtxContext } from '@kaelio/ktx';

// Initialize context
const ctx = await KtxContext.load({
  projectDir: process.cwd(),
});

// Search semantic layer
const metrics = await ctx.semanticLayer.search('revenue', {
  limit: 5,
  types: ['metric'],
});

// Search wiki
const wikiPages = await ctx.wiki.search('refund policy', {
  limit: 10,
});

// Get metric definition
const metric = await ctx.semanticLayer.getMetric('monthly_recurring_revenue');
console.log(metric.sql);
console.log(metric.dimensions);
console.log(metric.measures);
```

### Custom Integration

```typescript
import { createMCPServer } from '@kaelio/ktx/mcp';

// Create custom MCP server
const server = await createMCPServer({
  projectDir: '/path/to/project',
  port: 3000,
});

await server.start();

// Register custom tools
server.registerTool({
  name: 'custom_query',
  description: 'Execute a custom query',
  inputSchema: {
    type: 'object',
    properties: {
      sql: { type: 'string' },
    },
    required: ['sql'],
  },
  handler: async ({ sql }) => {
    const result = await ctx.db.query('warehouse', sql);
    return result;
  },
});
```

## Database Connectors

### PostgreSQL

```yaml
databases:
  warehouse:
    type: postgres
    host: ${POSTGRES_HOST}
    port: 5432
    database: ${POSTGRES_DB}
    user: ${POSTGRES_USER}
    password: ${POSTGRES_PASSWORD}
    ssl: true
    sslMode: require
```

### Snowflake

```yaml
databases:
  snowflake:
    type: snowflake
    account: ${SNOWFLAKE_ACCOUNT}
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}
```

### BigQuery

```yaml
databases:
  bigquery:
    type: bigquery
    projectId: ${GCP_PROJECT_ID}
    datasetId: analytics
    keyFilePath: ${GOOGLE_APPLICATION_CREDENTIALS}
```

### ClickHouse

```yaml
databases:
  clickhouse:
    type: clickhouse
    host: ${CLICKHOUSE_HOST}
    port: 8123
    database: default
    user: ${CLICKHOUSE_USER}
    password: ${CLICKHOUSE_PASSWORD}
```

## Context Source Integrations

### dbt

```yaml
contextSources:
  dbt_prod:
    type: dbt
    manifestPath: ./target/manifest.json
    catalogPath: ./target/catalog.json
    runResultsPath: ./target/run_results.json
```

### LookML (Looker)

```yaml
contextSources:
  looker:
    type: lookml
    projectPath: ./looker-project
    modelsDir: models
    viewsDir: views
```

### Metabase

```yaml
contextSources:
  metabase:
    type: metabase
    url: ${METABASE_URL}
    apiKey: ${METABASE_API_KEY}
```

### Notion

```yaml
contextSources:
  notion_wiki:
    type: notion
    apiKey: ${NOTION_API_KEY}
    databaseId: ${NOTION_DATABASE_ID}
```

## Semantic Layer YAML Format

ktx generates YAML files in `semantic-layer/` that define metrics, dimensions, and entities:

```yaml
# semantic-layer/warehouse/revenue.yaml
version: 1
type: metric

metric:
  name: monthly_recurring_revenue
  label: Monthly Recurring Revenue
  description: Sum of all active subscription revenue in a month
  
  sql: |
    SELECT
      DATE_TRUNC('month', subscription_date) AS month,
      SUM(amount) AS mrr
    FROM {{ ref('subscriptions') }}
    WHERE status = 'active'
    GROUP BY 1
  
  type: simple
  measure: sum
  
  dimensions:
    - name: month
      type: time
      timeGrains: [day, month, quarter, year]
    
    - name: plan_type
      type: categorical
      
  filters:
    - field: status
      operator: equals
      value: active
```

## Common Patterns

### Adding Business Knowledge to Wiki

Create markdown files in `wiki/global/`:

```markdown
<!-- wiki/global/revenue-definitions.md -->

# Revenue Definitions

## Monthly Recurring Revenue (MRR)

MRR represents the normalized monthly revenue from active subscriptions.

**Formula**: Sum of all active subscription amounts normalized to monthly

**Filters**:
- Include only `status = 'active'`
- Exclude trial subscriptions
- Include annual subscriptions divided by 12

## Annual Recurring Revenue (ARR)

ARR = MRR × 12

**Use cases**:
- Board reporting
- Valuation discussions
- Long-term forecasting
```

### Querying Data Through Agent

```text
User: What was our MRR growth last quarter?

Agent workflow:
1. ktx sl "monthly recurring revenue"  # Find metric definition
2. Get approved SQL from semantic layer
3. Execute query against warehouse
4. Calculate quarter-over-quarter growth
5. Return results with context
```

### Creating Custom Metrics

```typescript
import { MetricBuilder } from '@kaelio/ktx/semantic-layer';

const customMetric = new MetricBuilder()
  .name('customer_ltv')
  .label('Customer Lifetime Value')
  .description('Average revenue per customer over their lifetime')
  .type('derived')
  .sql(`
    SELECT
      customer_id,
      SUM(order_total) / COUNT(DISTINCT customer_id) AS ltv
    FROM {{ ref('orders') }}
    GROUP BY 1
  `)
  .addDimension('customer_segment', 'categorical')
  .addDimension('cohort_month', 'time')
  .build();

await ctx.semanticLayer.addMetric(customMetric);
```

## Troubleshooting

### MCP Server Not Starting

Check project status:

```bash
ktx status
```

If output shows `ktx mcp start --project-dir ...`, run the exact command before opening your agent client.

Verify MCP configuration:

```bash
# Check Claude Desktop config
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Context Build Failing

Check database connectivity:

```bash
ktx db test warehouse
```

View detailed logs:

```bash
ktx ingest --verbose
```

Force clean rebuild:

```bash
rm -rf .ktx/state.db raw-sources/
ktx ingest --force
```

### LLM Provider Issues

Verify API keys are set:

```bash
echo $ANTHROPIC_API_KEY
echo $OPENAI_API_KEY
```

Test LLM configuration:

```bash
ktx setup --reconfigure-llm
```

### Search Returning No Results

Rebuild vector embeddings:

```bash
ktx ingest --rebuild-embeddings
```

Check what was indexed:

```bash
ktx sl "*" --limit 100
ktx wiki "*" --limit 100
```

### Database Permission Errors

Ensure read-only user has necessary grants:

```sql
-- PostgreSQL example
GRANT USAGE ON SCHEMA public TO readonly_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;
GRANT SELECT ON information_schema.tables TO readonly_user;
GRANT SELECT ON information_schema.columns TO readonly_user;
```

### Project Not Found

Explicitly set project directory:

```bash
export KTX_PROJECT_DIR=/path/to/project
ktx status
```

Or use flag:

```bash
ktx ingest --project-dir /path/to/project
```

## Advanced Configuration

### Custom LLM Providers

```yaml
llm:
  provider: vertex-ai
  model: claude-3-5-sonnet-v2@20241022
  project: ${GCP_PROJECT_ID}
  location: us-central1
```

### Multiple Database Connections

```yaml
databases:
  production:
    type: postgres
    host: ${PROD_DB_HOST}
    # ... production config
    
  staging:
    type: postgres
    host: ${STAGING_DB_HOST}
    # ... staging config
    
  clickhouse_events:
    type: clickhouse
    host: ${CLICKHOUSE_HOST}
    # ... events config
```

### Excluding Tables from Ingestion

```yaml
databases:
  warehouse:
    type: postgres
    # ... connection config
    excludePatterns:
      - "tmp_*"
      - "_airbyte_*"
      - "test_*"
```

### Custom Embedding Models

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-large
  dimensions: 3072
  apiKey: ${OPENAI_API_KEY}
```

## Telemetry

ktx collects anonymous usage telemetry from interactive CLI runs. No file paths, hostnames, SQL, schema names, or error messages are recorded.

Opt out:

```bash
ktx telemetry disable
```

Re-enable:

```bash
ktx telemetry enable
```

Check status:

```bash
ktx telemetry status
```

## Best Practices

1. **Commit semantic layer and wiki** to version control
2. **Keep `.ktx/` local** — add to `.gitignore`
3. **Use read-only database users** for all connections
4. **Document metrics in wiki/** for agent context
5. **Run `ktx ingest`** regularly to keep context fresh
6. **Test queries** before deploying new semantic definitions
7. **Use environment variables** for all secrets
8. **Review contradiction reports** after ingestion

## Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
