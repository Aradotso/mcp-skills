---
name: ktx-data-agent-context-layer
description: Executable context layer that teaches AI agents to query data warehouses accurately using approved metrics, semantic layer, and business knowledge through MCP
triggers:
  - set up ktx for data analysis
  - configure ktx semantic layer
  - query warehouse using ktx
  - ingest dbt models into ktx
  - search ktx wiki for business context
  - build ktx context from database
  - connect ktx to snowflake/bigquery
  - use ktx with Claude Code
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It ingests approved metric definitions, detects joinable columns, absorbs business knowledge from wikis, and exposes everything through MCP tools so agents can query with semantic understanding instead of re-exploring schemas on every prompt.

## What ktx Does

- **Learns from company knowledge**: Ingests dbt, Looker, Metabase, Notion content and organizes it
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns automatically
- **Builds a semantic layer**: Combines raw tables and metrics through a join graph that resolves fan/chasm traps
- **Serves agents**: Exposes CLI and MCP tools with full-text + semantic search across wiki and semantic sources
- **Read-only by design**: Never writes to your warehouse

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite.

## Installation

```bash
npm install -g @kaelio/ktx
```

Or use npx without installing:

```bash
npx @kaelio/ktx setup
```

For agent-based installation (Claude Code, Codex, Cursor):

```bash
npx skills add Kaelio/ktx --skill ktx
```

## Quick Setup

```bash
# Initialize or resume ktx project
ktx setup

# Check project status
ktx status

# Build context from all configured sources
ktx ingest

# Search semantic layer
ktx sl "monthly revenue"

# Search wiki content
ktx wiki "refund policy"
```

The `ktx setup` command is interactive and will:
1. Create or locate `ktx.yaml` project file
2. Configure LLM provider (Anthropic, Google Vertex, AI Gateway, or Claude Code session)
3. Configure embedding provider (OpenAI, Google, Voyage, Claude Agent SDK)
4. Set up database connections
5. Configure context sources (dbt, Looker, Metabase, Notion)
6. Build initial context
7. Install agent integration (MCP server)

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

Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local.

## Configuration

### Basic ktx.yaml

```yaml
project:
  name: my-analytics-project
  version: 1.0.0

providers:
  llm:
    type: anthropic
    model: claude-sonnet-4-6
    apiKeyEnv: ANTHROPIC_API_KEY
  
  embeddings:
    type: openai
    model: text-embedding-3-small
    apiKeyEnv: OPENAI_API_KEY

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    user: readonly_user
    passwordEnv: WAREHOUSE_PASSWORD

sources:
  dbt_main:
    type: dbt
    connection: warehouse
    projectPath: ../dbt-project
    profilesDir: ~/.dbt
    target: prod
```

### Snowflake Connection

```yaml
connections:
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: SERVICE_ACCOUNT
    passwordEnv: SNOWFLAKE_PASSWORD
    role: READONLY_ROLE
```

### BigQuery Connection

```yaml
connections:
  bigquery_prod:
    type: bigquery
    project: my-gcp-project
    dataset: analytics
    credentialsEnv: GOOGLE_APPLICATION_CREDENTIALS
```

### Multiple Context Sources

```yaml
sources:
  dbt_main:
    type: dbt
    connection: warehouse
    projectPath: ../dbt-project
    profilesDir: ~/.dbt
    target: prod
  
  looker_models:
    type: lookml
    connection: warehouse
    repoPath: ../looker-repo
  
  metabase_cards:
    type: metabase
    connection: warehouse
    url: https://metabase.company.com
    apiKeyEnv: METABASE_API_KEY
  
  notion_wiki:
    type: notion
    apiKeyEnv: NOTION_API_KEY
    databaseIds:
      - a1b2c3d4e5f6
      - f6e5d4c3b2a1
```

## CLI Commands

### Status and Setup

```bash
# Check project readiness
ktx status

# Initialize/update project
ktx setup

# Show project directory location
ktx status --json | jq -r .projectDir
```

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific source
ktx ingest --source dbt_main

# Force full re-ingestion
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, measures)
ktx sl "monthly recurring revenue"

# Search wiki content
ktx wiki "customer refund policy"

# Search with connection filter
ktx sl "orders" --connection warehouse

