---
name: ktx-ai-data-agents
description: Install and use ktx, the context layer that teaches AI agents to query data warehouses accurately with semantic layers, wikis, and approved metrics
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer for warehouse
  - use ktx to teach agent about our data
  - install ktx mcp server for claude
  - build ktx context from dbt and warehouse
  - query warehouse with ktx approved metrics
  - ingest data sources into ktx wiki
  - troubleshoot ktx agent integration
---

# ktx AI Data Agents Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It combines approved metric definitions, joinable column detection, business knowledge from wikis/dbt/Looker, and semantic layer capabilities into a single searchable surface for agents via CLI and MCP.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, dbt metadata, LookML, Looker, Metabase, and Notion; organizes, deduplicates, and flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds a semantic layer**: Combines raw tables and high-level metrics through a join graph that resolves chasm and fan traps
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search across wiki and semantic-layer entities

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, and SQLite.

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Using npx (No Installation)

```bash
npx @kaelio/ktx setup
```

### For Agent Skill Integration

From your project directory in Claude Code, Codex, Cursor, or OpenCode:

```text
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install and configure ktx in this project.
```

## Initial Setup

Run the interactive setup wizard:

```bash
ktx setup
```

This will:
1. Create or resume a local ktx project (generates `ktx.yaml`)
2. Configure LLM provider (Anthropic API, Vertex AI, AI Gateway, or Claude Code session)
3. Configure embedding provider (OpenAI, Vertex AI, Voyage AI, or Claude Code)
4. Set up database connections (warehouse credentials)
5. Configure context sources (dbt, Looker, Metabase, Notion, etc.)
6. Build initial context
7. Install agent integration (MCP server configuration)

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

## Configuration

### ktx.yaml Structure

```yaml
version: "1"
llm:
  provider: anthropic
  model: claude-sonnet-4-6
embeddings:
  provider: openai
  model: text-embedding-3-small
databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    username: readonly_user
    # password stored in .ktx/secrets.yaml
context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
  notion_wiki:
    type: notion
    # credentials in .ktx/secrets.yaml
```

### Environment Variables for Secrets

ktx stores secrets in `.ktx/secrets.yaml` (gitignored). Reference environment variables:

```bash
export ANTHROPIC_API_KEY=your_key_here
export OPENAI_API_KEY=your_key_here
export NOTION_API_KEY=your_key_here
```

### LLM Provider Configuration

```bash
# Configure Anthropic API
ktx config set llm.provider anthropic
ktx config set llm.model claude-sonnet-4-6
ktx config set llm.api_key $ANTHROPIC_API_KEY

# Configure Vertex AI
ktx config set llm.provider vertex
ktx config set llm.project your-gcp-project
ktx config set llm.location us-central1

# Use Claude Code session (no API key needed)
ktx config set llm.provider claude-code
```

### Database Connection

```bash
# Add PostgreSQL connection
ktx connection add warehouse \
  --type postgres \
  --host localhost \
  --port 5432 \
  --database analytics \
  --username readonly_user \
  --password $DB_PASSWORD

# Add Snowflake connection
ktx connection add snowflake_prod \
  --type snowflake \
  --account myorg-prod \
  --warehouse COMPUTE_WH \
  --database ANALYTICS \
  --schema PUBLIC \
  --username $SNOWFLAKE_USER \
  --password $SNOWFLAKE_PASSWORD
```

### Context Sources

```bash
# Add dbt source
ktx source add dbt_main \
  --type dbt \
  --path ./dbt \
  --profiles-dir ~/.dbt

# Add Looker source
ktx source add looker \
  --type looker \
  --instance mycompany.looker.com \
  --client-id $LOOKER_CLIENT_ID \
  --client-secret $LOOKER_CLIENT_SECRET

# Add Notion source
ktx source add notion_wiki \
  --type notion \
  --token $NOTION_API_KEY \
  --database-id your-database-id
```

## Core Commands

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific source
ktx ingest --source dbt_main

# Force re-ingest (ignore cache)
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, entities)
ktx sl "revenue"
ktx sl "customer lifetime value"

# Search wiki (business knowledge, documentation)
ktx wiki "refund policy"
ktx wiki "metric definitions"

# Search with JSON output
ktx sl "revenue" --json
```

### Semantic Layer Queries

```typescript
// Query planning happens in the semantic layer
// Agents use ktx to resolve metrics declaratively

// Example: Agent asks "What was revenue last month?"
// ktx resolves to the canonical metric definition:

import { executeQuery } from '@kaelio/ktx';

const result = await executeQuery({
  metrics: ['revenue'],
  dimensions: ['date'],
  filters: {
    date: {
      operator: 'between',
      values: ['2024-04-01', '2024-04-30']
    }
  }
});
```

### MCP Server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status

# Stop MCP server
ktx mcp stop
```

## Project Layout

