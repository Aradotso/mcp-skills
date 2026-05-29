---
name: ktx-ai-data-agents-context-layer
description: Self-improving context layer that teaches AI agents to query data warehouses accurately with approved metrics, semantic layer, and business knowledge
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer for my warehouse
  - how do I use ktx with Claude Code for analytics
  - integrate ktx context layer with my database
  - build ktx wiki and semantic sources
  - query my data warehouse through ktx MCP
  - ingest my dbt project into ktx
  - search ktx semantic layer for metrics
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**ktx** is an executable context layer that enables AI agents (Claude Code, Codex, Cursor, OpenCode) to query data warehouses accurately. It automatically:

- Learns from company knowledge (wikis, Notion, dbt, Looker, Metabase)
- Maps your data stack (detects joinable columns, samples tables, captures metadata)
- Builds a semantic layer with approved metric definitions
- Resolves chasm and fan traps automatically
- Exposes CLI and MCP tools for agent execution

ktx runs **locally** with your own LLM API keys. No data leaves your machine except to your configured LLM provider.

## Installation

Install globally via npm:

```bash
npm install -g @kaelio/ktx
```

Or use directly with npx:

```bash
npx @kaelio/ktx --help
```

### Initial Setup

Run interactive setup to configure your project:

```bash
ktx setup
```

This will:
1. Create or resume a ktx project in the current directory
2. Configure LLM provider (Anthropic, Google Vertex AI, AI Gateway, or Claude Agent SDK)
3. Set up embedding provider
4. Connect to your database(s)
5. Configure context sources (dbt, MetricFlow, LookML, Looker, Metabase, Notion)
6. Build initial context
7. Install agent integration (Codex, Cursor, Claude Code)

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

## Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration (commit)
├── semantic-layer/<connection-id>/  # YAML semantic sources (commit)
├── wiki/global/                     # Shared business context (commit)
├── wiki/user/<user-id>/             # User-scoped notes (commit)
├── raw-sources/<connection-id>/     # Ingest artifacts and reports (commit)
└── .ktx/                            # Local state and secrets (git-ignore)
```

## Configuration

### ktx.yaml Structure

```yaml
version: '1.0'
project:
  name: my-analytics-project
  id: proj_abc123

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key stored in .ktx/secrets.json

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key stored in .ktx/secrets.json

databases:
  warehouse:
    type: postgres
    host: db.example.com
    port: 5432
    database: analytics
    username: readonly_user
    # password stored in .ktx/secrets.json
    ssl: true
    readOnly: true

contextSources:
  dbt_main:
    type: dbt
    projectPath: ../dbt-project
    profilesDir: ~/.dbt
    target: prod
    enabled: true

  notion_wiki:
    type: notion
    enabled: true
    # token stored in .ktx/secrets.json
```

### Supported Database Types

- PostgreSQL
- Snowflake
- BigQuery
- ClickHouse
- MySQL
- SQL Server
- SQLite

### Supported Context Sources

- **dbt**: Ingests models, metrics, and documentation
- **MetricFlow**: Imports semantic models
- **LookML**: Parses views and explores
- **Looker**: Fetches instance metadata
- **Metabase**: Extracts questions and dashboards
- **Notion**: Syncs pages and databases

## Key Commands

### Building Context

Ingest all configured sources:

```bash
ktx ingest
```

Ingest specific connection:

```bash
ktx ingest --connection-id warehouse
```

Force re-ingestion:

```bash
ktx ingest --force
```

### Searching Context

Search semantic layer for metrics and entities:

```bash
ktx sl "monthly revenue"
ktx sl "customer churn rate"
```

Search wiki for business knowledge:

```bash
ktx wiki "refund policy"
ktx wiki "how we calculate ARR"
```

List all semantic sources:

```bash
ktx sl list
```

Show specific semantic source details:

```bash
ktx sl show orders
```

### MCP Server

Start the MCP server for agent integration:

```bash
ktx mcp start
```

Start with specific project directory:

```bash
ktx mcp start --project-dir /path/to/project
```

The MCP server exposes tools for agents to:
- Search semantic layer
- Search wiki
- Query semantic sources
- List available entities
- Get join paths between entities

### Project Management

Initialize new project:

```bash
ktx init
```

Add database connection:

```bash
ktx config add-database
```

Add context source:

```bash
ktx config add-source
```

Validate configuration:

```bash
ktx config validate
```

## Agent Integration

### Using with Claude Code

1. Install ktx globally or in your project
2. Run `ktx setup` and select "claudecode" as agent integration
3. Start Claude Code from your project directory
4. ktx tools are automatically available

Ask Claude Code:
```text
Search the ktx semantic layer for revenue metrics and show me how to query them
```

### Using with Codex

Install as a skill:

```bash
npx skills add Kaelio/ktx --skill ktx
```

Or via prompt:
```text
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install
and configure ktx in this project.
```

### Using with Cursor

1. Run `ktx setup` and select "cursor" as agent integration
2. ktx is added to your project's MCP configuration
3. Restart Cursor
4. Use ktx tools via the MCP integration

## TypeScript API Usage

### Programmatic Context Search

```typescript
import { KtxProject } from '@kaelio/ktx';

