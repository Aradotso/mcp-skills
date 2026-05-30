---
name: ktx-data-agent-context-layer
description: Context layer for AI data agents—automatically builds semantic layers, ingests wiki knowledge, and maps warehouses for accurate SQL queries through MCP
triggers:
  - set up ktx context layer for my warehouse
  - configure ktx to teach agents about my data warehouse
  - build a semantic layer with ktx for AI agents
  - ingest dbt models and wiki pages into ktx
  - connect ktx to my PostgreSQL/Snowflake/BigQuery database
  - create approved metric definitions with ktx
  - use ktx MCP server with Claude Code
  - search ktx semantic layer and wiki
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is an executable context layer that teaches AI agents (Claude Code, Codex, Cursor, OpenCode) how to query data warehouses accurately. It automatically builds a semantic layer from your database schema, ingests business knowledge from wikis and tools (dbt, Looker, Metabase, Notion), detects joinable columns, and exposes everything through MCP tools and CLI commands.

**Key features:**
- Automatically maps warehouse tables and detects join paths
- Ingests and organizes company knowledge from multiple sources
- Builds reusable metric definitions with fan/chasm trap resolution
- Exposes semantic layer + wiki through MCP for agent execution
- Read-only by design—never writes to your warehouse

**Supported databases:** PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite  
**Integrations:** dbt, MetricFlow, LookML, Looker, Metabase, Notion

## Installation

Install globally via npm:

```bash
npm install -g @kaelio/ktx
```

Or use directly with npx:

```bash
npx @kaelio/ktx setup
```

## Quick Setup

Run the interactive setup wizard:

```bash
ktx setup
```

This will:
1. Create or resume a ktx project in the current directory
2. Configure LLM provider (Anthropic, Vertex AI, or Claude Code session)
3. Configure embedding provider (OpenAI, Vertex AI, Voyage, or local)
4. Set up database connections
5. Configure context sources (dbt, Looker, Metabase, Notion)
6. Build initial context
7. Install agent integration (MCP)

Verify setup:

```bash
ktx status
```

Expected output after successful setup:

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

ktx creates the following structure:

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/
│   └── <connection-id>/             # YAML semantic sources (metrics, dimensions)
├── wiki/
│   ├── global/                      # Shared business context
│   └── user/<user-id>/              # User-scoped notes
├── raw-sources/
│   └── <connection-id>/             # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Commit to git:** `ktx.yaml`, `semantic-layer/`, `wiki/`  
**Keep local:** `.ktx/`

## Configuration

### ktx.yaml Structure

```yaml
project:
  name: my-analytics-project
  version: "0.1.0"

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key in .ktx/secrets.yaml or ANTHROPIC_API_KEY env var

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key in .ktx/secrets.yaml or OPENAI_API_KEY env var

connections:
  warehouse:
    type: postgresql
    host: localhost
    port: 5432
    database: analytics
    # Credentials in .ktx/secrets.yaml or env vars

context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./target/manifest.json
    catalog_path: ./target/catalog.json
```

### Environment Variables

Set provider API keys:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/vertex-ai-key.json
```

Database credentials:

```bash
export POSTGRES_USER=analytics_reader
export POSTGRES_PASSWORD=...
export SNOWFLAKE_ACCOUNT=...
export BIGQUERY_PROJECT_ID=...
```

## Core Commands

### Build and Update Context

Ingest from all configured sources:

```bash
ktx ingest
```

Ingest specific connection:

```bash
ktx ingest --connection warehouse
```

Force re-scan (ignore cache):

```bash
ktx ingest --force
```

### Search Semantic Layer

Search for metrics, dimensions, and sources:

```bash
ktx sl "revenue by region"
ktx sl "customer lifetime value"
ktx sl "churn rate"
```

Example output:

```text
Metric: monthly_recurring_revenue
  Description: Sum of active subscription values
  SQL: SUM(subscriptions.mrr_value)
  Dimensions: region, plan_type, customer_segment

Source: customers
  Columns: customer_id, region, segment, created_at
  Join keys: customer_id
