---
name: ktx-ai-data-agents
description: Context layer for AI data agents that teaches them to query warehouses accurately with approved metrics, wiki knowledge, and semantic layers via MCP
triggers:
  - set up ktx for data analysis
  - configure ktx semantic layer
  - add ktx to my analytics project
  - help AI agents query my data warehouse
  - integrate ktx with Claude Code
  - build context for database with ktx
  - create ktx metrics and wiki
  - connect ktx to my data sources
---

# ktx AI Data Agents

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

ktx is an executable context layer that teaches AI agents (Claude Code, Codex, Cursor, OpenCode) how to query your data warehouse accurately. It automatically builds context from your data stack, wiki content, dbt projects, and semantic layers, then exposes this knowledge through MCP tools and CLI commands.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds a semantic layer**: Combines raw tables and metrics with automatic join resolution (chasm/fan trap handling)
- **Serves agents**: Exposes CLI and MCP tools with full-text + semantic search across wiki and semantic-layer entities

## Installation

```bash
npm install -g @kaelio/ktx
```

Or add to a specific project:

```bash
npm install --save-dev @kaelio/ktx
```

## Initial Setup

Run the interactive setup wizard:

```bash
ktx setup
```

This will:
1. Create or resume a local ktx project
2. Configure LLM providers (Anthropic API, Vertex AI, AI Gateway, or Claude Code SDK)
3. Configure embedding providers (OpenAI, Vertex AI, Voyage AI)
4. Add database connections (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite)
5. Configure context sources (dbt, MetricFlow, LookML, Looker, Metabase, Notion)
6. Build initial context
7. Install agent integration

Check project status:

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

ktx projects are configured via `ktx.yaml`:

```yaml
name: my-analytics
version: 1.0.0

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key from environment: ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key from environment: OPENAI_API_KEY

connections:
  warehouse:
    type: postgresql
    host: localhost
    port: 5432
    database: analytics
    # Credentials from environment: PGUSER, PGPASSWORD
    read_only: true
    schemas:
      - public
      - analytics

context_sources:
  dbt_main:
    type: dbt
    path: ./dbt_project
    connection: warehouse
  
  company_wiki:
    type: notion
    # Token from environment: NOTION_TOKEN
    database_id: abc123
```

## Building Context

Ingest all configured sources:

```bash
ktx ingest
```

Ingest a specific connection or source:

```bash
ktx ingest --connection warehouse
ktx ingest --source dbt_main
```

Force re-ingestion:

```bash
ktx ingest --force
```

## Semantic Layer

ktx generates YAML semantic sources in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/revenue_metrics.yaml
version: 1
kind: semantic_source
metadata:
  name: revenue_metrics
  description: Monthly revenue metrics by customer segment
  connection: warehouse
  tags:
    - revenue
    - metrics
    - finance

sql: |
  SELECT
    date_trunc('month', order_date) as month,
    customer_segment,
    SUM(amount) as total_revenue,
    COUNT(DISTINCT customer_id) as unique_customers
  FROM orders
  WHERE status = 'completed'
  GROUP BY 1, 2

columns:
  - name: month
    type: timestamp
    description: Start of the month
    primary_key: true
  
  - name: customer_segment
    type: string
    description: Customer segment (Enterprise, SMB, Self-serve)
    primary_key: true
  
  - name: total_revenue
    type: numeric
    description: Total revenue in USD for the month and segment
    aggregation: sum
  
  - name: unique_customers
    type: integer
    description: Count of distinct customers who purchased
    aggregation: sum

joins:
  - to: customer_segments
    on: customer_segment = segment_name
    type: left
```

Create custom semantic sources manually or let ktx generate them from introspection:

```bash
# Generate from database introspection
ktx ingest --connection warehouse --generate-semantic-layer
```

## Wiki Pages

Add business context in `wiki/global/` (shared) or `wiki/user/<user-id>/` (personal):

```markdown
<!-- wiki/global/revenue-policy.md -->
# Revenue Recognition Policy

## Completed Orders
Revenue is recognized when order status = 'completed' and payment is received.

## Refunds
- Full refunds within 30 days: subtract from period revenue
- Partial refunds: adjust amount but keep order in completed status
- Canceled orders: status = 'canceled', never counted in revenue

