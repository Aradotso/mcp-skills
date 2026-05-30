---
name: ktx-ai-data-agents
description: Use ktx to build an executable context layer for AI data agents, enabling accurate warehouse queries through semantic layer, skills, and memory
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer for my warehouse
  - install ktx to let agents query my database
  - build ktx context from dbt and warehouse metadata
  - create ktx wiki pages for business metrics
  - integrate ktx with Claude Code for data analysis
  - search ktx semantic layer for revenue metrics
  - troubleshoot ktx context ingestion
---

# ktx AI Data Agents

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It combines approved metric definitions, joinable columns, business knowledge from wikis and dbt, and a semantic layer into one searchable surface. Agents use ktx through MCP tools to fetch canonical SQL and metric logic instead of inventing queries from scratch.

## What ktx Does

- **Learns from company knowledge**: Ingests dbt, Looker, Metabase, Notion, and wiki content; organizes and deduplicates it; flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, resolves fan/chasm traps
- **Builds a semantic layer**: Combines raw tables and high-level metrics through a join graph
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search across wiki and semantic entities

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite.

## Installation

### Global CLI

```bash
npm install -g @kaelio/ktx
```

### Project-local

```bash
npm install @kaelio/ktx
npx ktx setup
```

## Quick Setup

```bash
ktx setup
```

This interactive command:
1. Creates or resumes a local ktx project (`ktx.yaml`)
2. Configures LLM provider (Anthropic API, Vertex AI, or Claude Code SDK)
3. Configures embedding provider (OpenAI, Voyage AI, etc.)
4. Adds database connections (warehouse credentials)
5. Adds context sources (dbt projects, Looker instances, Notion pages)
6. Builds initial context via `ktx ingest`
7. Installs agent integration (MCP for Claude Code, Codex, Cursor, OpenCode)

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

## Key Commands

### Check project status

```bash
ktx status
```

Shows readiness of LLM, embeddings, databases, context sources, and agent integration.

### Build context

```bash
# Ingest all configured connections
ktx ingest

# Ingest specific connection
ktx ingest --connection-id warehouse

# Ingest specific context source
ktx ingest --source-id dbt_main
```

### Search semantic layer

```bash
ktx sl "revenue"
ktx sl "monthly active users" --limit 10
```

Returns semantic sources (metrics, dimensions, entities) matching the query.

### Search wiki

```bash
ktx wiki "refund policy"
ktx wiki "customer lifecycle" --scope global
ktx wiki "my notes on churn" --scope user
```

Returns wiki pages from `wiki/global/` or `wiki/user/<user-id>/`.

### Start MCP server

```bash
ktx mcp start
ktx mcp start --project-dir /path/to/project
```

Required for agent clients (Claude Code, Codex, Cursor, OpenCode) to use ktx tools.

### Validate configuration

```bash
ktx validate
ktx validate --connection-id warehouse
```

Tests database connections and context source access.

## Configuration

### ktx.yaml Structure

```yaml
version: 1
project:
  name: analytics
  llm:
    provider: anthropic
    model: claude-sonnet-4-6
  embeddings:
    provider: openai
    model: text-embedding-3-small

connections:
  - id: warehouse
    type: postgres
    config:
      host: ${POSTGRES_HOST}
      port: 5432
      database: analytics
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      schema: public
    options:
      sampleRows: 1000
      maxTables: 500

contextSources:
  - id: dbt_main
    type: dbt
    config:
      profilesDir: ~/.dbt
      projectDir: ./dbt
      profile: analytics
      target: prod
  - id: notion_docs
    type: notion
    config:
      token: ${NOTION_TOKEN}
      databaseId: ${NOTION_DATABASE_ID}
```

Secrets go in `.ktx/secrets.env`:

```bash
POSTGRES_HOST=warehouse.example.com
POSTGRES_USER=readonly_user
POSTGRES_PASSWORD=secret123
NOTION_TOKEN=secret_xyz
NOTION_DATABASE_ID=abc123
```

### LLM Providers

#### Anthropic API

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  apiKey: ${ANTHROPIC_API_KEY}
```

#### Google Vertex AI

```yaml
llm:
  provider: vertex
  model: claude-sonnet-4-6
  config:
    projectId: ${GCP_PROJECT_ID}
    region: us-central1