```

### Search Wiki

Search business knowledge:

```bash
ktx wiki "refund policy"
ktx wiki "customer segmentation logic"
ktx wiki "revenue recognition rules"
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

The MCP server exposes tools for:
- Searching semantic layer
- Searching wiki
- Querying metrics with automatic join resolution
- Inspecting data sources

## Working with Semantic Sources

### Create a Metric

Create `semantic-layer/warehouse/metrics.yaml`:

```yaml
metrics:
  - name: total_revenue
    description: Sum of all completed order amounts
    type: sum
    sql: orders.amount
    filters:
      - column: orders.status
        operator: "="
        value: "'completed'"
    dimensions:
      - customer.region
      - customer.segment
      - orders.created_at

  - name: average_order_value
    description: Average value of completed orders
    type: average
    sql: orders.amount
    filters:
      - column: orders.status
        operator: "="
        value: "'completed'"
```

### Create a Dimension

```yaml
dimensions:
  - name: customer_region
    description: Geographic region of customer
    type: categorical
    sql: customers.region
    values:
      - NA
      - EU
      - APAC
      - LATAM

  - name: order_date
    description: Date order was placed
    type: time
    sql: orders.created_at
    time_granularities:
      - day
      - week
      - month
      - quarter
      - year
```

### Define Joins

```yaml
joins:
  - left_source: orders
    right_source: customers
    join_type: left
    on:
      - left: orders.customer_id
        right: customers.customer_id

  - left_source: orders
    right_source: products
    join_type: left
    on:
      - left: orders.product_id
        right: products.product_id
```

## Working with Wiki

### Create Wiki Pages

Create `wiki/global/revenue-recognition.md`:

```markdown
# Revenue Recognition Policy

Revenue is recognized when:
1. Order status = 'completed'
2. Payment received
3. Service delivered (for SaaS: subscription active)

## Calculation Rules

- MRR: Sum of active subscription values at month-end
- ARR: MRR * 12
- Churn: Cancelled subscriptions / total subscriptions at period start

## Data Sources

Primary table: `analytics.orders`
Required filters: `status = 'completed' AND payment_status = 'paid'`
```

### User-Scoped Notes

Create `wiki/user/<your-user-id>/analysis-notes.md`:

```markdown
# Q4 2024 Revenue Analysis Notes

Customer segment "enterprise" shows 40% higher LTV than "smb".
Churn spikes in month 3—investigate onboarding process.
```

## TypeScript Integration

ktx is primarily a CLI tool, but you can integrate it programmatically:

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function searchSemanticLayer(query: string): Promise<string> {
  const { stdout } = await execAsync(`ktx sl "${query}"`);
  return stdout;
}

async function ingestContext(connectionId?: string): Promise<void> {
  const cmd = connectionId 
    ? `ktx ingest --connection ${connectionId}`
    : 'ktx ingest';
  await execAsync(cmd);
}

// Example usage
async function analyzeRevenue() {
  const results = await searchSemanticLayer('revenue metrics');
  console.log(results);
}
```

## Common Patterns

### Daily Context Refresh

Automate context updates with a cron job or CI workflow:

```bash
#!/bin/bash
# update-ktx-context.sh

cd /path/to/project
ktx ingest --force
git add semantic-layer/ wiki/
git commit -m "chore: update ktx context [skip ci]"
git push
```

### Multi-Database Setup

Configure multiple warehouses in `ktx.yaml`:

```yaml
connections:
  production:
    type: snowflake
    account: ${SNOWFLAKE_ACCOUNT}
    warehouse: ANALYTICS_WH
    database: PROD

  staging:
    type: snowflake
    account: ${SNOWFLAKE_ACCOUNT}
    warehouse: ANALYTICS_WH
    database: STAGING

  local:
    type: postgresql
    host: localhost
    database: dev_analytics
```

Ingest from specific connection:

```bash
ktx ingest --connection production
ktx ingest --connection staging
```

### Integrating dbt

Point ktx at your dbt project artifacts:

```yaml
context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./dbt/target/manifest.json
    catalog_path: ./dbt/target/catalog.json
    sources_path: ./dbt/models/sources.yml