## Customer Segments
- **Enterprise**: Annual contract > $100k
- **SMB**: Annual contract $10k-$100k  
- **Self-serve**: Pay-as-you-go, no contract
```

ktx automatically indexes wiki content and makes it searchable.

## CLI Commands

### Search semantic layer

```bash
ktx sl "revenue"
ktx sl "monthly active users" --limit 5
```

### Search wiki

```bash
ktx wiki "refund policy"
ktx wiki "how we define churn"
```

### Query execution

```bash
# Execute SQL using context
ktx query "show me revenue by segment last month"

# With specific connection
ktx query --connection warehouse "SELECT * FROM orders LIMIT 10"
```

### MCP Server

Start the MCP server for agent integration:

```bash
ktx mcp start
```

With custom project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

The MCP server exposes tools:
- `ktx_search_semantic_layer`: Search metrics, sources, and columns
- `ktx_search_wiki`: Search business context
- `ktx_query`: Execute SQL with context awareness
- `ktx_get_joins`: Get available joins for a source
- `ktx_resolve_metric`: Get canonical SQL for a metric

## Agent Integration

### Claude Code

ktx auto-configures Claude Code during setup. Ensure `ktx mcp start` is running, then use natural language:

```text
User: What was our revenue last month?
```

Claude will use ktx tools to:
1. Search semantic layer for revenue metrics
2. Search wiki for revenue recognition policy
3. Generate and execute accurate SQL
4. Return results with business context

### Codex (Cursor)

Add to Codex project skills:

```bash
npx skills add Kaelio/ktx --skill ktx
```

Or manually add to `.cursor/skills.json`:

```json
{
  "skills": [
    {
      "name": "ktx",
      "path": "Kaelio/ktx"
    }
  ]
}
```

Then start the MCP server:

```bash
ktx mcp start --project-dir .
```

### OpenCode / Other Agents

Configure MCP in agent settings:

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

## TypeScript API

For programmatic access:

```typescript
import { KtxContext } from '@kaelio/ktx';

// Initialize context
const ctx = await KtxContext.load({
  projectDir: process.cwd(),
  llmProvider: 'anthropic',
  llmModel: 'claude-sonnet-4-6'
});

// Search semantic layer
const metrics = await ctx.searchSemanticLayer('revenue', { limit: 5 });
console.log(metrics);

// Search wiki
const wikiPages = await ctx.searchWiki('refund policy');
console.log(wikiPages);

// Query with context
const result = await ctx.query('SELECT * FROM revenue_metrics WHERE month >= NOW() - INTERVAL \'1 month\'');
console.log(result.rows);

// Get joins for a source
const joins = await ctx.getJoins('revenue_metrics');
console.log(joins);
```

## Configuration Patterns

### Multi-environment setup

```yaml
connections:
  prod_warehouse:
    type: postgresql
    host: prod.db.company.com
    database: analytics
    schemas: [public, analytics]
  
  dev_warehouse:
    type: postgresql
    host: localhost
    database: analytics_dev
    schemas: [public, analytics]

context_sources:
  prod_dbt:
    type: dbt
    path: ./dbt_project
    connection: prod_warehouse
    profiles_dir: ./dbt_profiles
    target: prod
  
  dev_dbt:
    type: dbt
    path: ./dbt_project
    connection: dev_warehouse
    profiles_dir: ./dbt_profiles
    target: dev
```

### Multiple data sources

```yaml
connections:
  warehouse:
    type: snowflake
    account: xy12345.us-east-1
    database: ANALYTICS
    warehouse: COMPUTE_WH
    role: ANALYST
  
  operational_db:
    type: postgresql
    host: prod.db.company.com
    database: app_production
    schemas: [public]

context_sources:
  dbt_warehouse:
    type: dbt
    connection: warehouse
    path: ./analytics_dbt
  
  metabase_ops:
    type: metabase
    connection: operational_db
    url: https://metabase.company.com
    # Token from: METABASE_API_TOKEN
```

## Environment Variables

ktx reads credentials from environment variables:

```bash
# LLM Providers
export ANTHROPIC_API_KEY=sk-ant-...
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Embedding Providers
export OPENAI_API_KEY=sk-...
export VOYAGE_API_KEY=pa-...

# Database Connections
export PGUSER=readonly_user
export PGPASSWORD=secure_password
export SNOWFLAKE_USER=analyst
export SNOWFLAKE_PASSWORD=secure_password

