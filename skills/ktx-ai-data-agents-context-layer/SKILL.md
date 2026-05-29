---
name: ktx-ai-data-agents-context-layer
description: Expert in ktx - the self-improving context layer that teaches AI agents to query data warehouses accurately with approved metrics, wiki knowledge, and semantic layers
triggers:
  - set up ktx for data agents
  - configure ktx semantic layer
  - integrate ktx with claude code
  - build warehouse context with ktx
  - query data through ktx mcp
  - connect ktx to my database
  - use ktx for analytics agents
  - add ktx wiki knowledge
---

# ktx AI Data Agents Context Layer Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

**ktx** is an executable context layer that enables AI agents (Claude Code, Codex, Cursor, OpenCode) to query data warehouses accurately. It automatically builds and maintains:

- **Semantic layer** with approved metric definitions, joinable columns, and fan/chasm trap resolution
- **Wiki knowledge** from company documentation, dbt, Looker, Metabase, and Notion
- **Database context** through table introspection and usage pattern detection
- **MCP server** for agent integration with CLI and programmatic access

**Key benefits:**
- Agents use canonical SQL instead of inventing metrics each time
- Automatically detects contradictions across knowledge sources
- Read-only by design for warehouse safety
- Runs locally with your own LLM API keys

**Supported databases:** PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite

**Integrations:** dbt, MetricFlow, LookML, Looker, Metabase, Notion

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
ktx --version
```

### Project-Specific Setup

```bash
cd /path/to/your/analytics-project
ktx setup
```

The `ktx setup` wizard will:
1. Create or resume a ktx project
2. Configure LLM and embedding providers
3. Add database connections
4. Configure context sources (dbt, Looker, Metabase, Notion)
5. Build initial context
6. Install agent integration

### Agent Integration (Auto-install via Skills)

From Claude Code, Codex, Cursor, or OpenCode:

```text
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install
and configure ktx in this project.
```

## Project Structure

```text
my-project/
├── ktx.yaml                         # Main configuration
├── semantic-layer/
│   └── warehouse/                   # Semantic sources per connection
│       ├── customers.yaml
│       └── revenue.yaml
├── wiki/
│   ├── global/                      # Shared business context
│   │   ├── refund-policy.md
│   │   └── metric-definitions.md
│   └── user/
│       └── alice/                   # User-scoped notes
├── raw-sources/
│   └── warehouse/                   # Ingest artifacts
│       ├── introspection/
│       └── reports/
└── .ktx/                            # Local state (git-ignored)
    ├── secrets.yaml
    └── embeddings/
```

**Git best practices:**
- Commit: `ktx.yaml`, `semantic-layer/`, `wiki/global/`
- Ignore: `.ktx/`, `wiki/user/`

## Configuration

### ktx.yaml Structure

```yaml
version: 1.0

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key stored in .ktx/secrets.yaml

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key stored in .ktx/secrets.yaml

databases:
  - id: warehouse
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    # Credentials in .ktx/secrets.yaml
    introspection:
      schemas:
        - public
        - analytics
      sample_rows: 100

context_sources:
  - type: dbt
    id: dbt_main
    database_connection: warehouse
    profiles_dir: ~/.dbt
    project_dir: ./dbt
    
  - type: looker
    id: looker_prod
    base_url: https://company.looker.com
    # API credentials in .ktx/secrets.yaml
    
  - type: notion
    id: notion_docs
    # Token in .ktx/secrets.yaml
    pages:
      - Data Team Wiki
      - Metric Definitions

wiki:
  global_dir: ./wiki/global
  user_dir: ./wiki/user

mcp:
  enabled: true
  port: 3000
```

### Secrets Management (.ktx/secrets.yaml)

Never commit this file. Reference environment variables:

```yaml
llm:
  api_key: ${ANTHROPIC_API_KEY}

embeddings:
  api_key: ${OPENAI_API_KEY}

databases:
  warehouse:
    user: ${DB_USER}
    password: ${DB_PASSWORD}