async function searchContext() {
  const project = await KtxProject.load('/path/to/project');
  
  // Search semantic layer
  const semanticResults = await project.searchSemanticLayer('revenue');
  console.log('Semantic sources:', semanticResults);
  
  // Search wiki
  const wikiResults = await project.searchWiki('refund policy');
  console.log('Wiki pages:', wikiResults);
  
  // Get semantic source details
  const source = await project.getSemanticSource('orders');
  console.log('Orders entity:', source);
}
```

### Query Planning

```typescript
import { QueryPlanner } from '@kaelio/ktx/semantic-layer';

async function planQuery() {
  const planner = new QueryPlanner(project);
  
  const plan = await planner.plan({
    metrics: ['revenue', 'order_count'],
    dimensions: ['customer_region', 'order_date'],
    filters: {
      order_date: { gte: '2024-01-01' }
    }
  });
  
  console.log('SQL:', plan.sql);
  console.log('Join path:', plan.joinPath);
}
```

### Custom Ingestion

```typescript
import { Ingestor, DbtConnector } from '@kaelio/ktx/context';

async function customIngest() {
  const project = await KtxProject.load();
  const ingestor = new Ingestor(project);
  
  // Ingest dbt project
  const dbtConnector = new DbtConnector({
    projectPath: '../dbt-project',
    profilesDir: '~/.dbt',
    target: 'prod'
  });
  
  const result = await ingestor.ingest('dbt_main', dbtConnector);
  console.log('Ingested sources:', result.sources.length);
  console.log('Wiki pages created:', result.wikiPages.length);
}
```

## Semantic Layer YAML Format

ktx stores semantic sources as YAML files in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/orders.yaml
name: orders
description: Customer orders with line items
type: entity
database: warehouse
schema: public
table: orders

columns:
  - name: order_id
    type: integer
    primaryKey: true
    description: Unique order identifier

  - name: customer_id
    type: integer
    foreignKey:
      references: customers
      column: customer_id
    description: Reference to customer

  - name: order_date
    type: date
    description: Date order was placed

  - name: status
    type: string
    description: Order status (pending, completed, cancelled)

metrics:
  - name: order_count
    description: Total number of orders
    type: count
    sql: COUNT(DISTINCT order_id)

  - name: total_revenue
    description: Sum of order amounts
    type: sum
    sql: SUM(amount)
    format: currency

dimensions:
  - name: order_month
    type: time
    sql: DATE_TRUNC('month', order_date)
    granularity: month

  - name: order_status
    type: categorical
    sql: status
```

## Wiki Format

Wiki pages are stored as markdown in `wiki/global/` or `wiki/user/<user-id>/`:

```markdown
# Revenue Calculation

Updated: 2024-01-15
Tags: metrics, finance

## Definition

Revenue is calculated as the sum of all completed order amounts, excluding:
- Refunded orders
- Test transactions
- Internal orders

## SQL Implementation

```sql
SELECT SUM(amount)
FROM orders
WHERE status = 'completed'
  AND is_test = false
  AND is_internal = false
```

## Related Metrics

- `arr`: Annual Recurring Revenue
- `mrr`: Monthly Recurring Revenue
- `net_revenue`: Revenue minus refunds
```

## Common Patterns

### Setting Up for a New Warehouse

```bash
# Initialize project
ktx init

# Add database connection interactively
ktx config add-database

# Or configure via ktx.yaml
cat >> ktx.yaml <<EOF
databases:
  warehouse:
    type: snowflake
    account: xy12345
    database: ANALYTICS
    warehouse: COMPUTE_WH
    username: readonly_user
    role: ANALYST
EOF

# Store password securely
export SNOWFLAKE_PASSWORD=your_password_here

# Run ingestion
ktx ingest --connection-id warehouse
```