```

Run dbt, then ingest:

```bash
cd dbt
dbt run
dbt docs generate
cd ..
ktx ingest --force
```

### Agent Workflow

From Claude Code, Codex, or Cursor:

**User:** "What's our MRR by customer segment last month?"

**Agent:**
1. Uses ktx MCP tool to search semantic layer for "MRR" and "customer segment"
2. Finds `monthly_recurring_revenue` metric with `customer.segment` dimension
3. Uses ktx to query metric with time filter
4. Returns results and visualization

Example MCP tool call (handled automatically by agent):

```json
{
  "tool": "ktx_query_metric",
  "arguments": {
    "metric": "monthly_recurring_revenue",
    "dimensions": ["customer_segment"],
    "time_range": {
      "start": "2024-04-01",
      "end": "2024-04-30"
    }
  }
}
```

## Troubleshooting

### "LLM ready: no"

Ensure API key is set:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
ktx setup  # Re-run setup to verify
```

Or configure in `.ktx/secrets.yaml`:

```yaml
anthropic_api_key: sk-ant-...
```

### "Embeddings ready: no"

Set OpenAI API key:

```bash
export OPENAI_API_KEY=sk-...
ktx setup
```

Or use local embeddings (slower):

```bash
ktx setup  # Choose "local" when prompted for embeddings
```

### Database Connection Failed

Verify credentials:

```bash
# PostgreSQL example
psql -h localhost -U analytics_reader -d analytics -c "SELECT 1;"
```

Check `.ktx/secrets.yaml` has correct values:

```yaml
connections:
  warehouse:
    user: analytics_reader
    password: correct_password
```

### "ktx context built: no"

Run ingest manually:

```bash
ktx ingest --force
```

Check for errors in output. Common issues:
- Missing dbt artifacts: run `dbt docs generate`
- Database permissions: ensure read access to all tables
- LLM quota exceeded: wait or check provider billing

### MCP Server Not Starting

Check if another instance is running:

```bash
ps aux | grep "ktx mcp"
```

Kill existing process:

```bash
pkill -f "ktx mcp start"
```

Restart:

```bash
ktx mcp start --project-dir /path/to/project
```

### Empty Search Results

Context may not be built:

```bash
ktx status  # Check "ktx context built"
ktx ingest --force  # Rebuild
```

Verify semantic sources exist:

```bash
ls -la semantic-layer/warehouse/
cat semantic-layer/warehouse/metrics.yaml
```

## Advanced Configuration

### Custom LLM Provider

Use AI Gateway or self-hosted endpoint:

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  base_url: https://my-gateway.example.com/v1
  api_key: ${GATEWAY_API_KEY}
```

### Local Embeddings

For air-gapped environments:

```yaml
embeddings:
  provider: local
  model: all-MiniLM-L6-v2  # Downloaded automatically
```

### Read-Only Database User

Create restricted user for ktx:

```sql
-- PostgreSQL
CREATE USER ktx_reader WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE analytics TO ktx_reader;
GRANT USAGE ON SCHEMA public TO ktx_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ktx_reader;
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
  GRANT SELECT ON TABLES TO ktx_reader;
```

```yaml
# ktx.yaml
connections:
  warehouse:
    type: postgresql
    user: ktx_reader
    # password in .ktx/secrets.yaml
```

## Best Practices

1. **Commit semantic sources and wiki to git**—they represent approved business logic
2. **Run `ktx ingest` regularly**—daily or after schema changes
3. **Use descriptive metric names**—`monthly_recurring_revenue` not `mrr_calc`
4. **Document filter requirements**—capture business rules in metric definitions
5. **Start MCP server before opening agent**—run `ktx mcp start` first
6. **Test queries manually**—use `ktx sl` to verify semantic layer before agents use it
7. **Keep `.ktx/` local**—never commit secrets or cache to git

## Related Commands

```bash
ktx --help                    # Full command reference
ktx --version                 # Check installed version
ktx config get               # View current configuration
ktx config set llm.model claude-opus-4  # Update config
ktx validate                 # Check semantic sources for errors
ktx export --format json     # Export semantic layer metadata
```

For complete documentation, visit [docs.kaelio.com/ktx](https://docs.kaelio.com/ktx).