context_sources:
  looker_prod:
    client_id: ${LOOKER_CLIENT_ID}
    client_secret: ${LOOKER_CLIENT_SECRET}
  
  notion_docs:
    token: ${NOTION_TOKEN}
```

## Core Commands

### Project Management

```bash
# Create or resume project
ktx setup

# Check project status
ktx status

# Output example:
# ktx project: /home/user/analytics
# Project ready: yes
# LLM ready: yes (claude-sonnet-4-6)
# Databases configured: yes (warehouse)
# ktx context built: yes
```

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific source
ktx ingest --source dbt_main

# Ingest specific database
ktx ingest --database warehouse

# Force re-ingestion
ktx ingest --force

# Dry run to preview changes
ktx ingest --dry-run
```

### Searching Context

```bash
# Search semantic layer
ktx sl "monthly recurring revenue"
ktx sl "customer churn rate"

# Search wiki
ktx wiki "refund policy"
ktx wiki "how to calculate ltv"

# Combined search (semantic + full-text)
ktx search "revenue by customer segment"
```

### Semantic Layer Management

```bash
# List all semantic sources
ktx sl list

# Show specific source details
ktx sl show customers

# Validate semantic layer
ktx sl validate

# Test metric query
ktx sl query "monthly_revenue" --filters "date >= '2024-01-01'"
```

### MCP Server

```bash
# Start MCP server for agent integration
ktx mcp start

# Start with custom port
ktx mcp start --port 3001

# Start with specific project
ktx mcp start --project-dir /path/to/project

# Check MCP status
ktx mcp status
```

### Wiki Management

```bash
# Add wiki page
ktx wiki add --file ./docs/metrics.md --scope global

# Update existing page
ktx wiki update metric-definitions

# List wiki pages
ktx wiki list

# Search and retrieve
ktx wiki "customer segmentation"
```

## Semantic Layer Patterns

### Defining a Metric Source

`semantic-layer/warehouse/revenue.yaml`:

```yaml
version: 1.0
source_id: monthly_revenue
display_name: Monthly Revenue
description: Total monthly revenue from all orders
type: metric

base_table:
  connection: warehouse
  schema: analytics
  table: orders

measures:
  - name: total_revenue
    expression: SUM(order_total)
    data_type: decimal
    description: Sum of all order totals
    
  - name: average_order_value
    expression: AVG(order_total)
    data_type: decimal
    
dimensions:
  - name: order_month
    expression: DATE_TRUNC('month', order_date)
    data_type: date
    
  - name: customer_segment
    column: segment
    data_type: string

join_keys:
  - name: customer_id
    column: customer_id
    references:
      - source: customers
        column: id

filters:
  - name: status
    expression: status = 'completed'
    default: true
```

### Defining a Dimension Source

`semantic-layer/warehouse/customers.yaml`:

```yaml
version: 1.0
source_id: customers
display_name: Customers
description: Customer dimension table
type: dimension

base_table:
  connection: warehouse
  schema: public
  table: customers

dimensions:
  - name: customer_id
    column: id
    data_type: integer
    primary_key: true
    
  - name: email
    column: email
    data_type: string
    
  - name: signup_date
    column: created_at
    data_type: timestamp
    
  - name: segment
    column: segment
    data_type: string
    values:
      - enterprise
      - smb
      - self_serve

join_keys:
  - name: id
    column: id
```

### Join Graph Resolution

ktx automatically detects and resolves:

**Fan traps** (one-to-many causing count inflation):
```yaml
# ktx adds distinct clauses automatically
# Customer → Orders (1:N)
# Revenue calculation stays accurate
```

**Chasm traps** (multiple paths causing duplicates):
```yaml
# ktx uses subqueries to isolate paths
# Customer → Orders + Customer → Support_Tickets
# Prevents cross-product
```

## Agent Integration Examples

### Using ktx from Claude Code