# Context Sources
export NOTION_TOKEN=secret_...
export METABASE_API_TOKEN=mb-...
export LOOKER_CLIENT_ID=abc123
export LOOKER_CLIENT_SECRET=xyz789
```

Set in `.env` file (git-ignored):

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
PGUSER=readonly
PGPASSWORD=secure
```

ktx automatically loads `.env` files.

## Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/
│   └── warehouse/                   # Per-connection semantic sources
│       ├── revenue_metrics.yaml
│       └── customer_segments.yaml
├── wiki/
│   ├── global/                      # Shared business context
│   │   ├── revenue-policy.md
│   │   └── customer-segments.md
│   └── user/
│       └── alice/                   # User-scoped notes
├── raw-sources/
│   └── warehouse/                   # Ingest artifacts
│       ├── introspection.json
│       └── sample-data.parquet
├── .ktx/                            # Local state (git-ignored)
│   ├── state.json
│   └── secrets.enc
└── .env                             # Environment variables (git-ignored)
```

**Git recommendations:**
- Commit: `ktx.yaml`, `semantic-layer/`, `wiki/global/`
- Ignore: `.ktx/`, `.env`, `wiki/user/`, `raw-sources/`

## Troubleshooting

### Status shows project not ready

```bash
ktx status
# Project ready: no
```

Run setup again:

```bash
ktx setup
```

### LLM provider errors

Verify API key is set:

```bash
echo $ANTHROPIC_API_KEY
# Should print: sk-ant-...
```

Test LLM connection:

```bash
ktx setup --reconfigure-llm
```

### Database connection fails

Test connection directly:

```bash
ktx ingest --connection warehouse --dry-run
```

Verify credentials:

```bash
echo $PGUSER
echo $PGPASSWORD
```

### Context not updating

Force re-ingestion:

```bash
ktx ingest --force
```

Clear and rebuild:

```bash
rm -rf .ktx/state.json
ktx ingest
```

### MCP server won't start

Check if already running:

```bash
ps aux | grep "ktx mcp"
```

Kill existing process:

```bash
pkill -f "ktx mcp"
ktx mcp start
```

### Search returns no results

Check context was built:

```bash
ktx status
# ktx context built: yes
```

Rebuild indexes:

```bash
ktx ingest --rebuild-index
```

### Semantic layer queries fail

Validate semantic source YAML:

```bash
ktx validate semantic-layer/warehouse/revenue_metrics.yaml
```

Check join graph:

```bash
ktx graph --connection warehouse
```

## Advanced Usage

### Custom LLM configuration

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  temperature: 0.0
  max_tokens: 4096
  top_p: 1.0
```

### Embedding configuration

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-small
  dimensions: 1536
  batch_size: 100
```

### Connection pooling

```yaml
connections:
  warehouse:
    type: postgresql
    pool_size: 5
    pool_timeout: 30
    connection_timeout: 10
```

### Ingestion scheduling

Use cron or CI/CD to keep context fresh:

```bash
# Daily context refresh
0 2 * * * cd /path/to/project && ktx ingest --connection warehouse
```

### Programmatic metric resolution

```typescript
import { KtxContext } from '@kaelio/ktx';

const ctx = await KtxContext.load();

// Resolve metric to SQL
const sql = await ctx.resolveMetric('monthly_revenue', {
  filters: { customer_segment: 'Enterprise' },
  timeRange: { start: '2024-01-01', end: '2024-12-31' }
});

console.log(sql);
// SELECT month, SUM(total_revenue) as monthly_revenue
// FROM revenue_metrics  
// WHERE customer_segment = 'Enterprise'
//   AND month BETWEEN '2024-01-01' AND '2024-12-31'
// GROUP BY month
```

## Best Practices

1. **Start with setup wizard**: Run `ktx setup` interactively before scripting
2. **Use read-only credentials**: Never give ktx write access to databases
3. **Commit semantic layer**: Track `semantic-layer/` in git for version control
4. **Document in wiki**: Add business context to `wiki/global/` for better agent responses
5. **Refresh regularly**: Schedule `ktx ingest` to keep context current
6. **Validate sources**: Run `ktx validate` before committing semantic source changes
7. **Use environment variables**: Never commit secrets in `ktx.yaml`