### Integrating dbt Project

```bash
# Add dbt as context source
ktx config add-source

# Select dbt from the menu, provide:
# - Project path: ../dbt-project
# - Profiles dir: ~/.dbt
# - Target: prod

# Ingest dbt models and metrics
ktx ingest --connection-id dbt_main

# View ingested metrics
ktx sl list --source dbt
```

### Creating Custom Wiki Pages

```bash
# Create global wiki page
cat > wiki/global/customer-definitions.md <<'EOF'
# Customer Definitions

## Active Customer

A customer with at least one order in the last 90 days.

## Churned Customer

A previously active customer with no orders in 90+ days.

## Implementation

Use the `customer_status` dimension in the customers semantic source.
EOF

# Rebuild wiki index
ktx ingest --wiki-only
```

### Querying via MCP Tools

When an agent has ktx MCP integration:

```typescript
// Agent uses these tools automatically

// Search for revenue metrics
const results = await useMcpTool('ktx_search_semantic_layer', {
  query: 'revenue',
  limit: 5
});

// Get semantic source details
const source = await useMcpTool('ktx_get_semantic_source', {
  sourceName: 'orders'
});

// Search wiki for context
const wikiPages = await useMcpTool('ktx_search_wiki', {
  query: 'how to calculate churn',
  limit: 3
});
```

## Troubleshooting

### "ktx mcp start required" in status

Start the MCP server before opening your agent client:

```bash
ktx mcp start --project-dir /path/to/project
```

Keep this running in a terminal while using the agent.

### Agent can't find ktx tools

1. Verify MCP server is running: `ps aux | grep "ktx mcp"`
2. Check agent MCP config (location varies by agent):
   - Codex: `.codex/mcp.json`
   - Claude Code: Auto-configured
   - Cursor: `.cursor/mcp.json`
3. Restart agent after running `ktx setup`

### Database connection fails

```bash
# Test connection manually
ktx config test-database --connection-id warehouse

# Common fixes:
# - Verify credentials in .ktx/secrets.json
# - Check firewall/network access
# - Ensure read-only user has necessary grants
# - For BigQuery, verify service account JSON path
```

### LLM provider errors

```bash
# Verify API key
cat .ktx/secrets.json | grep apiKey

# Test LLM connection
ktx config test-llm

# Switch providers
ktx setup --reconfigure-llm
```

### Ingestion fails or produces no results

```bash
# Check logs
ktx ingest --verbose

# Verify source configuration
ktx config validate

# For dbt: ensure dbt compile works
cd ../dbt-project
dbt compile --target prod

# For Notion: verify token has read permissions
# For Looker: check API credentials
```

### Semantic layer queries fail

```bash
# Validate semantic source YAML
ktx sl validate --source-name orders

# Check for missing join paths
ktx sl graph --from orders --to customers

# Rebuild context
ktx ingest --force
```

### Context search returns irrelevant results

```bash
# Rebuild embeddings
ktx ingest --rebuild-embeddings

# Check embedding provider configuration
ktx config show embeddings

# For wiki: review and consolidate duplicate pages
# For semantic layer: improve column descriptions
```

## Environment Variables

```bash
# Override project directory
export KTX_PROJECT_DIR=/path/to/project

# LLM provider API keys (alternatively stored in .ktx/secrets.json)
export ANTHROPIC_API_KEY=sk-ant-...
export OPENAI_API_KEY=sk-...
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Database credentials
export SNOWFLAKE_PASSWORD=...
export POSTGRES_PASSWORD=...
export BIGQUERY_SERVICE_ACCOUNT_JSON=/path/to/key.json

# Notion integration
export NOTION_API_TOKEN=secret_...

# Disable telemetry
export KTX_TELEMETRY_DISABLED=1
```

## Best Practices

1. **Commit semantic-layer and wiki**: These are your source of truth
2. **Git-ignore .ktx/**: Contains secrets and local state
3. **Use read-only database credentials**: ktx never writes to your warehouse
4. **Run `ktx ingest` regularly**: Keep context fresh as schemas change
5. **Document metrics in wiki**: Complement semantic layer with business logic
6. **Review ingestion reports**: Check `raw-sources/<connection>/report.json`
7. **Keep ktx.yaml minimal**: Store secrets in `.ktx/secrets.json`
8. **Test queries in CLI first**: Use `ktx sl` before having agents generate SQL

## Additional Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