```typescript
// Claude Code session with ktx MCP integration

// 1. Search for relevant metrics
const revenueMetrics = await mcp.call('ktx_search', {
  query: 'monthly recurring revenue',
  source_types: ['semantic_layer']
});

// 2. Get metric definition
const metric = await mcp.call('ktx_sl_get', {
  source_id: 'monthly_revenue'
});

// 3. Query with filters
const result = await mcp.call('ktx_sl_query', {
  source_id: 'monthly_revenue',
  measures: ['total_revenue'],
  dimensions: ['order_month'],
  filters: {
    order_month: '>= 2024-01-01',
    customer_segment: 'enterprise'
  }
});

// 4. Search wiki for context
const policy = await mcp.call('ktx_wiki_search', {
  query: 'revenue recognition policy'
});
```

### MCP Tools Available to Agents

```typescript
// Tool: ktx_search
{
  name: 'ktx_search',
  description: 'Search across semantic layer and wiki',
  parameters: {
    query: string,
    source_types?: ['semantic_layer', 'wiki'],
    limit?: number
  }
}

// Tool: ktx_sl_list
{
  name: 'ktx_sl_list',
  description: 'List all semantic sources',
  parameters: {
    type?: 'metric' | 'dimension'
  }
}

// Tool: ktx_sl_get
{
  name: 'ktx_sl_get',
  description: 'Get semantic source definition',
  parameters: {
    source_id: string
  }
}

// Tool: ktx_sl_query
{
  name: 'ktx_sl_query',
  description: 'Execute metric query',
  parameters: {
    source_id: string,
    measures: string[],
    dimensions?: string[],
    filters?: Record<string, string>,
    limit?: number
  }
}

// Tool: ktx_wiki_search
{
  name: 'ktx_wiki_search',
  description: 'Search wiki pages',
  parameters: {
    query: string,
    scope?: 'global' | 'user'
  }
}

// Tool: ktx_wiki_get
{
  name: 'ktx_wiki_get',
  description: 'Retrieve wiki page by ID',
  parameters: {
    page_id: string
  }
}
```

## Common Workflows

### Initial Setup Workflow

```bash
# 1. Install globally
npm install -g @kaelio/ktx

# 2. Navigate to project
cd ~/analytics

# 3. Run interactive setup
ktx setup
# Follow prompts to:
#   - Choose LLM provider (Anthropic, Vertex, AI Gateway)
#   - Configure embedding provider (OpenAI, Vertex)
#   - Add database connections
#   - Configure context sources (dbt, Looker, etc.)

# 4. Verify configuration
ktx status

# 5. Build initial context
ktx ingest

# 6. Test search
ktx sl "revenue"
ktx wiki "metrics"
```

### Adding a New Data Source

```bash
# 1. Add connection in ktx.yaml
# databases:
#   - id: prod_warehouse
#     type: snowflake
#     account: xy12345.us-east-1
#     database: ANALYTICS

# 2. Add credentials to .ktx/secrets.yaml
# databases:
#   prod_warehouse:
#     user: ${SNOWFLAKE_USER}
#     password: ${SNOWFLAKE_PASSWORD}

# 3. Ingest new connection
ktx ingest --database prod_warehouse

# 4. Verify introspection
ls raw-sources/prod_warehouse/introspection/
```

### Creating Custom Metrics

```bash
# 1. Create semantic source file
cat > semantic-layer/warehouse/user_retention.yaml << 'EOF'
version: 1.0
source_id: user_retention
display_name: User Retention
type: metric

base_table:
  connection: warehouse
  schema: analytics
  table: user_activity

measures:
  - name: retained_users
    expression: COUNT(DISTINCT user_id)
    description: Users active in both current and previous period

dimensions:
  - name: cohort_month
    expression: DATE_TRUNC('month', first_activity_date)
    data_type: date
    
  - name: period_number
    expression: months_since_first_activity
    data_type: integer

join_keys:
  - name: user_id
    column: user_id
    references:
      - source: users
        column: id
EOF

# 2. Validate
ktx sl validate

# 3. Test query
ktx sl query "user_retention" \
  --measures retained_users \
  --dimensions cohort_month,period_number \
  --filters "cohort_month >= '2024-01-01'"
```

