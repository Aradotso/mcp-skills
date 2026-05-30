---
name: ktx-ai-data-agents
description: Context layer for data and analytics AI agents with semantic layer, skills, and memory via MCP
triggers:
  - set up ktx for my data warehouse
  - configure ktx semantic layer for analytics
  - add ktx context to my data project
  - connect ktx to my database
  - build data agent context with ktx
  - enable AI agents to query my warehouse accurately
  - install ktx for Claude Code data analysis
  - create ktx semantic sources for my metrics
---

# ktx-ai-data-agents

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is an executable context layer for data and analytics agents. It allows Claude Code, Codex, Cursor, and other AI agents to query data warehouses accurately through MCP by providing skills, memory, and a semantic layer. ktx automatically learns from your company knowledge (wikis, dbt, Looker, Metabase), maps your data stack, detects joinable columns, resolves fan/chasm traps, and serves approved metric definitions to agents.

## Installation

Install ktx globally via npm:

```bash
npm install -g @kaelio/ktx
```

Or use npx for one-off commands:

```bash
npx @kaelio/ktx setup
```

## Quick Start

Initialize a ktx project in your analytics directory:

```bash
ktx setup
```

This interactive command:
- Creates or resumes a local ktx project
- Configures LLM and embedding providers
- Connects to your data warehouse(s)
- Configures context sources (dbt, Looker, Metabase, Notion)
- Builds initial context
- Installs agent integration

Check project status:

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

After running `ktx setup`, your project structure:

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Never commit `.ktx/`.

## Core Commands

### Context Building

Build context for all configured connections:

```bash
ktx ingest
```

Build context for a specific connection:

```bash
ktx ingest --connection-id warehouse
```

Force rebuild all context:

```bash
ktx ingest --force
```

### Searching Context

Search semantic layer sources:

```bash
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn rate"
```

Search wiki knowledge:

```bash
ktx wiki "refund policy"
ktx wiki "customer segmentation"
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

## Configuration

### ktx.yaml Example

```yaml
version: 1
project:
  name: analytics
  description: Company data warehouse and analytics

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key stored in .ktx/secrets.yaml

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key stored in .ktx/secrets.yaml

connections:
  warehouse:
    type: postgres
    description: Main data warehouse
    # Connection details in .ktx/secrets.yaml

context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
    connection_id: warehouse
  
  looker_models:
    type: lookml
    path: ./looker
    connection_id: warehouse
  
  notion_docs:
    type: notion
    # Notion token in .ktx/secrets.yaml
```

### Database Connection Types

ktx supports:
- PostgreSQL
- Snowflake
- BigQuery
- ClickHouse
- MySQL
- SQL Server
- SQLite

Example PostgreSQL connection configuration:

```yaml
connections:
  warehouse:
    type: postgres
    description: Production warehouse
    host: db.example.com
    port: 5432
    database: analytics
    schema: public
    # username and password in .ktx/secrets.yaml
```

### LLM Provider Configuration

Configure Anthropic API:

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
```

Configure Google Vertex AI:

```yaml
llm:
  provider: vertex
  model: claude-sonnet-4-6
  project: my-gcp-project
  location: us-central1
```

Configure AI Gateway:

```yaml
llm:
  provider: ai-gateway
  endpoint: https://gateway.example.com/v1
  model: claude-sonnet-4-6
```

Use Claude Code session (no API key needed):

```yaml
llm:
  provider: claude-agent-sdk
```

API keys are stored in `.ktx/secrets.yaml` (never committed):

```yaml
llm:
  api_key: ${ANTHROPIC_API_KEY}

embeddings:
  api_key: ${OPENAI_API_KEY}

connections:
  warehouse:
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

## Semantic Layer

### Creating Semantic Sources

Semantic sources are YAML files in `semantic-layer/<connection-id>/`. ktx auto-generates these during ingestion but you can create custom sources.

Example metric source (`semantic-layer/warehouse/revenue_metrics.yaml`):

```yaml
version: 1
name: revenue_metrics
description: Core revenue and ARR metrics
connection_id: warehouse

metrics:
  - name: monthly_recurring_revenue
    description: Total MRR from active subscriptions
    type: sum
    sql: |
      SELECT SUM(amount)
      FROM subscriptions
      WHERE status = 'active'
        AND billing_period = 'monthly'
    
  - name: annual_recurring_revenue
    description: ARR (MRR * 12)
    type: derived
    sql: |
      SELECT monthly_recurring_revenue * 12
    dependencies:
      - monthly_recurring_revenue