```text
my-project/
├── ktx.yaml                         # Project configuration (commit)
├── semantic-layer/
│   └── warehouse/                   # YAML semantic sources (commit)
│       ├── metrics/
│       │   └── revenue.yaml
│       └── entities/
│           └── customer.yaml
├── wiki/
│   ├── global/                      # Shared business context (commit)
│   │   └── metric-definitions.md
│   └── user/
│       └── alice/                   # User-scoped notes (commit)
│           └── analysis-notes.md
├── raw-sources/
│   └── warehouse/                   # Ingest artifacts (commit)
│       ├── tables.json
│       └── join-graph.json
└── .ktx/                            # Local state and secrets (gitignore)
    ├── secrets.yaml
    └── state.json
```

## Code Examples

### TypeScript: Using ktx in a Node.js Agent

```typescript
import { KtxClient } from '@kaelio/ktx';

// Initialize ktx client
const ktx = new KtxClient({
  projectDir: process.cwd()
});

// Search semantic layer
async function findMetric(query: string) {
  const results = await ktx.searchSemanticLayer(query, {
    limit: 5,
    threshold: 0.7
  });
  
  return results.map(r => ({
    name: r.name,
    description: r.description,
    sql: r.sql,
    score: r.score
  }));
}

// Search wiki
async function findBusinessContext(query: string) {
  const results = await ktx.searchWiki(query, {
    limit: 10
  });
  
  return results.map(r => ({
    title: r.title,
    content: r.content,
    source: r.source
  }));
}

// Get metric definition
async function getMetricSQL(metricName: string) {
  const metric = await ktx.getMetric(metricName);
  return metric?.sql;
}

// Execute semantic layer query
async function queryRevenue(startDate: string, endDate: string) {
  return await ktx.query({
    metrics: ['revenue', 'order_count'],
    dimensions: ['date', 'product_category'],
    filters: {
      date: { operator: 'between', values: [startDate, endDate] }
    },
    orderBy: [{ field: 'revenue', direction: 'desc' }],
    limit: 100
  });
}
```

### Python: Using ktx Semantic Layer

```python
from ktx_sl import SemanticLayer, QueryRequest

# Initialize semantic layer
sl = SemanticLayer(project_dir=".")

# Build query plan
query = QueryRequest(
    metrics=["revenue", "profit_margin"],
    dimensions=["customer_segment", "region"],
    filters={
        "order_date": {
            "operator": "between",
            "values": ["2024-01-01", "2024-12-31"]
        }
    }
)

# Generate SQL
sql = sl.plan_query(query)
print(sql.to_sql())

# Execute with connection
results = sl.execute_query(query, connection="warehouse")
```

### YAML: Defining a Semantic Source

```yaml
# semantic-layer/warehouse/metrics/revenue.yaml
version: "1"
type: metric
name: revenue
description: Total revenue from completed orders
sql: SUM(orders.amount)
entity: order
filters:
  - field: orders.status
    operator: eq
    value: completed
joins:
  - entity: customer
    on: orders.customer_id = customer.id
  - entity: product
    on: orders.product_id = product.id
dimensions:
  - order_date
  - customer_segment
  - product_category
```

### YAML: Defining an Entity

```yaml
# semantic-layer/warehouse/entities/customer.yaml
version: "1"
type: entity
name: customer
description: Customer dimension
table: analytics.customers
primary_key: id
columns:
  - name: id
    type: string
  - name: email
    type: string
  - name: segment
    type: string
  - name: created_at
    type: timestamp
```

## Common Patterns

### Setting Up ktx for a New Project

```bash
# 1. Navigate to project directory
cd /path/to/analytics-project

# 2. Run setup wizard
ktx setup

# 3. Verify configuration
ktx status

# 4. Build initial context
ktx ingest

# 5. Test semantic layer search
ktx sl "revenue"

# 6. Test wiki search
ktx wiki "customer"

# 7. Start MCP server for agents
ktx mcp start
```

### Integrating dbt with ktx

```bash
# Add dbt source
ktx source add dbt_main \
  --type dbt \
  --path ./dbt \
  --profiles-dir ~/.dbt

# Ingest dbt metadata
ktx ingest --source dbt_main

# Search for dbt models
ktx sl "customer_lifetime_value"
```

### Agent Workflow: Answering Data Questions

1. **Agent receives question**: "What was revenue by region last quarter?"
2. **Agent searches ktx**: `ktx sl "revenue"`
3. **ktx returns metric definition**: Canonical SQL, joins, filters
4. **Agent constructs query** using approved metric logic
5. **Agent executes query** against warehouse
6. **Agent returns answer** with confidence in accuracy

### Handling Contradictions

```bash
# Ingest multiple sources
ktx ingest

# ktx flags contradictions in output
# Example: "revenue" defined differently in dbt vs Looker

# Review contradictions
ktx contradictions list

# Resolve by editing semantic-layer YAML
vim semantic-layer/warehouse/metrics/revenue.yaml

# Mark as resolved
ktx contradictions resolve revenue --source dbt_main
```

### Updating Context

```bash
# When warehouse schema changes
ktx ingest --connection warehouse --force

# When dbt models are updated
ktx ingest --source dbt_main --force

# When wiki content changes
ktx ingest --source notion_wiki --force

# Full re-ingest of everything
ktx ingest --all --force
```

