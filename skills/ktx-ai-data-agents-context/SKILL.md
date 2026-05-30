---
name: ktx-ai-data-agents-context
description: Executable context layer for AI data agents to query warehouses accurately through MCP with skills and memory
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer for my warehouse
  - help me build context with ktx
  - integrate ktx with claude code for data analysis
  - create ktx semantic sources from dbt
  - query my warehouse using ktx skills
  - troubleshoot ktx context ingestion
  - configure ktx mcp server for agents
---

# ktx AI Data Agents Context Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It combines approved metric definitions, joinable columns, wiki knowledge, and dbt/Looker metadata into one searchable surface for agents like Claude Code, Codex, Cursor, and OpenCode.

## What ktx Does

- **Auto-learns warehouse structure**: Samples tables, detects joinable columns, resolves fan/chasm traps
- **Ingests company knowledge**: Combines dbt, MetricFlow, LookML, Looker, Metabase, and Notion content
- **Builds semantic layer**: Creates reusable metric definitions with automatic join resolution
- **Serves agents via MCP**: Exposes CLI and Model Context Protocol tools for agent execution
- **Flags contradictions**: Identifies conflicts across wiki pages and metric definitions
- **Read-only by design**: Never writes to your warehouse

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-local Installation

```bash
npm install --save-dev @kaelio/ktx
```

## Initial Setup

### Interactive Setup

```bash
ktx setup
```

This command:
1. Creates or resumes a ktx project
2. Configures LLM provider (Anthropic API, Google Vertex, AI Gateway, or Claude Code session)
3. Sets up embeddings provider (OpenAI, Google, or AI Gateway)
4. Connects to databases (PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite)
5. Configures context sources (dbt, MetricFlow, LookML, Looker, Metabase, Notion)
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

**Commit**: `ktx.yaml`, `semantic-layer/`, `wiki/`  
**Ignore**: `.ktx/`

## Configuration

### ktx.yaml Example

```yaml
version: 1
project:
  name: analytics
  description: Company analytics warehouse

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key in .ktx/secrets.yaml or ANTHROPIC_API_KEY env var

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key in OPENAI_API_KEY env var

databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    # Credentials in .ktx/secrets.yaml

contextSources:
  dbt_main:
    type: dbt
    path: ./dbt-project
    target: prod
```

### Secrets Management

Never commit secrets. Store in `.ktx/secrets.yaml` (auto git-ignored) or use environment variables:

```bash
export ANTHROPIC_API_KEY=your_key_here
export OPENAI_API_KEY=your_key_here
export WAREHOUSE_PASSWORD=your_password_here
```

## Key CLI Commands

### Context Building

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific context source
ktx ingest --source dbt_main

# Force re-ingestion (skip cache)
ktx ingest --force
```

### Search Commands

```bash
# Search semantic layer (metrics, dimensions, tables)
ktx sl "revenue"
ktx sl "customer count by region"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "how we calculate churn"

# Limit results
ktx sl "orders" --limit 5
```

### MCP Server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check if MCP server is needed
ktx status
# If output shows: "Run: ktx mcp start --project-dir ..."
# then execute that command before opening your agent
```

### Project Management

```bash
# Initialize new project
ktx init

# Validate configuration
ktx validate

# Show project info
ktx info
```

## Real Usage Examples

### Example 1: Setting Up ktx for a dbt + Snowflake Project

```bash
# Navigate to your dbt project
cd ~/analytics

# Initialize ktx
ktx setup

# During setup, configure:
# - LLM: Anthropic API (claude-sonnet-4-6)
# - Embeddings: OpenAI (text-embedding-3-small)
# - Database: Snowflake (warehouse connection)
# - Context source: dbt (./dbt-project, target: prod)

# Verify setup
ktx status

# Build context
ktx ingest

# Search for a metric
ktx sl "monthly recurring revenue"
```

### Example 2: Creating a Semantic Source (YAML)

**File**: `semantic-layer/warehouse/metrics.yaml`

```yaml
version: 1
type: semantic-source
connection: warehouse

entities:
  - name: orders
    type: table
    sql_table: analytics.orders
    description: All customer orders
    columns:
      - name: order_id
        type: primary_key
      - name: customer_id
        type: foreign_key
        references: customers.customer_id
      - name: order_date
        type: timestamp
      - name: total_amount
        type: number

  - name: customers
    type: table
    sql_table: analytics.customers
    description: Customer dimension
    columns:
      - name: customer_id
        type: primary_key
      - name: email
        type: string
      - name: created_at
        type: timestamp

metrics:
  - name: total_revenue
    type: simple
    sql: "SUM(${orders.total_amount})"
    description: Sum of all order amounts
    
  - name: customer_count
    type: simple
    sql: "COUNT(DISTINCT ${customers.customer_id})"
    description: Total unique customers
    
  - name: average_order_value
    type: derived
    sql: "${total_revenue} / COUNT(DISTINCT ${orders.order_id})"
    description: Average revenue per order
```