dimensions:
  - name: subscription_plan
    description: Subscription tier (free, pro, enterprise)
    column: subscriptions.plan_name
    
  - name: customer_segment
    description: Customer business segment
    column: customers.segment
```

Example entity source with join graph:

```yaml
version: 1
name: customer_orders
description: Customer and order entities with relationship
connection_id: warehouse

entities:
  - name: customers
    description: Customer accounts
    table: public.customers
    primary_key: customer_id
    
  - name: orders
    description: Order transactions
    table: public.orders
    primary_key: order_id
    foreign_keys:
      - column: customer_id
        references: customers.customer_id

joins:
  - from: orders
    to: customers
    type: many_to_one
    on: orders.customer_id = customers.customer_id
```

### Querying Semantic Layer

Agents can query the semantic layer via MCP tools or CLI:

```bash
# Search for revenue-related metrics
ktx sl "revenue"

# Get specific metric definition
ktx sl "monthly_recurring_revenue" --exact

# List all metrics in a source
ktx sl --source revenue_metrics
```

## Wiki Management

### Adding Wiki Content

Create global wiki pages in `wiki/global/`:

```bash
mkdir -p wiki/global
cat > wiki/global/refund-policy.md <<EOF
# Refund Policy

Our refund policy allows:
- Full refund within 30 days
- Partial refund (50%) within 90 days
- No refunds after 90 days

Process: Customer contacts support → Support approves → Finance processes
EOF
```

Create user-scoped notes in `wiki/user/<user-id>/`:

```bash
mkdir -p wiki/user/alice
cat > wiki/user/alice/analysis-notes.md <<EOF
# Q1 2026 Revenue Analysis Notes

Key findings:
- Enterprise segment grew 40% QoQ
- Churn increased in SMB segment
- Marketing attribution needs review
EOF
```

### Searching Wiki

```bash
ktx wiki "refund policy"
ktx wiki "customer segmentation"
ktx wiki "analysis notes" --user alice
```

## Agent Integration

### Claude Code / Codex Setup

From your project directory in Claude Code or Codex:

```text
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install
and configure ktx in this project.
```

Or manually add to MCP settings:

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

### Available MCP Tools

Once the MCP server is running, agents have access to:

- `ktx_search_semantic_layer` - Search metrics, dimensions, entities
- `ktx_search_wiki` - Search wiki knowledge
- `ktx_get_metric_definition` - Get canonical metric SQL
- `ktx_get_join_path` - Get join graph between entities
- `ktx_validate_query` - Validate SQL against semantic layer

## Real-World Usage Patterns

### Pattern 1: Agent Queries Approved Metrics

Agent prompt:
```text
What was our MRR last month?
```

Agent workflow (via MCP):
1. `ktx_search_semantic_layer("monthly recurring revenue")`
2. `ktx_get_metric_definition("monthly_recurring_revenue")`
3. Execute canonical SQL against warehouse
4. Return result

### Pattern 2: Agent Joins Entities Correctly

Agent prompt:
```text
Show me average order value by customer segment
```

Agent workflow:
1. `ktx_search_semantic_layer("order value")`
2. `ktx_search_semantic_layer("customer segment")`
3. `ktx_get_join_path("orders", "customers")`
4. Build query using approved join logic
5. Execute and return results

### Pattern 3: Agent Consults Business Knowledge

Agent prompt:
```text
How should I handle refunds in the revenue analysis?
```

Agent workflow:
1. `ktx_search_wiki("refund policy")`
2. Read company refund policy
3. `ktx_search_semantic_layer("revenue")`
4. Combine knowledge to provide accurate answer

## Ingestion Sources

### dbt Integration

Configure dbt source:

```yaml
context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
    connection_id: warehouse
    manifest_path: ./dbt/target/manifest.json  # Optional
```

ktx ingests:
- Model definitions and descriptions
- Metric definitions (dbt metrics)
- Column-level documentation
- Tests and constraints
- Lineage information

### LookML Integration

```yaml
context_sources:
  looker_models:
    type: lookml
    path: ./looker
    connection_id: warehouse
```

ktx ingests:
- Explores and views
- Dimensions and measures
- Join relationships
- Derived tables

### Metabase Integration

```yaml
context_sources:
  metabase_instance:
    type: metabase
    url: https://metabase.example.com
    connection_id: warehouse
    # API key in .ktx/secrets.yaml