### Updating Wiki Knowledge

```bash
# 1. Create wiki page
mkdir -p wiki/global
cat > wiki/global/revenue-metrics.md << 'EOF'
# Revenue Metrics Guide

## Monthly Recurring Revenue (MRR)

MRR is calculated as the sum of all active subscriptions normalized to monthly value.

**Formula:** `SUM(subscription_value * billing_frequency_multiplier)`

**Exclusions:**
- One-time charges
- Usage-based fees
- Cancelled subscriptions

**Source:** `semantic-layer/warehouse/revenue.yaml`

## Revenue Recognition Policy

Revenue is recognized when:
1. Service is delivered
2. Payment is probable
3. Amount is determinable

See Finance Wiki for full policy.
EOF

# 2. Ingest wiki updates
ktx ingest --source wiki

# 3. Search to verify
ktx wiki "revenue recognition"
```

### Agent Query Pattern

When an agent needs to answer a data question:

```typescript
// Agent receives: "What was our enterprise revenue last quarter?"

// Step 1: Search for relevant semantic sources
const sources = await mcp.call('ktx_search', {
  query: 'enterprise revenue quarterly',
  source_types: ['semantic_layer']
});
// Returns: monthly_revenue source with customer_segment dimension

// Step 2: Get full definition to understand filters and dimensions
const source = await mcp.call('ktx_sl_get', {
  source_id: 'monthly_revenue'
});
// Learns: segment dimension exists, total_revenue measure available

// Step 3: Execute query with appropriate filters
const result = await mcp.call('ktx_sl_query', {
  source_id: 'monthly_revenue',
  measures: ['total_revenue'],
  dimensions: ['order_month'],
  filters: {
    customer_segment: 'enterprise',
    order_month: '>= 2024-10-01 AND order_month < 2025-01-01'
  }
});

// Step 4: Check wiki for business context
const context = await mcp.call('ktx_wiki_search', {
  query: 'enterprise revenue definition'
});
// Validates: revenue recognition policy, segment definition
```

## Troubleshooting

### Project Status Issues

```bash
# Problem: "Project ready: no"
ktx setup  # Re-run setup wizard

# Problem: "LLM ready: no"
# Check .ktx/secrets.yaml has valid API key
export ANTHROPIC_API_KEY=sk-ant-...
ktx setup --reconfigure-llm

# Problem: "ktx context built: no"
ktx ingest  # Build initial context
```

### Database Connection Issues

```bash
# Test connection directly
ktx ingest --database warehouse --dry-run

# Check credentials
cat .ktx/secrets.yaml  # Verify env vars are set

# Enable debug logging
export KTX_LOG_LEVEL=debug
ktx ingest --database warehouse

# Common issues:
# - Firewall blocking connection
# - Invalid credentials in secrets.yaml
# - Schema permissions insufficient (needs SELECT on all tables)
```

### Semantic Layer Validation

```bash
# Validate all sources
ktx sl validate

# Common errors:

# 1. Invalid join reference
# Error: "Source 'orders' references unknown source 'customers'"
# Fix: Create customers.yaml or fix reference

# 2. Circular join
# Error: "Circular join detected: orders -> customers -> orders"
# Fix: Remove redundant join_key

# 3. Invalid expression
# Error: "Measure expression invalid: SUM(invalid_column)"
# Fix: Verify column exists in base_table
```

### MCP Server Issues

```bash
# Problem: Agent can't connect to ktx
ktx mcp status

# If not running:
ktx mcp start

# Problem: "Port already in use"
ktx mcp start --port 3001

# Update agent config with new port
# Claude Desktop: ~/Library/Application Support/Claude/claude_desktop_config.json
# {
#   "mcpServers": {
#     "ktx": {
#       "command": "ktx",
#       "args": ["mcp", "start", "--port", "3001"]
#     }
#   }
# }

# Problem: MCP server crashes
export KTX_LOG_LEVEL=debug
ktx mcp start 2>&1 | tee ktx-mcp.log
```

