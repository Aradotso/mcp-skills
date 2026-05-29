---
name: ktx-ai-data-agents-context-layer
description: Executable context layer for data and analytics agents that enables accurate querying through MCP with skills and memory
triggers:
  - set up ktx for data agents
  - configure ktx context layer
  - help me install and use ktx
  - create a ktx semantic layer
  - integrate ktx with Claude Code
  - build data agent context with ktx
  - query warehouse using ktx
  - configure ktx for my data warehouse
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It automatically ingests warehouse metadata, wiki content, dbt models, and BI tool definitions to build a unified semantic layer with approved metrics, joinable columns, and business knowledge.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content, organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns
- **Builds semantic layer**: Combines raw tables and metrics through a join graph that resolves chasm/fan traps
- **Serves agents**: Exposes CLI and MCP tools with full-text and semantic search

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite. Integrates with dbt, MetricFlow, LookML, Looker, Metabase, Notion.

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install @kaelio/ktx
```

### Initial Setup

```bash
ktx setup
```

This interactive command:
- Creates or resumes a ktx project
- Configures LLM providers (Anthropic, Vertex AI, AI Gateway)
- Configures database connections
- Sets up context sources (dbt, Looker, Metabase, Notion)
- Builds initial context
- Installs agent integration

## Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local.

## Configuration

### ktx.yaml Example

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

databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    # Credentials stored in .ktx/secrets.yaml

context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
  
  looker_main:
    type: looker
    base_url: https://company.looker.com
    
  notion_wiki:
    type: notion
    # Token stored in .ktx/secrets.yaml
```

### Environment Variables

```bash
# LLM Provider
export ANTHROPIC_API_KEY=your-key-here
export OPENAI_API_KEY=your-key-here

# Project directory (optional)
export KTX_PROJECT_DIR=/path/to/project
```

## Key Commands

### Project Management

```bash
# Check project status
ktx status

# Create or update project
ktx setup

# Validate configuration
ktx config validate

# Show current configuration
ktx config show
```

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Force re-ingest
ktx ingest --force

# Ingest with custom sampling
ktx ingest --sample-size 1000
```

### Searching Context

```bash
# Search semantic layer
ktx sl "revenue"
ktx sl "customer churn rate"

# Search wiki
ktx wiki "refund policy"
ktx wiki "how to calculate ARR"

# Combined search
ktx search "monthly recurring revenue"
```

### MCP Server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with specific project
ktx mcp start --project-dir /path/to/project

# Check MCP status
ktx mcp status
```

### Semantic Layer Operations

```bash
# List all semantic sources
ktx sl list

# Show source details
ktx sl show customers

# Validate semantic layer
ktx sl validate

# Query using semantic layer
ktx sl query "SELECT revenue FROM sales WHERE date >= '2024-01-01'"
```

## TypeScript/JavaScript Usage

### Programmatic API

```typescript
import { KtxContext, KtxConfig } from '@kaelio/ktx';

// Initialize context
const config = await KtxConfig.load('/path/to/project');
const context = new KtxContext(config);

// Search semantic layer
const results = await context.searchSemanticLayer('revenue');
console.log(results);

// Search wiki
const wikiResults = await context.searchWiki('refund policy');
console.log(wikiResults);

// Get connection info
const connection = config.getDatabase('warehouse');
console.log(connection);
```

### MCP Integration

```typescript
import { KtxMcpServer } from '@kaelio/ktx/mcp';

const server = new KtxMcpServer({
  projectDir: process.env.KTX_PROJECT_DIR || process.cwd()
});

await server.start();
```

## Agent Integration Patterns

### Claude Code / Codex Integration

ktx automatically configures MCP integration during `ktx setup`. Verify with:

```bash
ktx status
```

If output shows `ktx mcp start --project-dir ...`, run that command before opening your agent client.

### Using ktx as a Skill

When using ktx from an AI agent:

```text
Use the ktx skill to search for the definition of "monthly recurring revenue" 
and write a SQL query to calculate it for the last 6 months.
```

The agent can:
- Search semantic layer for metric definitions
- Search wiki for business context
- Query using approved metric logic
- Validate queries against semantic layer