# JSON output for programmatic use
ktx sl "revenue" --json
```

### MCP Server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with custom project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status
```

### Semantic Layer Operations

```bash
# List all semantic sources
ktx sl list

# Describe a specific metric
ktx sl describe revenue_monthly

# Validate semantic layer definitions
ktx sl validate

# Generate SQL for a metric
ktx sl query "revenue by customer segment"
```

## Agent Integration Patterns

### Using ktx in Claude Code

After running `ktx setup`, Claude Code automatically connects via MCP. Use natural language:

```text
Use ktx to find the definition of monthly recurring revenue
```

```text
Search ktx wiki for our refund policy and use ktx semantic layer to calculate total refunds this quarter
```

```text
Use ktx to list all available metrics related to customer retention
```

### Using ktx in Codex/Cursor

Install the ktx skill:

```bash
npx skills add Kaelio/ktx --skill ktx
```

Then use in chat:

```text
Use ktx to explore our analytics warehouse and find revenue metrics
```

### Programmatic MCP Access (TypeScript)

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

async function searchKtxSemanticLayer(query: string) {
  const transport = new StdioClientTransport({
    command: 'ktx',
    args: ['mcp', 'start', '--project-dir', '/path/to/project']
  });
  
  const client = new Client({
    name: 'my-agent',
    version: '1.0.0'
  }, {
    capabilities: {}
  });
  
  await client.connect(transport);
  
  const result = await client.callTool({
    name: 'ktx_search_semantic_layer',
    arguments: {
      query,
      limit: 10
    }
  });
  
  return result.content;
}

// Usage
const metrics = await searchKtxSemanticLayer('revenue');
console.log(metrics);
```

## Real-World Code Examples

### Building Context from dbt Project

```yaml
# ktx.yaml
sources:
  dbt_analytics:
    type: dbt
    connection: warehouse
    projectPath: ./dbt
    profilesDir: ~/.dbt
    target: prod
```

```bash
# Build context
ktx ingest --source dbt_analytics

# Search for dbt metrics
ktx sl "mrr" --json | jq '.results[0]'
```

### Adding Business Context to Wiki

```bash
# Create wiki page
mkdir -p wiki/global/metrics
cat > wiki/global/metrics/revenue-definitions.md << 'EOF'
# Revenue Definitions

## Monthly Recurring Revenue (MRR)
Sum of all active subscription values normalized to monthly amounts.

**Calculation**: 
- Annual plans: divide by 12
- Monthly plans: use as-is
- Exclude one-time charges

**Data Quality Notes**:
- Exclude customers in "cancelled" status
- Include "trial_converted" status
- Timezone: UTC
EOF

# Ingest wiki
ktx ingest

# Search wiki
ktx wiki "how to calculate MRR"
```

### Custom Semantic Source Definition

```yaml
# semantic-layer/warehouse/revenue_metrics.yaml
version: 1

metrics:
  - name: mrr
    label: Monthly Recurring Revenue
    type: sum
    sql: |
      CASE 
        WHEN billing_period = 'annual' THEN amount / 12
        WHEN billing_period = 'monthly' THEN amount
        ELSE 0
      END
    table: subscriptions
    filters:
      - column: status
        operator: in
        values: [active, trial_converted]
    
  - name: arr
    label: Annual Recurring Revenue
    type: derived
    sql: mrr * 12

dimensions:
  - name: customer_segment
    type: string
    sql: segment
    table: customers
    
  - name: signup_month
    type: time
    sql: DATE_TRUNC('month', created_at)
    table: customers

joins:
  - from: subscriptions
    to: customers
    type: left
    on: subscriptions.customer_id = customers.id
```

```bash
# Validate semantic layer
ktx sl validate

# Query the metric
ktx sl "show me MRR by customer segment"
```

### Querying Multiple Data Sources

```typescript
// query-warehouse.ts
import { execSync } from 'child_process';

function searchKtx(type: 'sl' | 'wiki', query: string) {
  const cmd = `ktx ${type} "${query}" --json`;
  const output = execSync(cmd, { encoding: 'utf-8' });
  return JSON.parse(output);
}