### Context Ingestion Issues

```bash
# Problem: dbt ingestion fails
ktx ingest --source dbt_main --dry-run

# Check:
# - profiles.yml location correct in ktx.yaml
# - dbt project compiles: cd dbt && dbt compile
# - Database connection in dbt profile matches ktx

# Problem: Looker ingestion fails
# Check:
# - API credentials valid
# - User has "See LookML" permission
# - Base URL includes https://

# Problem: Notion ingestion hangs
# Check:
# - Integration has access to specified pages
# - Page IDs are valid (not titles)
# - Rate limiting (Notion API: 3 req/sec)
```

### Search Quality Issues

```bash
# Problem: Searches return irrelevant results

# 1. Check if embeddings are built
ls .ktx/embeddings/

# 2. Rebuild embeddings
ktx ingest --rebuild-embeddings

# 3. Adjust search parameters in code:
# - Increase top_k for more results
# - Adjust semantic vs lexical weighting
# - Check query phrasing

# Problem: Missing expected sources

# 1. Verify source was ingested
ktx sl list | grep revenue

# 2. Check ingestion logs
ktx ingest --source dbt_main --verbose

# 3. Manually inspect
cat semantic-layer/warehouse/revenue.yaml
```

### Performance Optimization

```bash
# Slow context search
# 1. Check embedding index size
du -sh .ktx/embeddings/

# 2. Reduce sample_rows in ktx.yaml
# databases:
#   - id: warehouse
#     introspection:
#       sample_rows: 50  # Reduce from 100

# 3. Limit schema introspection
# databases:
#   - id: warehouse
#     introspection:
#       schemas:
#         - analytics  # Only scan needed schemas

# Slow ingestion
# 1. Use incremental ingestion
ktx ingest --incremental

# 2. Ingest specific sources
ktx ingest --source dbt_main

# 3. Parallel ingestion (if supported)
ktx ingest --parallel
```

## Environment Variables

```bash
# LLM Provider
export ANTHROPIC_API_KEY=sk-ant-...
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/vertex-key.json
export OPENAI_API_KEY=sk-...  # For AI Gateway

# Embedding Provider
export OPENAI_API_KEY=sk-...

# Database Credentials
export DB_USER=analytics_ro
export DB_PASSWORD=...
export SNOWFLAKE_USER=...
export SNOWFLAKE_PASSWORD=...

# Context Sources
export LOOKER_CLIENT_ID=...
export LOOKER_CLIENT_SECRET=...
export NOTION_TOKEN=secret_...

# ktx Configuration
export KTX_PROJECT_DIR=/path/to/project
export KTX_LOG_LEVEL=info  # debug, info, warn, error
export KTX_TELEMETRY_ENABLED=false  # Opt out of telemetry

# MCP Server
export KTX_MCP_PORT=3000
```

## Advanced Configuration

### Custom LLM Configuration

```yaml
# ktx.yaml
llm:
  provider: vertex
  project: my-gcp-project
  location: us-central1
  model: claude-3-5-sonnet-v2@20241022
  max_tokens: 4096
  temperature: 0.1
```

### Multi-Database Setup

```yaml
# ktx.yaml
databases:
  - id: prod_warehouse
    type: snowflake
    account: prod.us-east-1
    database: ANALYTICS
    introspection:
      schemas: [PUBLIC, ANALYTICS]
      
  - id: staging_warehouse
    type: snowflake
    account: staging.us-east-1
    database: ANALYTICS_STAGING
    introspection:
      schemas: [PUBLIC]
      
  - id: metrics_db
    type: postgres
    host: metrics.internal
    database: metrics
    introspection:
      schemas: [public]
      sample_rows: 50
```

### Custom Wiki Organization

```yaml
# ktx.yaml
wiki:
  global_dir: ./wiki/global
  user_dir: ./wiki/user
  
  # Custom taxonomy
  categories:
    - metrics
    - policies
    - data-models
    - playbooks
  
  # Ingestion filters
  exclude_patterns:
    - "**/*.tmp"
    - "**/draft-*"
```