After creating/editing semantic sources:

```bash
# Re-ingest to index changes
ktx ingest --connection warehouse
```

### Example 3: Adding Wiki Knowledge

**File**: `wiki/global/refund-policy.md`

```markdown
# Refund Policy

Our refund policy states that customers can request a full refund within 30 days of purchase.

## Business Rules

- Refunds are processed within 5-7 business days
- Partial refunds are not supported
- Refunded orders are marked with `status = 'refunded'` in the orders table

## Related Metrics

- **refund_rate**: `COUNT(refunded_orders) / COUNT(total_orders)`
- Exclude refunded orders from revenue calculations using `WHERE status != 'refunded'`
```

After adding wiki content:

```bash
# Ingest wiki updates
ktx ingest

# Search for the policy
ktx wiki "refund policy"
```

### Example 4: Agent Integration with Claude Code

```bash
# In your project directory
npx skills add Kaelio/ktx --skill ktx

# Or manually add to skills.json:
# {
#   "skills": [
#     {
#       "name": "ktx",
#       "source": "Kaelio/ktx"
#     }
#   ]
# }

# Start MCP server if needed
ktx mcp start

# Now ask Claude Code:
# "What is our total revenue metric defined as?"
# "Show me all joinable columns between orders and customers"
# "Search the wiki for our refund policy"
```

### Example 5: TypeScript API Usage (Programmatic)

```typescript
import { KtxClient } from '@kaelio/ktx';

// Initialize client
const client = new KtxClient({
  projectDir: '/path/to/project',
});

// Search semantic layer
const metrics = await client.searchSemanticLayer('revenue', {
  limit: 10,
  type: 'metric',
});

console.log(metrics);
// [
//   {
//     name: 'total_revenue',
//     type: 'metric',
//     sql: 'SUM(orders.total_amount)',
//     description: 'Sum of all order amounts',
//     score: 0.95
//   },
//   ...
// ]

// Search wiki
const wikiResults = await client.searchWiki('refund policy');

console.log(wikiResults);
// [
//   {
//     title: 'Refund Policy',
//     path: 'wiki/global/refund-policy.md',
//     excerpt: 'Our refund policy states...',
//     score: 0.92
//   },
//   ...
// ]

// Execute a metric query (via semantic layer)
const result = await client.executeMetric('total_revenue', {
  filters: {
    order_date: { gte: '2024-01-01' }
  },
  dimensions: ['region']
});

console.log(result);
// {
//   sql: "SELECT region, SUM(total_amount) FROM ...",
//   rows: [
//     { region: 'US', total_revenue: 1500000 },
//     { region: 'EU', total_revenue: 980000 }
//   ]
// }
```

## Common Patterns

### Pattern 1: dbt Integration

```bash
# Setup assumes dbt project in same directory
ktx setup
# Select "dbt" as context source
# Point to ./dbt-project
# Choose target: prod

# ktx will ingest:
# - models (as semantic tables)
# - metrics.yml (as semantic metrics)
# - docs blocks (as wiki content)
# - column descriptions
```

### Pattern 2: Multi-Database Setup

```yaml
# ktx.yaml
databases:
  warehouse:
    type: snowflake
    account: xy12345
    database: analytics
    schema: public
    
  events:
    type: clickhouse
    host: localhost
    port: 9000
    database: events
```

```bash
# Ingest both
ktx ingest

# Or target specific database
ktx ingest --connection events
```

### Pattern 3: User-scoped Wiki Pages

```bash
# User-specific notes (not shared)
# File: wiki/user/alice/analysis-notes.md

# Global shared knowledge
# File: wiki/global/business-glossary.md

# ktx automatically scopes searches:
ktx wiki "my notes"  # searches user + global
ktx wiki "glossary"  # searches global
```

### Pattern 4: Continuous Context Updates

```bash
# Add to CI/CD or cron job
#!/bin/bash
cd /path/to/analytics
ktx ingest --force

# Or use pre-commit hook:
# .git/hooks/pre-commit
#!/bin/bash
if git diff --cached --name-only | grep -q "^semantic-layer/"; then
  ktx validate
fi
```

## Troubleshooting

### MCP Server Not Starting

**Symptom**: Agent can't find ktx tools