```

ktx ingests:
- Question definitions
- Metric definitions
- Dashboard descriptions

### Notion Integration

```yaml
context_sources:
  notion_docs:
    type: notion
    database_id: abc123...
    # Token in .ktx/secrets.yaml
```

ktx ingests:
- Page content and descriptions
- Database properties
- Relationships between pages

## Troubleshooting

### MCP Server Not Starting

Check if MCP server is required:

```bash
ktx status
```

If output includes `ktx mcp start --project-dir ...`, run that command before opening your agent client.

### Connection Issues

Test database connection:

```bash
ktx ingest --connection-id warehouse --dry-run
```

Verify credentials in `.ktx/secrets.yaml`:

```yaml
connections:
  warehouse:
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

Ensure environment variables are set:

```bash
export DB_USERNAME=myuser
export DB_PASSWORD=mypassword
ktx ingest --connection-id warehouse
```

### LLM Provider Errors

Verify API key configuration:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
ktx setup
```

Check LLM configuration in `ktx.yaml`:

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
```

Test LLM connection:

```bash
ktx ingest --force  # Will test LLM during ingestion
```

### Context Not Building

Force rebuild all context:

```bash
ktx ingest --force
```

Check ingestion logs in `raw-sources/<connection-id>/`:

```bash
cat raw-sources/warehouse/ingestion.log
```

Validate source configuration:

```bash
ktx status
```

### Semantic Layer Search Returns Nothing

Ensure context is built:

```bash
ktx ingest
```

Check if semantic sources exist:

```bash
ls -la semantic-layer/warehouse/
```

Search with broader terms:

```bash
ktx sl "revenue"  # Instead of "mrr_monthly_q1"
```

## Advanced Usage

### Custom Semantic Source Templates

Generate template for new source:

```bash
ktx sl generate --name customer_health --connection-id warehouse
```

Edit generated file in `semantic-layer/warehouse/customer_health.yaml`

### Multi-Environment Setup

Create separate ktx projects per environment:

```text
analytics/
├── production/
│   └── ktx.yaml
├── staging/
│   └── ktx.yaml
└── development/
    └── ktx.yaml
```

Switch between environments:

```bash
ktx ingest --project-dir ./production
ktx ingest --project-dir ./staging
```

### CI/CD Integration

Validate semantic layer in CI:

```bash
#!/bin/bash
ktx ingest --dry-run --connection-id warehouse
if [ $? -ne 0 ]; then
  echo "ktx validation failed"
  exit 1
fi
```

### Environment Variables

ktx respects:

- `KTX_PROJECT_DIR` - Default project directory
- `ANTHROPIC_API_KEY` - Anthropic API key
- `OPENAI_API_KEY` - OpenAI API key
- `GOOGLE_APPLICATION_CREDENTIALS` - GCP credentials for Vertex AI
- Database-specific vars (`DB_USERNAME`, `DB_PASSWORD`, etc.)

## Development & Extension

### Local Development Setup

Clone and build ktx:

```bash
git clone https://github.com/kaelio/ktx.git
cd ktx
pnpm install
uv sync --all-groups
pnpm run build
```

Link development CLI:

```bash
pnpm run setup:dev
pnpm run link:dev
ktx-dev --help
```

Run tests:

```bash
pnpm run test
uv run pytest -q
```

### Custom Connectors

ktx connector interface (TypeScript):

```typescript
import { Connector } from '@kaelio/ktx/connectors';

export class CustomConnector extends Connector {
  async connect(): Promise<void> {
    // Establish connection
  }

  async introspect(): Promise<SchemaMetadata> {
    // Return table/column metadata
  }

  async sample(table: string, limit: number): Promise<Row[]> {
    // Return sample rows
  }

  async detectJoins(): Promise<JoinCandidate[]> {
    // Detect foreign key relationships
  }
}
```

Register in `ktx.yaml`:

```yaml
connectors:
  custom:
    module: ./connectors/custom.ts
    type: custom
```

## Read-Only Guarantee

ktx never writes to your data warehouse. All connections are read-only:

- PostgreSQL: Uses role without INSERT/UPDATE/DELETE grants
- Snowflake: Requires SELECT-only role
- BigQuery: Needs `roles/bigquery.dataViewer`
- Other: Similar read-only permissions

Verify connection is read-only:

```bash
ktx ingest --dry-run --connection-id warehouse
```

## Resources

- **Documentation**: https://docs.kaelio.com/ktx
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Quickstart**: https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub**: https://github.com/Kaelio/ktx
