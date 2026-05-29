---
name: ktx-data-agent-context-layer
description: Install and use ktx, the executable context layer that teaches AI agents to query data warehouses accurately through MCP with skills and memory
triggers:
  - set up ktx for data agent queries
  - configure ktx with my warehouse
  - build ktx semantic layer context
  - search ktx wiki or semantic layer
  - integrate ktx with Claude Code or Codex
  - use ktx to query my database
  - ingest metadata into ktx context
  - troubleshoot ktx setup or connection
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

ktx is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It builds and maintains approved metric definitions, joinable columns, and business knowledge by ingesting from dbt, MetricFlow, LookML, Looker, Metabase, Notion, and raw database introspection. ktx exposes this context through CLI and MCP tools for agents like Claude Code, Codex, Cursor, and OpenCode.

## Key Capabilities

- **Automatic warehouse mapping**: Samples tables, detects joinable columns, resolves fan/chasm traps
- **Semantic layer**: Combines raw tables and high-level metrics with declarative query interface
- **Wiki integration**: Ingests company knowledge, removes duplicates, flags contradictions
- **Agent-ready**: CLI and MCP server with full-text + semantic search
- **Read-only**: Never writes to your database
- **Local-first**: No hosted service, runs entirely on your machine

## Installation

Install ktx globally via npm:

```bash
npm install -g @kaelio/ktx
```

Or run directly with npx:

```bash
npx @kaelio/ktx --help
```

For agent skill installation (if using Codex/Claude Code with skills):

```bash
npx skills add Kaelio/ktx --skill ktx
```

## Setup and Configuration

### Initial Setup

Create or resume a ktx project:

```bash
ktx setup
```

This interactive command:
1. Creates `ktx.yaml` in the current directory
2. Configures LLM provider (Anthropic, Vertex AI, or Claude Agent SDK)
3. Configures embedding provider (OpenAI, Voyage, or Vertex AI)
4. Sets up database connections
5. Configures context sources (dbt, MetricFlow, LookML, etc.)
6. Builds initial context
7. Installs agent integration

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

Edit `ktx.yaml` to configure providers and connections:

```yaml
version: 1

llm:
  provider: anthropic
  model: claude-sonnet-4-6

embeddings:
  provider: openai
  model: text-embedding-3-small

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    ssl: false

context_sources:
  dbt_main:
    type: dbt
    path: ./dbt-project
    connection_id: warehouse

  notion_wiki:
    type: notion
    connection_id: warehouse
```

### Environment Variables

Store secrets in environment variables (never commit):

```bash
# LLM provider
export ANTHROPIC_API_KEY=sk-ant-...
# or
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/vertex-key.json

# Embeddings provider
export OPENAI_API_KEY=sk-...
# or
export VOYAGE_API_KEY=pa-...

# Database credentials
export DATABASE_PASSWORD=...

# Notion (if using)
export NOTION_API_KEY=secret_...
```

## Core Commands

### Build Context

Ingest metadata and build context from all configured sources:

```bash
ktx ingest
```

Ingest from specific connection:

```bash
ktx ingest --connection warehouse
```

Force rebuild (skip cache):

```bash
ktx ingest --force
```

### Search Semantic Layer

Search for metrics, dimensions, or tables:

```bash
ktx sl "monthly revenue"
ktx sl "customer churn rate"
ktx sl "orders table"
```

Example output:

```text
Semantic Layer Results (3 matches)

1. Metric: monthly_revenue
   Description: Total revenue aggregated by month
   SQL: SUM(orders.amount)
   Dimensions: month, product_category
   Joinable from: orders, customers

2. Dimension: customer_churn_date
   Table: customers
   Type: date
   Description: Date when customer churned
```

### Search Wiki

Search business knowledge and documentation:

```bash
ktx wiki "refund policy"
ktx wiki "fiscal year definition"
ktx wiki "customer segmentation logic"
```

Example output:

```text
Wiki Results (2 matches)

1. Refund Policy (global/policies/refunds.md)
   Last updated: 2026-05-15
   Excerpt: Customers can request refunds within 30 days...

2. Revenue Recognition (global/finance/revenue.md)
   Last updated: 2026-05-10
   Excerpt: We recognize revenue when the service is delivered...
```

### MCP Server

Start the MCP server for agent clients:

```bash
ktx mcp start
```

With custom port:

```bash
ktx mcp start --port 3001
```

With specific project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

The MCP server exposes tools for agents:
- `ktx_search_semantic_layer`: Search metrics and dimensions
- `ktx_search_wiki`: Search business knowledge
- `ktx_get_metric`: Get detailed metric definition
- `ktx_list_tables`: List available tables
- `ktx_get_table`: Get table schema and metadata

## Agent Integration

### Claude Code / Codex