### Example Agent Workflow

```typescript
// Agent discovers metric definition
const metric = await ktx.searchSemanticLayer('monthly_recurring_revenue');

// Agent finds related business context
const context = await ktx.searchWiki('MRR calculation rules');

// Agent generates query using approved definition
const query = `
  SELECT 
    date_trunc('month', subscription_start_date) as month,
    SUM(${metric.sql_expression}) as mrr
  FROM ${metric.source_table}
  WHERE subscription_start_date >= CURRENT_DATE - INTERVAL '6 months'
  GROUP BY 1
  ORDER BY 1
`;

// Agent validates query
await ktx.validateQuery(query);
```

## Semantic Layer Definition

### Creating Metrics

```yaml
# semantic-layer/warehouse/metrics.yaml
version: 1
metrics:
  - name: monthly_recurring_revenue
    description: Monthly recurring revenue from active subscriptions
    type: sum
    sql: |
      CASE 
        WHEN subscription_status = 'active' 
        THEN monthly_amount 
        ELSE 0 
      END
    dimensions:
      - date_month
      - customer_id
      - plan_type
    filters:
      - subscription_status IN ('active', 'trialing')
    
  - name: customer_lifetime_value
    description: Total revenue from customer over their lifetime
    type: sum
    sql: total_revenue
    dimensions:
      - customer_id
      - cohort_month
```

### Defining Dimensions

```yaml
# semantic-layer/warehouse/dimensions.yaml
version: 1
dimensions:
  - name: date_month
    type: time
    sql: date_trunc('month', created_at)
    grain: month
    
  - name: customer_id
    type: categorical
    sql: customer_id
    primary_key: true
    
  - name: plan_type
    type: categorical
    sql: subscription_plan_type
    values:
      - starter
      - professional
      - enterprise
```

### Join Graph

```yaml
# semantic-layer/warehouse/joins.yaml
version: 1
joins:
  - left: customers
    right: subscriptions
    type: left
    on:
      - customers.id = subscriptions.customer_id
    relationship: one_to_many
    
  - left: subscriptions
    right: invoices
    type: left
    on:
      - subscriptions.id = invoices.subscription_id
    relationship: one_to_many
    
  - left: customers
    right: events
    type: left
    on:
      - customers.id = events.customer_id
    relationship: one_to_many
    fan_trap_resolution: aggregate_first
```

## Wiki Pages

### Creating Wiki Content

```markdown
<!-- wiki/global/revenue-definitions.md -->
# Revenue Definitions

## Monthly Recurring Revenue (MRR)

MRR is the normalized monthly revenue from active subscriptions.

**Calculation**: Sum of monthly_amount for all active subscriptions.

**Exclusions**:
- One-time charges
- Cancelled subscriptions
- Trial subscriptions (unless explicitly included)

**Related Metrics**:
- Annual Recurring Revenue (ARR) = MRR × 12
- Net MRR = New MRR + Expansion - Contraction - Churn
```

### User-Scoped Notes

```markdown
<!-- wiki/user/john/analysis-notes.md -->
# Q4 Revenue Analysis Notes

Found discrepancy in MRR calculation:
- Dashboard shows $1.2M
- SQL query shows $1.18M
- Issue: Dashboard includes trialing subscriptions

Resolution: Updated semantic layer to exclude trials by default.
```

## Database Connection Examples

### PostgreSQL

```yaml
databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    ssl: true
```

### Snowflake

```yaml
databases:
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    role: ANALYST
```

### BigQuery

```yaml
databases:
  bigquery_prod:
    type: bigquery
    project_id: my-project
    dataset: analytics
    location: US
```

## Context Source Integration

### dbt Integration

```yaml
context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
    profiles_dir: ~/.dbt
    target: prod
```

ktx automatically:
- Parses dbt models and metrics
- Extracts documentation
- Imports relationships and tests
- Syncs to semantic layer

### Looker Integration

```yaml
context_sources:
  looker_prod:
    type: looker
    base_url: https://company.looker.com
    verify_ssl: true
```

Imports:
- LookML models and explores
- Dimensions and measures
- Join relationships
- User-defined metrics

### Metabase Integration