## MCP Integration

### Claude Desktop Configuration

After `ktx setup`, your Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS) should include:

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
        "/path/to/your/project"
      ]
    }
  }
}
```

### Using ktx in Claude Code

Once MCP server is running, ask Claude:

```text
Use ktx to find the definition of revenue metric
```

```text
Search ktx wiki for information about our refund policy
```

```text
Use ktx to query revenue by region for last quarter
```

### MCP Tools Available

- `ktx_search_semantic_layer`: Search metrics, dimensions, entities
- `ktx_search_wiki`: Search business knowledge and documentation
- `ktx_get_metric`: Get specific metric definition
- `ktx_query`: Execute semantic layer query
- `ktx_list_connections`: List available database connections
- `ktx_get_schema`: Get table schema for a connection

## Troubleshooting

### "ktx project not found"

```bash
# Ensure you're in a directory with ktx.yaml or set project dir
ktx status --project-dir /path/to/project

# Or set environment variable
export KTX_PROJECT_DIR=/path/to/project
```

### "LLM provider not configured"

```bash
# Check current configuration
ktx config get llm.provider

# Set provider and API key
ktx config set llm.provider anthropic
ktx config set llm.api_key $ANTHROPIC_API_KEY

# Verify with status
ktx status
```

### "Database connection failed"

```bash
# Test connection
ktx connection test warehouse

# Check credentials in .ktx/secrets.yaml
cat .ktx/secrets.yaml

# Update password
ktx connection update warehouse --password $NEW_PASSWORD
```

### "No context found" after ingest

```bash
# Check ingest logs
ktx ingest --verbose

# Verify source configuration
cat ktx.yaml

# Force re-ingest
ktx ingest --force

# Check raw-sources directory
ls -la raw-sources/warehouse/
```

### MCP server won't start

```bash
# Check MCP status
ktx mcp status

# Stop existing server
ktx mcp stop

# Start with verbose logging
ktx mcp start --verbose

# Check Claude Desktop logs (macOS)
tail -f ~/Library/Logs/Claude/mcp*.log
```

### Semantic layer query fails

```bash
# Validate semantic layer YAML
ktx sl validate

# Check join graph
cat raw-sources/warehouse/join-graph.json

# Test metric directly
ktx sl "your_metric_name" --json

# Check SQL generation
ktx query --metrics revenue --dimensions date --dry-run
```

### Context is stale

```bash
# Re-ingest specific connection
ktx ingest --connection warehouse --force

# Re-ingest all sources
ktx ingest --all --force

# Check last ingest timestamp
ktx status --verbose
```

### Performance issues

```bash
# Reduce embedding batch size
ktx config set embeddings.batch_size 10

# Skip sampling on large tables
ktx config set context.skip_sampling true

# Limit table introspection
ktx config set context.max_tables 100
```

## Advanced Usage

### Programmatic API

```typescript
import { Ktx } from '@kaelio/ktx';

const ktx = new Ktx({
  projectDir: '/path/to/project',
  llm: {
    provider: 'anthropic',
    apiKey: process.env.ANTHROPIC_API_KEY
  }
});

await ktx.initialize();
await ktx.ingest({ force: true });

const metrics = await ktx.searchSemanticLayer('revenue');
console.log(metrics);
```

### Custom Semantic Sources

Create custom metric definitions in `semantic-layer/<connection>/metrics/`:

```yaml
version: "1"
type: metric
name: net_revenue
description: Revenue minus refunds and discounts
sql: |
  SUM(orders.amount) - 
  COALESCE(SUM(refunds.amount), 0) - 
  COALESCE(SUM(discounts.amount), 0)
entity: order
joins:
  - entity: refund
    on: orders.id = refunds.order_id
    type: left
  - entity: discount
    on: orders.id = discounts.order_id
    type: left
```

### Wiki Management

```bash
# Add global wiki page
echo "# Metric Definitions\n..." > wiki/global/metrics.md

# Add user-specific notes
mkdir -p wiki/user/$USER
echo "# Analysis Notes\n..." > wiki/user/$USER/notes.md

# Ingest wiki changes
ktx ingest --source wiki
```

## Reference

### CLI Commands

- `ktx setup` - Interactive project setup
- `ktx status` - Check project readiness
- `ktx ingest` - Build context from sources
- `ktx sl <query>` - Search semantic layer
- `ktx wiki <query>` - Search wiki
- `ktx query` - Execute semantic layer query
- `ktx config` - Manage configuration
- `ktx connection` - Manage database connections
- `ktx source` - Manage context sources
- `ktx mcp` - Manage MCP server
- `ktx contradictions` - Manage contradictions
- `ktx validate` - Validate semantic layer

### Configuration Options

See full reference at https://docs.kaelio.com/ktx/docs/cli-reference/ktx

## Resources

- **Documentation**: https://docs.kaelio.com/ktx/docs/
- **Quickstart**: https://docs.kaelio.com/ktx/docs/getting-started/quickstart
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Setup**: https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues**: https://github.com/Kaelio/ktx/issues