After running `ktx setup`, the CLI configures your agent automatically. To use ktx from Claude Code:

```text
Search ktx for revenue metrics
```

```text
What tables are joinable to the customers table?
```

```text
Show me the SQL definition for customer_lifetime_value
```

### Cursor / OpenCode

Add ktx as an MCP server in your agent settings:

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

## TypeScript API (Programmatic Usage)

While ktx is primarily a CLI tool, you can import its core modules:

```typescript
import { ContextEngine } from '@kaelio/ktx/context';
import { LLMProvider } from '@kaelio/ktx/llm';
import { DatabaseConnector } from '@kaelio/ktx/connectors';

// Initialize context engine
const engine = new ContextEngine({
  projectDir: '/path/to/project',
  llmProvider: new LLMProvider({
    provider: 'anthropic',
    model: 'claude-sonnet-4-6',
    apiKey: process.env.ANTHROPIC_API_KEY
  })
});

// Load project
await engine.loadProject();

// Search semantic layer
const results = await engine.searchSemanticLayer('revenue', {
  limit: 10,
  includeMetrics: true,
  includeDimensions: true
});

console.log(results);

// Search wiki
const wikiResults = await engine.searchWiki('refund policy', {
  limit: 5
});

console.log(wikiResults);
```

### Database Connector Example

```typescript
import { PostgresConnector } from '@kaelio/ktx/connectors';

const connector = new PostgresConnector({
  host: 'localhost',
  port: 5432,
  database: 'analytics',
  schema: 'public',
  user: process.env.DATABASE_USER,
  password: process.env.DATABASE_PASSWORD
});

// Connect and introspect
await connector.connect();
const tables = await connector.listTables();
const schema = await connector.getTableSchema('orders');

console.log(`Found ${tables.length} tables`);
console.log(`orders table has ${schema.columns.length} columns`);

await connector.disconnect();
```

## Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/                  # Generated semantic sources
│   └── warehouse/
│       ├── metrics.yaml
│       └── dimensions.yaml
├── wiki/                            # Business knowledge
│   ├── global/                      # Shared context
│   │   ├── policies/
│   │   └── definitions/
│   └── user/                        # User-scoped notes
│       └── alice/
├── raw-sources/                     # Ingest artifacts
│   └── warehouse/
│       ├── tables.json
│       └── sample-queries.json
└── .ktx/                           # Local state (git-ignored)
    ├── embeddings.db
    └── secrets.json
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local (add to `.gitignore`).

## Common Patterns

### Pattern: First-time Setup

```bash
# Install globally
npm install -g @kaelio/ktx

# Navigate to your project
cd ~/projects/analytics

# Run interactive setup
ktx setup

# Verify everything is ready
ktx status

# Test semantic layer search
ktx sl "revenue"

# Test wiki search
ktx wiki "business logic"
```

### Pattern: Daily Agent Usage

From Claude Code or Codex:

```text
User: What's our total revenue last quarter?

Agent: Let me search ktx for revenue metrics.
[Uses ktx_search_semantic_layer tool]

Agent: I found the quarterly_revenue metric. It's defined as 
SUM(orders.amount) grouped by fiscal_quarter. Let me query it.
[Generates SQL using metric definition]
```

### Pattern: Incremental Context Updates

```bash
# After adding new dbt models
cd ~/dbt-project
git pull origin main

# Rebuild ktx context
ktx ingest --connection warehouse

# Verify new metrics are available
ktx sl "new_metric_name"
```

### Pattern: Multiple Warehouses

Configure multiple connections in `ktx.yaml`:

```yaml
connections:
  production:
    type: postgres
    host: prod.example.com
    database: analytics

  staging:
    type: postgres
    host: staging.example.com
    database: analytics

context_sources:
  dbt_prod:
    type: dbt
    path: ./dbt-project
    connection_id: production

  dbt_staging:
    type: dbt
    path: ./dbt-project
    connection_id: staging
```

Ingest from specific connection:

```bash
ktx ingest --connection production
ktx ingest --connection staging
```

### Pattern: Team Wiki Collaboration

```bash
# Add global business knowledge
mkdir -p wiki/global/policies
cat > wiki/global/policies/refunds.md << EOF
# Refund Policy

Customers can request refunds within 30 days of purchase.
Refunds are processed within 5-7 business days.
EOF

# Rebuild context to index new content
ktx ingest

# Search for it
ktx wiki "refund policy"
```

### Pattern: Contradiction Detection

ktx automatically flags contradictions across sources:

```bash
ktx ingest --connection warehouse

# Output may include:
# Warning: Contradictory definitions found for metric 'monthly_revenue'
#   - dbt defines it as SUM(orders.amount)
#   - Looker defines it as SUM(orders.total)
# Review semantic-layer/warehouse/contradictions.yaml
```