```yaml
context_sources:
  metabase:
    type: metabase
    base_url: https://metabase.company.com
    database_id: 1
```

### Notion Integration

```yaml
context_sources:
  notion_wiki:
    type: notion
    space_id: 12345
    include_archived: false
```

## Common Patterns

### Initializing a New Project

```bash
# Create project directory
mkdir my-analytics && cd my-analytics

# Initialize git
git init
echo ".ktx/" >> .gitignore

# Setup ktx
ktx setup

# Verify setup
ktx status

# Build initial context
ktx ingest

# Start MCP server for agents
ktx mcp start
```

### Adding a New Database

```bash
# Edit ktx.yaml to add database config
# Then update secrets
ktx config edit

# Test connection
ktx ingest --connection new_warehouse --dry-run

# Ingest
ktx ingest --connection new_warehouse
```

### Searching Before Writing Queries

```typescript
// Agent workflow: Search first
const revenueMetrics = await ktx.searchSemanticLayer('revenue');
const revenueRules = await ktx.searchWiki('revenue recognition');

// Use discovered context to build accurate query
const query = buildQuery(revenueMetrics[0], revenueRules);
```

### Validating Semantic Layer Changes

```bash
# Make changes to semantic-layer/*.yaml
# Validate before committing
ktx sl validate

# If valid, re-ingest
ktx ingest --force

# Test search
ktx sl "your new metric"
```

### Sharing Project with Team

```bash
# Commit configuration and context
git add ktx.yaml semantic-layer/ wiki/
git commit -m "Add ktx context layer"
git push

# Team members clone and setup
git clone <repo>
cd <repo>

# Configure their own credentials
ktx setup
# Choose "resume existing project"
# Enter their LLM API keys
# Enter their database credentials

# They're ready
ktx status
ktx mcp start
```

## Troubleshooting

### "Project not found"

```bash
# Check current directory has ktx.yaml
ls ktx.yaml

# Or specify project directory
ktx --project-dir /path/to/project status

# Or set environment variable
export KTX_PROJECT_DIR=/path/to/project
ktx status
```

### "Database connection failed"

```bash
# Verify connection config
ktx config show

# Test with dry-run
ktx ingest --connection warehouse --dry-run

# Check credentials in .ktx/secrets.yaml
cat .ktx/secrets.yaml

# Re-run setup to update credentials
ktx setup
```

### "LLM provider error"

```bash
# Verify API key is set
echo $ANTHROPIC_API_KEY

# Or check .ktx/secrets.yaml
cat .ktx/secrets.yaml

# Update provider
ktx config edit
# Change llm.provider and llm.model

# Test with simple search
ktx sl "test"
```

### "Semantic layer validation failed"

```bash
# Show validation errors
ktx sl validate

# Common issues:
# - Invalid YAML syntax
# - Reference to non-existent table/column
# - Circular join dependencies

# Fix errors, then revalidate
ktx sl validate
```

### "MCP server won't start"

```bash
# Check if already running
ktx mcp status

# Kill existing process
pkill -f "ktx mcp"

# Start fresh
ktx mcp start

# Check logs
tail -f ~/.ktx/logs/mcp.log
```

### "Context not updating after changes"

```bash
# Force re-ingest
ktx ingest --force

# Clear cache
rm -rf .ktx/cache/

# Rebuild
ktx ingest
```

### "Search returns no results"

```bash
# Check if context is built
ktx status

# Rebuild embeddings
ktx ingest --rebuild-embeddings

# Try different search terms
ktx sl "revenue"  # Try singular
ktx sl "revenues" # Try plural
ktx sl "sales"    # Try synonym
```

## Best Practices

1. **Commit semantic layer and wiki**: These are your team's shared knowledge
2. **Keep .ktx/ local**: Contains secrets and user-specific state
3. **Validate before committing**: Run `ktx sl validate` before pushing changes
4. **Document metrics thoroughly**: Include business logic in description fields
5. **Use wiki for context**: Document why metrics are calculated a certain way
6. **Ingest regularly**: Schedule `ktx ingest` to keep context fresh
7. **Test agent queries**: Validate that agents use approved definitions

## Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