### Context Source Priorities

```yaml
# ktx.yaml
context_sources:
  - type: dbt
    id: dbt_main
    priority: 1  # Highest priority for conflicts
    
  - type: looker
    id: looker_prod
    priority: 2
    
  - type: metabase
    id: metabase
    priority: 3  # Lowest priority
```

## Best Practices

### Semantic Layer Design

1. **One source per logical entity**: Create separate YAML files for `customers`, `orders`, `revenue`, etc.
2. **Clear naming**: Use `snake_case` for IDs, human-readable `display_name`
3. **Document thoroughly**: Add `description` to every measure and dimension
4. **Define join keys explicitly**: Even if ktx can infer, explicit is better
5. **Use filters for data quality**: Exclude test data, cancelled records, etc.

### Wiki Organization

1. **Global for team knowledge**: Metric definitions, policies, data models
2. **User for personal notes**: Query patterns, analysis notes, WIP docs
3. **Link to sources**: Reference semantic source IDs in wiki pages
4. **Update on changes**: Keep wiki in sync with schema changes

### Agent Integration

1. **Start MCP before agent**: Run `ktx mcp start` before opening Claude Desktop
2. **Use search before query**: Let agents discover sources via search
3. **Provide context in prompts**: "Use ktx to find revenue metrics"
4. **Review generated SQL**: ktx provides canonical queries, but validate results

### Security

1. **Never commit secrets**: Keep `.ktx/` in `.gitignore`
2. **Use read-only DB users**: ktx doesn't need write access
3. **Limit schema access**: Only introspect schemas agents need
4. **Rotate API keys regularly**: Especially for shared LLM providers

## Migration Guides

### From dbt Semantic Layer

```bash
# 1. ktx ingests dbt metrics.yml automatically
# Configure in ktx.yaml:
context_sources:
  - type: dbt
    id: dbt_main
    database_connection: warehouse
    profiles_dir: ~/.dbt
    project_dir: ./dbt

# 2. Run ingestion
ktx ingest --source dbt_main

# 3. Review generated semantic sources
ls semantic-layer/warehouse/
# dbt metrics become ktx metric sources

# 4. Enhance with ktx features
# - Add join_keys for cross-source queries
# - Add wiki pages explaining business logic
# - Define additional dimensions not in dbt
```

### From Looker LookML

```bash
# 1. Configure Looker source
context_sources:
  - type: looker
    id: looker_prod
    base_url: https://company.looker.com

# 2. Add API credentials to secrets
# .ktx/secrets.yaml:
# context_sources:
#   looker_prod:
#     client_id: ${LOOKER_CLIENT_ID}
#     client_secret: ${LOOKER_CLIENT_SECRET}

# 3. Ingest
ktx ingest --source looker_prod

# 4. LookML views → ktx dimension sources
# LookML measures → ktx metric sources
```

## CLI Reference Summary

| Command | Purpose |
|---------|---------|
| `ktx setup` | Initialize or update project |
| `ktx status` | Check project health |
| `ktx ingest` | Build context from sources |
| `ktx sl list` | List semantic sources |
| `ktx sl show <id>` | Show source details |
| `ktx sl validate` | Validate semantic layer |
| `ktx sl query <id>` | Execute metric query |
| `ktx wiki list` | List wiki pages |
| `ktx wiki search <query>` | Search wiki |
| `ktx wiki add --file <path>` | Add wiki page |
| `ktx search <query>` | Search all context |
| `ktx mcp start` | Start MCP server |
| `ktx mcp status` | Check MCP status |
| `ktx config show` | Show configuration |
| `ktx config validate` | Validate config files |

## Resources

- **Documentation**: https://docs.kaelio.com/ktx
- **GitHub**: https://github.com/Kaelio/ktx
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **Issues**: https://github.com/Kaelio/ktx/issues
- **License**: Apache 2.0