Review and resolve in the generated file:

```yaml
contradictions:
  - metric: monthly_revenue
    sources:
      - dbt: SUM(orders.amount)
      - looker: SUM(orders.total)
    resolution: prefer_dbt  # or: prefer_looker, manual
```

## Troubleshooting

### "Project not ready" Error

```bash
ktx status
# Output: Project ready: no

# Run setup to initialize or resume project
ktx setup
```

### "LLM not ready" Error

Check that API keys are set:

```bash
echo $ANTHROPIC_API_KEY
# or
echo $GOOGLE_APPLICATION_CREDENTIALS
```

Re-run setup:

```bash
ktx setup
```

### "Database connection failed"

Verify credentials and network access:

```bash
# Test connection manually
psql -h localhost -U myuser -d analytics

# Check ktx.yaml connection config
cat ktx.yaml | grep -A 10 connections:

# Verify environment variables
echo $DATABASE_PASSWORD
```

### "Context not built"

Rebuild context from scratch:

```bash
ktx ingest --force
```

### "MCP server not starting"

Check if port is already in use:

```bash
lsof -i :3000
# or
netstat -tuln | grep 3000
```

Start on different port:

```bash
ktx mcp start --port 3001
```

### Empty Search Results

Ensure context is built:

```bash
ktx status
# Should show: ktx context built: yes

# If not, rebuild
ktx ingest
```

Check that embeddings are configured:

```bash
cat ktx.yaml | grep -A 5 embeddings:
```

### dbt Source Not Found

Verify dbt project path in `ktx.yaml`:

```yaml
context_sources:
  dbt_main:
    type: dbt
    path: ./dbt-project  # Must point to directory with dbt_project.yml
    connection_id: warehouse
```

Ensure `dbt_project.yml` exists:

```bash
ls -la dbt-project/dbt_project.yml
```

### Notion Integration Issues

Verify API key:

```bash
echo $NOTION_API_KEY
```

Ensure Notion integration has access to pages:
1. Go to Notion workspace settings
2. Connections → Internal integrations
3. Select your integration
4. Share pages with the integration

### Performance Issues with Large Warehouses

Limit table sampling:

```yaml
connections:
  warehouse:
    type: postgres
    # ... connection details ...
    sample_rows: 100  # Default is 1000
    max_tables: 500   # Default is unlimited
```

Skip expensive introspection:

```bash
ktx ingest --skip-sampling
```

## Advanced Configuration

### Custom Semantic Layer Definitions

Manually define metrics in `semantic-layer/<connection-id>/metrics.yaml`:

```yaml
metrics:
  - name: customer_lifetime_value
    description: Total revenue per customer over their lifetime
    type: sum
    sql: orders.amount
    dimensions:
      - customer_id
      - signup_date
    filters:
      - orders.status = 'completed'
    
  - name: churn_rate
    description: Percentage of customers who churned this month
    type: ratio
    numerator: COUNT(DISTINCT churned_customers.id)
    denominator: COUNT(DISTINCT all_customers.id)
    dimensions:
      - month
```

### Custom Join Graph

Override automatic join detection in `semantic-layer/<connection-id>/joins.yaml`:

```yaml
joins:
  - left_table: orders
    right_table: customers
    left_column: customer_id
    right_column: id
    type: left
    
  - left_table: orders
    right_table: products
    left_column: product_id
    right_column: id
    type: left
```

### LLM Configuration

Use different models per task:

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  ingest_model: claude-haiku-4-6  # Faster for ingestion
  search_model: claude-sonnet-4-6  # Better for search
```

Use local Claude Code session:

```yaml
llm:
  provider: claude_agent_sdk
  # No API key needed
```

### Embedding Configuration

Switch providers:

```yaml
embeddings:
  provider: voyage
  model: voyage-3

# or

embeddings:
  provider: vertex
  model: text-embedding-004
  project: my-gcp-project
  location: us-central1
```

## Best Practices

1. **Version control**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Add `.ktx/` to `.gitignore`.

2. **Incremental updates**: Run `ktx ingest` after dbt model changes or wiki updates.

3. **Review contradictions**: Check `semantic-layer/<connection-id>/contradictions.yaml` regularly.

4. **Use descriptive metric names**: Prefer `monthly_recurring_revenue` over `mrr`.

5. **Document business logic**: Add definitions to `wiki/global/` so agents understand context.

6. **Test queries**: Use `ktx sl` to verify metrics are discoverable before relying on them in agent workflows.

7. **Separate environments**: Use different project directories for prod/staging to avoid mixing context.

8. **Monitor costs**: ktx uses your LLM API keys. Use cheaper models for ingestion if needed.

---

For full documentation, visit [docs.kaelio.com/ktx](https://docs.kaelio.com/ktx).