async function analyzeRevenue() {
  // Search for revenue metrics
  const metrics = searchKtx('sl', 'revenue');
  console.log('Found metrics:', metrics.results.map(m => m.name));
  
  // Get business context
  const context = searchKtx('wiki', 'revenue recognition policy');
  console.log('Business context:', context.results[0]?.content);
  
  // Now use this to generate accurate SQL
  const mrrMetric = metrics.results.find(m => m.name === 'mrr');
  if (mrrMetric) {
    console.log('MRR calculation:', mrrMetric.sql);
  }
}

analyzeRevenue();
```

### Integration with Data Pipeline

```typescript
// etl-pipeline.ts
import { spawn } from 'child_process';
import { promisify } from 'util';
import { exec } from 'child_process';

const execAsync = promisify(exec);

async function refreshKtxContext() {
  console.log('Starting ktx context refresh...');
  
  // Ingest latest dbt models
  await execAsync('ktx ingest --source dbt_main --force');
  
  // Ingest Looker updates
  await execAsync('ktx ingest --source looker_models');
  
  // Validate semantic layer
  const { stdout } = await execAsync('ktx sl validate --json');
  const validation = JSON.parse(stdout);
  
  if (validation.errors?.length > 0) {
    console.error('Semantic layer validation failed:', validation.errors);
    throw new Error('Invalid semantic layer');
  }
  
  console.log('ktx context refreshed successfully');
  return validation;
}

// Run after dbt transformations
async function pipeline() {
  // Run dbt
  await execAsync('cd dbt && dbt run');
  
  // Refresh ktx context
  await refreshKtxContext();
  
  console.log('Pipeline complete - agents now have latest context');
}

pipeline();
```

## Common Patterns

### Pattern 1: Development Workflow

```bash
# 1. Set up ktx in your project
cd my-analytics-project
ktx setup

# 2. Build initial context
ktx ingest

# 3. Check what's available
ktx sl list
ktx wiki "search anything"

# 4. Start coding with agent
# Agent now has full context via MCP

# 5. After schema changes, rebuild
ktx ingest --connection warehouse --force
```

### Pattern 2: Multi-Environment Setup

```yaml
# ktx.yaml
connections:
  warehouse_dev:
    type: postgres
    host: dev.db.company.com
    database: analytics_dev
    passwordEnv: WAREHOUSE_DEV_PASSWORD
  
  warehouse_prod:
    type: postgres
    host: prod.db.company.com
    database: analytics_prod
    passwordEnv: WAREHOUSE_PROD_PASSWORD

sources:
  dbt_dev:
    type: dbt
    connection: warehouse_dev
    target: dev
  
  dbt_prod:
    type: dbt
    connection: warehouse_prod
    target: prod
```

```bash
# Ingest dev environment
ktx ingest --connection warehouse_dev

# Ingest prod environment
ktx ingest --connection warehouse_prod

# Search across both
ktx sl "revenue"
```

### Pattern 3: CI/CD Integration

```yaml
# .github/workflows/validate-ktx.yml
name: Validate ktx Context

on:
  pull_request:
    paths:
      - 'semantic-layer/**'
      - 'wiki/**'
      - 'ktx.yaml'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install ktx
        run: npm install -g @kaelio/ktx
      
      - name: Validate semantic layer
        run: ktx sl validate
        
      - name: Check for wiki contradictions
        run: |
          ktx ingest --wiki-only
          # Custom check for contradiction flags
```

### Pattern 4: Metric Discovery for Agents

```typescript
// metric-discovery.ts
import { execSync } from 'child_process';

interface KtxMetric {
  name: string;
  label: string;
  type: string;
  sql: string;
  table: string;
  dimensions?: string[];
}

function discoverMetrics(searchTerm: string): KtxMetric[] {
  const output = execSync(`ktx sl "${searchTerm}" --json`, {
    encoding: 'utf-8'
  });
  
  const results = JSON.parse(output);
  return results.results.filter(r => r.type === 'metric');
}

function getMetricDefinition(metricName: string): KtxMetric | null {
  const output = execSync(`ktx sl describe ${metricName} --json`, {
    encoding: 'utf-8'
  });
  
  return JSON.parse(output);
}

// Agent uses this to discover and use metrics
const revenueMetrics = discoverMetrics('revenue');
console.log('Available revenue metrics:', revenueMetrics.map(m => m.name));