```

#### Claude Code SDK (local session)

```yaml
llm:
  provider: claude-code
  model: claude-sonnet-4-6
```

### Embedding Providers

#### OpenAI

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-small
  apiKey: ${OPENAI_API_KEY}
```

#### Voyage AI

```yaml
embeddings:
  provider: voyage
  model: voyage-3
  apiKey: ${VOYAGE_API_KEY}
```

## Project Layout

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/warehouse/        # YAML semantic sources per connection
│   ├── metrics/
│   │   └── revenue.yaml
│   ├── dimensions/
│   │   └── customer_segment.yaml
│   └── entities/
│       └── customer.yaml
├── wiki/global/                     # Shared business context
│   ├── refund-policy.md
│   └── metric-definitions.md
├── wiki/user/alice/                 # User-scoped notes
│   └── analysis-notes.md
├── raw-sources/warehouse/           # Ingest artifacts (git-ignored)
│   ├── tables.json
│   ├── columns.json
│   └── sample-data.parquet
└── .ktx/                            # Local state and secrets (git-ignored)
    ├── secrets.env
    ├── embeddings.db
    └── mcp-state.json
```

**Commit**: `ktx.yaml`, `semantic-layer/`, `wiki/global/`  
**Ignore**: `.ktx/`, `raw-sources/`

## Code Examples

### TypeScript: Programmatic ktx Usage

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load project
const project = await KtxProject.load('/path/to/project');

// Search semantic layer
const results = await project.semanticLayer.search('revenue', { limit: 5 });
for (const result of results) {
  console.log(`${result.type}: ${result.name}`);
  console.log(`SQL: ${result.sql}`);
}

// Search wiki
const wikiPages = await project.wiki.search('refund policy', { scope: 'global' });
for (const page of wikiPages) {
  console.log(`${page.title}: ${page.path}`);
}

// Ingest context
await project.ingest({ connectionId: 'warehouse' });
```

### TypeScript: Custom Context Source Connector

```typescript
import { ContextSource, ContextSourceConfig } from '@kaelio/ktx';

interface CustomSourceConfig extends ContextSourceConfig {
  apiUrl: string;
  apiKey: string;
}

class CustomContextSource extends ContextSource<CustomSourceConfig> {
  async validate(): Promise<void> {
    const response = await fetch(`${this.config.apiUrl}/health`, {
      headers: { Authorization: `Bearer ${this.config.apiKey}` }
    });
    if (!response.ok) throw new Error('Invalid API key or URL');
  }

  async ingest(project: KtxProject): Promise<void> {
    const response = await fetch(`${this.config.apiUrl}/metrics`, {
      headers: { Authorization: `Bearer ${this.config.apiKey}` }
    });
    const metrics = await response.json();

    for (const metric of metrics) {
      await project.semanticLayer.upsertMetric({
        id: metric.id,
        name: metric.name,
        sql: metric.sql,
        description: metric.description,
        connectionId: 'warehouse'
      });
    }
  }
}
```

### Python: Semantic Layer Query Planning

ktx includes a Python semantic-layer query planner (`ktx-sl`):

```python
from ktx_sl import SemanticLayer, MetricQuery

# Load semantic layer
sl = SemanticLayer.load('/path/to/project/semantic-layer/warehouse')

# Plan a metric query
query = MetricQuery(
    metrics=['revenue', 'order_count'],
    dimensions=['customer_segment', 'region'],
    filters={'order_date': {'gte': '2024-01-01'}}
)

plan = sl.plan(query)
print(plan.sql)  # Canonical SQL with joins resolved
```

## Common Patterns

### Adding a Wiki Page

Create `wiki/global/refund-policy.md`:

```markdown
# Refund Policy

Refunds are issued within 30 days of purchase for orders under $500.
Orders over $500 require manager approval.

Refund metric: `COUNT(CASE WHEN refund_issued THEN 1 END)`
```

Then rebuild embeddings:

```bash
ktx ingest --source-id wiki_global
```

### Defining a Semantic Metric

Create `semantic-layer/warehouse/metrics/revenue.yaml`:

```yaml
id: revenue
name: Revenue
type: metric
sql: SUM(orders.amount)
description: Total order revenue in USD
entity: order
connectionId: warehouse
tags:
  - finance
  - kpi
```

### Detecting Joinable Columns

ktx auto-detects joins during ingestion. To manually annotate:

```yaml
# semantic-layer/warehouse/entities/customer.yaml
id: customer
name: Customer
type: entity
primaryKey: customer_id
connectionId: warehouse
joinableWith:
  - entity: order
    foreignKey: customer_id
```

### Fan Trap Resolution

ktx resolves fan traps automatically. If a query joins `customers -> orders -> line_items`, ktx generates CTEs to prevent row multiplication:

```sql
WITH orders_agg AS (
  SELECT customer_id, SUM(amount) AS revenue
  FROM orders
  GROUP BY customer_id
)
SELECT c.customer_id, o.revenue
FROM customers c
LEFT JOIN orders_agg o ON c.customer_id = o.customer_id;
```

### Agent Integration via MCP

In your Claude Code / Codex project settings:

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

Or auto-install via `ktx setup` agent integration step.

## MCP Tools Exposed

When `ktx mcp start` is running, agents have access to:

- `ktx_search_semantic_layer`: Search metrics, dimensions, entities
- `ktx_search_wiki`: Search wiki pages
- `ktx_get_metric_sql`: Fetch canonical SQL for a metric
- `ktx_get_table_schema`: Get table DDL and sample rows
- `ktx_list_connections`: List configured database connections
- `ktx_validate_query`: Validate a SQL query against the semantic layer

Example agent prompt:

```text
Use ktx_search_semantic_layer to find the "monthly active users" metric,
then use ktx_get_metric_sql to show me the canonical SQL.
```

## Troubleshooting

### "LLM not configured"

Run `ktx setup` and select an LLM provider. Ensure API keys are in `.ktx/secrets.env`:

```bash
ANTHROPIC_API_KEY=sk-ant-...
```

Or use Claude Code SDK (no key required):

```yaml
llm:
  provider: claude-code
  model: claude-sonnet-4-6
```

### "Database connection failed"

Test connection:

```bash
ktx validate --connection-id warehouse
```

Verify credentials in `.ktx/secrets.env`. For read-only access, grant `SELECT` on all schemas:

```sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;
```

### "Context not built"

Run ingestion:

```bash
ktx ingest
```

Check `raw-sources/<connection-id>/` for ingestion logs. If tables are missing, verify schema permissions.

### "MCP server not responding"

Restart the server:

```bash
ktx mcp stop
ktx mcp start --project-dir /path/to/project
```

Ensure agent client is configured to call `ktx mcp start` with the correct `--project-dir`.

### "Semantic search returns no results"

Rebuild embeddings:

```bash
rm .ktx/embeddings.db
ktx ingest
```

Verify embedding provider is configured:

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-small
  apiKey: ${OPENAI_API_KEY}
```

### "Contradictions detected" warning

ktx flags contradictions when the same metric is defined differently across sources (e.g., dbt vs. Looker). Review `raw-sources/<connection-id>/contradictions.json` and reconcile definitions manually.

## Advanced: Custom Database Connector

```typescript
import { DatabaseConnector, TableMetadata } from '@kaelio/ktx';

class CustomDatabaseConnector extends DatabaseConnector {
  async connect(): Promise<void> {
    // Initialize connection
  }

  async listTables(schema: string): Promise<string[]> {
    // Return table names
  }

  async getTableMetadata(table: string): Promise<TableMetadata> {
    // Return columns, types, primary keys, foreign keys
  }

  async sampleRows(table: string, limit: number): Promise<Record<string, any>[]> {
    // Return sample rows
  }

  async executeQuery(sql: string): Promise<Record<string, any>[]> {
    // Execute read-only query
  }

  async disconnect(): Promise<void> {
    // Close connection
  }
}
```

Register in `ktx.yaml`:

```yaml
connections:
  - id: custom_db
    type: custom
    connector: ./connectors/custom-db.ts
    config:
      host: ${CUSTOM_DB_HOST}
      apiKey: ${CUSTOM_DB_KEY}
```

## Resources

- [ktx Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