```bash
# Check status
ktx status

# If it says "Run: ktx mcp start --project-dir ...", execute that
ktx mcp start --project-dir /path/to/project

# Verify MCP server is running
ps aux | grep "ktx mcp"
```

### LLM Provider Not Configured

**Symptom**: `ktx ingest` fails with "LLM provider not configured"

```bash
# Check configuration
ktx status

# Reconfigure LLM
ktx setup
# Select "Reconfigure LLM provider"

# Or set environment variable
export ANTHROPIC_API_KEY=sk-ant-...
```

### Embeddings Provider Issues

**Symptom**: Search returns no results or fails

```bash
# Verify embeddings are configured
ktx status

# Reconfigure embeddings
ktx setup
# Select "Reconfigure embeddings provider"

# Or set environment variable
export OPENAI_API_KEY=sk-...
```

### Database Connection Failures

**Symptom**: `ktx ingest` fails to connect to warehouse

```bash
# Test connection
ktx validate --connection warehouse

# Check secrets
cat .ktx/secrets.yaml

# Or use environment variables
export WAREHOUSE_PASSWORD=your_password
export WAREHOUSE_USER=your_user
```

### Ingest Hangs or Takes Too Long

**Symptom**: `ktx ingest` runs for hours

```bash
# Check raw-sources/<connection-id>/ for large tables
ls -lh raw-sources/warehouse/

# Exclude large tables in ktx.yaml
databases:
  warehouse:
    exclude_tables:
      - large_logs_table
      - raw_events

# Re-run ingest
ktx ingest --force
```

### Semantic Layer Not Found

**Symptom**: `ktx sl "metric"` returns no results

```bash
# Check if semantic sources exist
ls -la semantic-layer/

# Create a semantic source (see Example 2 above)
# Then re-ingest
ktx ingest
```

### Wiki Search Returns Nothing

**Symptom**: `ktx wiki "query"` returns empty

```bash
# Check wiki directory
ls -la wiki/global/
ls -la wiki/user/$USER/

# Add markdown files to wiki/global/
# Then re-ingest
ktx ingest
```

### Contradictions Detected

**Symptom**: `ktx ingest` reports contradictions

```text
Contradiction detected:
  - wiki/global/revenue.md defines revenue as SUM(amount)
  - semantic-layer/warehouse/metrics.yaml defines revenue as SUM(total)
  
Action required: Review and resolve conflict
```

**Resolution**:
1. Review both sources
2. Update one to match the other
3. Re-run `ktx ingest`

### Project Directory Not Found

**Symptom**: `ktx: command not found` or `Project not found`

```bash
# Ensure ktx is installed
npm list -g @kaelio/ktx

# Re-install if needed
npm install -g @kaelio/ktx

# Specify project directory explicitly
ktx status --project-dir /path/to/project

# Or set environment variable
export KTX_PROJECT_DIR=/path/to/project
ktx status
```

## Advanced Configuration

### Custom LLM Configuration

```yaml
# ktx.yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  temperature: 0.3
  max_tokens: 4096
  
  # Or use AI Gateway
  # provider: aigateway
  # endpoint: https://gateway.example.com/v1/chat/completions
  # model: claude-sonnet-4-6
```

### Selective Context Ingestion

```yaml
# ktx.yaml
contextSources:
  dbt_main:
    type: dbt
    path: ./dbt-project
    target: prod
    include:
      - models/marts/**
      - models/staging/**
    exclude:
      - models/staging/raw_*
```

### Join Graph Customization

```yaml
# semantic-layer/warehouse/joins.yaml
version: 1
type: join-graph
connection: warehouse

joins:
  - left: orders
    right: customers
    on: orders.customer_id = customers.customer_id
    type: left
    
  - left: orders
    right: products
    on: orders.product_id = products.product_id
    type: left
    relationship: many_to_one  # Prevents fan-out traps
```

## Best Practices

1. **Commit semantic layer and wiki**: Always version control `semantic-layer/` and `wiki/global/`
2. **Use environment variables for secrets**: Never commit `.ktx/secrets.yaml`
3. **Re-ingest after schema changes**: Run `ktx ingest` when warehouse schema evolves
4. **Document metrics in wiki**: Create wiki pages explaining complex metric logic
5. **Validate before commit**: Add `ktx validate` to pre-commit hooks
6. **Start MCP server before agent use**: Always run `ktx mcp start` before opening Claude Code/Codex
7. **Use descriptive entity names**: Name semantic sources after business concepts, not table names

## Resources

- **Docs**: https://docs.kaelio.com/ktx
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Setup**: https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues**: https://github.com/Kaelio/ktx/issues