const mrrDef = getMetricDefinition('mrr');
console.log('MRR calculation:', mrrDef?.sql);
```

## Troubleshooting

### Status Check Fails

```bash
# Check detailed status
ktx status --verbose

# Common issues:
# 1. Missing LLM API key
export ANTHROPIC_API_KEY=your-key-here

# 2. Missing embeddings API key
export OPENAI_API_KEY=your-key-here

# 3. Database connection issue
ktx ingest --connection warehouse --verbose
```

### MCP Server Won't Start

```bash
# Check if already running
ktx mcp status

# Kill existing process
pkill -f "ktx mcp"

# Start fresh
ktx mcp start --project-dir $(pwd)

# Check logs
tail -f ~/.ktx/logs/mcp.log
```

### Ingestion Fails

```bash
# Test database connection
ktx ingest --connection warehouse --dry-run

# Check specific source
ktx ingest --source dbt_main --verbose

# Force re-ingestion
ktx ingest --force

# Common issues:
# 1. Database credentials expired
# 2. dbt profiles.yml misconfigured
# 3. Insufficient permissions (ktx needs SELECT only)
```

### Semantic Layer Validation Errors

```bash
# Run validation
ktx sl validate

# Common issues:
# 1. Invalid YAML syntax
# 2. Undefined table references
# 3. Circular join dependencies
# 4. Conflicting metric definitions

# Fix and re-validate
ktx sl validate --fix-auto

# Check specific file
ktx sl validate --file semantic-layer/warehouse/metrics.yaml
```

### Search Returns No Results

```bash
# Check if context is built
ktx status

# Rebuild context
ktx ingest --force

# Test search with wildcard
ktx sl "*"

# Check index health
ktx sl list | wc -l  # Should show count of indexed items
```

### Agent Can't See ktx Tools

```bash
# Verify MCP server is running
ktx mcp status

# Check project directory matches
echo $KTX_PROJECT_DIR

# Reinstall agent integration
ktx setup --agent-only

# For Claude Code, verify in ~/Library/Application\ Support/Claude/claude_desktop_config.json
```

### Performance Issues

```bash
# Large warehouse taking too long to ingest
# Use sampling (default 10k rows per table)
ktx ingest --sample-size 1000

# Skip expensive tables
# Add to ktx.yaml:
connections:
  warehouse:
    exclude:
      - raw_logs
      - event_stream
      - deprecated_*

# Disable embeddings for faster ingestion
ktx ingest --no-embeddings
```

### Wiki Content Not Appearing

```bash
# Check wiki structure
tree wiki/

# Should be:
# wiki/
#   global/
#     *.md
#   user/<username>/
#     *.md

# Ingest wiki only
ktx ingest --wiki-only

# Search with debug
ktx wiki "test" --verbose
```

## Best Practices

1. **Version control semantic layer**: Commit `semantic-layer/` and treat it like code
2. **Keep wiki global**: Put team knowledge in `wiki/global/`, not `wiki/user/`
3. **Use read-only database users**: ktx never writes, but enforce it at DB level
4. **Regular re-ingestion**: Run `ktx ingest` after schema changes or metric updates
5. **Document metric calculations**: Add wiki pages explaining complex metric logic
6. **Test semantic layer**: Use `ktx sl validate` in CI/CD
7. **Environment-specific projects**: Separate ktx projects for dev/staging/prod
8. **Monitor MCP logs**: Check `~/.ktx/logs/` if agents behave unexpectedly

## Environment Variables

```bash
# LLM Provider
export ANTHROPIC_API_KEY=sk-ant-...
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Embeddings
export OPENAI_API_KEY=sk-...
export VOYAGE_API_KEY=pa-...

# Database Connections
export WAREHOUSE_PASSWORD=...
export SNOWFLAKE_PASSWORD=...
export BIGQUERY_CREDENTIALS=/path/to/credentials.json

# Context Sources
export METABASE_API_KEY=...
export NOTION_API_KEY=...
export LOOKER_CLIENT_ID=...
export LOOKER_CLIENT_SECRET=...

# ktx Configuration
export KTX_PROJECT_DIR=/path/to/project
export KTX_LOG_LEVEL=debug
```

## Additional Resources

- [Official Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Setup Guide](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [GitHub Repository](https://github.com/Kaelio/ktx)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
