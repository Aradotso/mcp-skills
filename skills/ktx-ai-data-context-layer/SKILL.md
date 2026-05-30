---
name: ktx-ai-data-context-layer
description: Expert in ktx - the executable context layer for data and analytics agents with skills, memory and semantic layer
triggers:
  - set up ktx for my data warehouse
  - configure ktx semantic layer for my database
  - how do I use ktx with Claude Code
  - help me ingest data sources into ktx
  - configure ktx to read my dbt models
  - troubleshoot ktx MCP server connection
  - search ktx semantic layer and wiki
  - build ktx context from my warehouse
---

# ktx AI Data Context Layer Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What is ktx?

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It automatically:

- **Learns from company knowledge** - ingests wiki content, organizes it, removes duplicates, flags contradictions
- **Maps the data stack** - samples tables, captures metadata, detects joinable columns
- **Builds a semantic layer** - combines raw tables and metrics through a join graph that resolves chasm and fan traps
- **Serves agents at execution** - exposes CLI and MCP tools with combined full-text and semantic search

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, and SQLite. Integrates with dbt, MetricFlow, LookML, Looker, Metabase, and Notion.

## Installation

### Global CLI Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install @kaelio/ktx
```

### Quick Setup

```bash
ktx setup
```

This interactive command:
1. Creates or resumes a local ktx project
2. Configures LLM and embedding providers
3. Sets up database connections
4. Configures context sources (dbt, Looker, etc.)
5. Builds initial context
6. Installs agent integration

## Project Structure

```
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local and git-ignored.

## Core Commands

### Check Project Status

```bash
ktx status
```

Example output:
```
ktx project: /home/user/analytics
Project ready: yes
LLM ready: yes (claude-sonnet-4-6)
Embeddings ready: yes (text-embedding-3-small)
Databases configured: yes (warehouse)
Context sources configured: yes (dbt_main)
ktx context built: yes
Agent integration ready: yes (codex:project)
```

### Build Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific source
ktx ingest --source dbt_main
```

### Search Semantic Layer

```bash
# Search for metrics and dimensions
ktx sl "revenue"

# Search with JSON output
ktx sl "customer lifetime value" --json
```

### Search Wiki

```bash
# Search wiki pages
ktx wiki "refund policy"

# Search with context
ktx wiki "how do we calculate churn"
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

## Configuration

### ktx.yaml Structure

```yaml
version: "1"
project:
  name: "analytics"
  description: "Company analytics warehouse"

llm:
  provider: "anthropic"
  model: "claude-sonnet-4-6"
  apiKeyEnvVar: "ANTHROPIC_API_KEY"

embeddings:
  provider: "openai"
  model: "text-embedding-3-small"
  apiKeyEnvVar: "OPENAI_API_KEY"

connections:
  warehouse:
    type: "postgres"
    host: "localhost"
    port: 5432
    database: "analytics"
    user: "readonly_user"
    passwordEnvVar: "DB_PASSWORD"
    ssl: false

sources:
  dbt_main:
    type: "dbt"
    connection: "warehouse"
    manifestPath: "./target/manifest.json"
    catalogPath: "./target/catalog.json"
```

### Environment Variables

Create a `.env` file in your project root:

```bash
# LLM Provider
ANTHROPIC_API_KEY=your_key_here

# Embeddings Provider
OPENAI_API_KEY=your_key_here

# Database Credentials
DB_PASSWORD=your_db_password_here

# Optional: Project directory override
KTX_PROJECT_DIR=/path/to/project
```

### LLM Provider Configuration

#### Anthropic API

```yaml
llm:
  provider: "anthropic"
  model: "claude-sonnet-4-6"
  apiKeyEnvVar: "ANTHROPIC_API_KEY"
```

#### Google Vertex AI

```yaml
llm:
  provider: "vertex"
  model: "claude-sonnet-4-6"
  projectId: "my-gcp-project"
  region: "us-central1"
  credentialsEnvVar: "GOOGLE_APPLICATION_CREDENTIALS"
```

#### Claude Code Session (Local)

```yaml
llm:
  provider: "claude-agent-sdk"
```

### Database Connection Examples

#### PostgreSQL

```yaml
connections:
  warehouse:
    type: "postgres"
    host: "db.example.com"
    port: 5432
    database: "analytics"
    user: "readonly"
    passwordEnvVar: "POSTGRES_PASSWORD"
    ssl: true
```

#### Snowflake

```yaml
connections:
  snowflake:
    type: "snowflake"
    account: "xy12345.us-east-1"
    warehouse: "COMPUTE_WH"
    database: "ANALYTICS"
    schema: "PUBLIC"
    user: "ktx_user"
    passwordEnvVar: "SNOWFLAKE_PASSWORD"
```

#### BigQuery

```yaml
connections:
  bigquery:
    type: "bigquery"
    projectId: "my-project"
    dataset: "analytics"
    credentialsEnvVar: "GOOGLE_APPLICATION_CREDENTIALS"
```

### Context Source Configuration

#### dbt

```yaml
sources:
  dbt_main:
    type: "dbt"
    connection: "warehouse"
    manifestPath: "./target/manifest.json"
    catalogPath: "./target/catalog.json"
    docsPath: "./target/index.html"  # optional
```

#### Looker

```yaml
sources:
  looker:
    type: "looker"
    connection: "warehouse"
    projectPath: "./looker-models"
```

#### Metabase

```yaml
sources:
  metabase:
    type: "metabase"
    connection: "warehouse"
    apiUrl: "https://metabase.example.com"
    apiKeyEnvVar: "METABASE_API_KEY"
```

#### Notion

```yaml
sources:
  notion_wiki:
    type: "notion"
    apiKeyEnvVar: "NOTION_API_KEY"
    databaseIds:
      - "abc123def456"
      - "789ghi012jkl"
```

## Agent Integration

### Claude Code

After running `ktx setup`, the integration is automatic. From your project directory:

```
What is our total revenue this quarter?
```

Claude Code will use ktx's semantic layer to query accurately.

### Codex

```bash
# Install ktx skill in Codex
npx skills add Kaelio/ktx --skill ktx

# Use in any project with ktx.yaml
```

### Cursor / OpenCode

Configure MCP in your editor settings:

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

## Semantic Layer Usage

### Defining Metrics

Create YAML files in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/revenue.yaml
version: "1"
type: "metric"
name: "total_revenue"
description: "Sum of all order amounts"
sql: "SUM(orders.amount)"
dimensions:
  - "customer_id"
  - "order_date"
filters:
  - "orders.status = 'completed'"
source_table: "orders"
```

### Defining Dimensions

```yaml
# semantic-layer/warehouse/customer_dimension.yaml
version: "1"
type: "dimension"
name: "customer_segment"
description: "Customer segment based on lifetime value"
sql: |
  CASE
    WHEN total_spent > 10000 THEN 'enterprise'
    WHEN total_spent > 1000 THEN 'mid-market'
    ELSE 'smb'
  END
source_table: "customers"
```

### Join Graph

ktx automatically detects joinable columns. You can override in `ktx.yaml`:

```yaml
semantic_layer:
  joins:
    - left_table: "orders"
      right_table: "customers"
      left_column: "customer_id"
      right_column: "id"
      type: "inner"
```

## Wiki Management

### Adding Wiki Pages

```bash
# Add to global wiki
mkdir -p wiki/global
cat > wiki/global/refund-policy.md <<EOF
# Refund Policy

Customers can request refunds within 30 days.
Full refunds issued if:
- Product not as described
- Technical issues unresolved

Partial refunds (50%) if:
- Customer changed mind
- Alternative solution offered
EOF
```

### User-Scoped Notes

```bash
# Add user-specific notes
mkdir -p wiki/user/alice
cat > wiki/user/alice/analysis-notes.md <<EOF
# Q1 Analysis Notes

Revenue spike in March due to new product launch.
Check customer_acquisition_source for details.
EOF
```

### Ingesting Wiki Content

```bash
# Rebuild wiki index
ktx ingest

# Search after ingestion
ktx wiki "refund timeline"
```

## Common Patterns

### Initial Project Setup

```typescript
// scripts/setup-ktx.ts
import { execSync } from 'child_process';
import * as fs from 'fs';
import * as path from 'path';

const projectDir = process.cwd();

// Create ktx.yaml
const config = {
  version: "1",
  project: {
    name: path.basename(projectDir),
    description: "Analytics warehouse"
  },
  llm: {
    provider: "anthropic",
    model: "claude-sonnet-4-6",
    apiKeyEnvVar: "ANTHROPIC_API_KEY"
  },
  embeddings: {
    provider: "openai",
    model: "text-embedding-3-small",
    apiKeyEnvVar: "OPENAI_API_KEY"
  },
  connections: {
    warehouse: {
      type: "postgres",
      host: process.env.DB_HOST || "localhost",
      port: parseInt(process.env.DB_PORT || "5432"),
      database: process.env.DB_NAME || "analytics",
      user: process.env.DB_USER || "readonly",
      passwordEnvVar: "DB_PASSWORD"
    }
  }
};

fs.writeFileSync(
  path.join(projectDir, 'ktx.yaml'),
  JSON.stringify(config, null, 2)
);

// Run setup
execSync('ktx setup', { stdio: 'inherit' });
```

### Programmatic Ingestion

```typescript
// scripts/daily-ingest.ts
import { execSync } from 'child_process';

async function runDailyIngest() {
  console.log('Starting daily ktx ingestion...');
  
  try {
    // Ingest all sources
    execSync('ktx ingest', { 
      stdio: 'inherit',
      env: { ...process.env, KTX_PROJECT_DIR: '/path/to/project' }
    });
    
    console.log('Ingestion complete');
  } catch (error) {
    console.error('Ingestion failed:', error);
    process.exit(1);
  }
}

runDailyIngest();
```

### Custom Metric Definition Workflow

```typescript
// scripts/add-metric.ts
import * as fs from 'fs';
import * as path from 'path';
import * as yaml from 'yaml';

interface MetricDefinition {
  version: string;
  type: 'metric';
  name: string;
  description: string;
  sql: string;
  dimensions?: string[];
  filters?: string[];
  source_table: string;
}

function addMetric(
  connectionId: string,
  metric: Omit<MetricDefinition, 'version' | 'type'>
) {
  const metricDef: MetricDefinition = {
    version: "1",
    type: "metric",
    ...metric
  };
  
  const dir = path.join(
    process.cwd(),
    'semantic-layer',
    connectionId
  );
  
  fs.mkdirSync(dir, { recursive: true });
  
  const filename = `${metric.name}.yaml`;
  const filepath = path.join(dir, filename);
  
  fs.writeFileSync(
    filepath,
    yaml.stringify(metricDef)
  );
  
  console.log(`Created metric: ${filepath}`);
}

// Usage
addMetric('warehouse', {
  name: 'daily_active_users',
  description: 'Count of unique users per day',
  sql: 'COUNT(DISTINCT user_id)',
  dimensions: ['event_date'],
  filters: ['event_type = \'login\''],
  source_table: 'user_events'
});
```

### Searching Programmatically

```typescript
// scripts/search-context.ts
import { execSync } from 'child_process';

function searchSemanticLayer(query: string): any {
  const result = execSync(`ktx sl "${query}" --json`, {
    encoding: 'utf-8',
    env: { ...process.env, KTX_PROJECT_DIR: '/path/to/project' }
  });
  
  return JSON.parse(result);
}

function searchWiki(query: string): any {
  const result = execSync(`ktx wiki "${query}" --json`, {
    encoding: 'utf-8',
    env: { ...process.env, KTX_PROJECT_DIR: '/path/to/project' }
  });
  
  return JSON.parse(result);
}

// Usage
const revenueMetrics = searchSemanticLayer('revenue');
console.log('Revenue metrics:', revenueMetrics);

const policies = searchWiki('refund policy');
console.log('Policies:', policies);
```

## MCP Integration Details

### Available MCP Tools

When ktx MCP server is running, agents have access to:

1. **search_semantic_layer** - Search metrics, dimensions, and tables
2. **search_wiki** - Search wiki pages and documentation
3. **get_metric_definition** - Get full metric SQL and metadata
4. **list_connections** - List available database connections
5. **get_table_schema** - Get table column details
6. **get_join_paths** - Find join paths between tables

### Example MCP Usage from Agent

```typescript
// Agent uses MCP to find revenue metric
const result = await useMcpTool('ktx', 'search_semantic_layer', {
  query: 'total revenue by customer segment'
});

// Get full metric definition
const metricDef = await useMcpTool('ktx', 'get_metric_definition', {
  metric_name: 'total_revenue'
});

// Find join path
const joinPath = await useMcpTool('ktx', 'get_join_paths', {
  from_table: 'orders',
  to_table: 'customers'
});
```

## Troubleshooting

### ktx status shows "Project ready: no"

```bash
# Check ktx.yaml exists
ls -la ktx.yaml

# If missing, run setup
ktx setup

# Verify project directory
echo $KTX_PROJECT_DIR
```

### LLM provider not configured

```bash
# Check environment variables
env | grep ANTHROPIC_API_KEY
env | grep OPENAI_API_KEY

# Add to .env file
echo "ANTHROPIC_API_KEY=your_key" >> .env
echo "OPENAI_API_KEY=your_key" >> .env

# Re-run setup
ktx setup
```

### Database connection fails

```bash
# Test connection manually
psql -h localhost -U readonly -d analytics

# Check ktx.yaml credentials
cat ktx.yaml | grep -A 10 connections

# Verify environment variable
env | grep DB_PASSWORD

# Try with explicit project dir
ktx ingest --project-dir /path/to/project
```

### MCP server won't start

```bash
# Check if already running
ktx mcp status

# Stop existing server
pkill -f "ktx mcp"

# Start with debug output
ktx mcp start --verbose

# Check MCP logs
tail -f ~/.ktx/logs/mcp.log
```

### Context ingestion fails

```bash
# Run with verbose output
ktx ingest --verbose

# Check specific source
ktx ingest --source dbt_main --verbose

# Verify source paths
ls -la target/manifest.json
ls -la target/catalog.json

# Check connection separately
ktx test-connection warehouse
```

### Search returns no results

```bash
# Rebuild context
ktx ingest

# Check if files exist
ls -la semantic-layer/
ls -la wiki/

# Try broader search
ktx sl "revenue" --verbose
ktx wiki "policy" --verbose
```

### Permission errors

```bash
# Check file permissions
ls -la ktx.yaml
ls -la .ktx/

# Fix ownership
chown -R $USER:$USER .ktx/

# Re-initialize
rm -rf .ktx/
ktx setup
```

### Agent can't find ktx

```bash
# Ensure MCP server is running
ktx mcp status

# If not, start it
ktx mcp start --project-dir $(pwd)

# Restart agent client (Claude Code, Cursor, etc.)

# Verify MCP configuration in agent settings
cat ~/.config/claude-code/mcp.json
```

### Telemetry Opt-Out

```bash
# Disable telemetry
export KTX_TELEMETRY_DISABLED=1

# Or add to .env
echo "KTX_TELEMETRY_DISABLED=1" >> .env

# Verify
ktx status
```

## Advanced Usage

### Custom Join Logic

```yaml
# ktx.yaml
semantic_layer:
  joins:
    - left_table: "orders"
      right_table: "customers"
      left_column: "customer_id"
      right_column: "id"
      type: "left"
    - left_table: "orders"
      right_table: "products"
      left_column: "product_id"
      right_column: "id"
      type: "inner"
      # Prevent fan-out
      cardinality: "many_to_one"
```

### Multi-Database Setup

```yaml
connections:
  warehouse:
    type: "postgres"
    host: "warehouse.example.com"
    database: "analytics"
    # ... credentials
  
  production:
    type: "postgres"
    host: "prod.example.com"
    database: "app_db"
    # ... credentials
  
sources:
  dbt_warehouse:
    type: "dbt"
    connection: "warehouse"
    manifestPath: "./warehouse/target/manifest.json"
  
  dbt_production:
    type: "dbt"
    connection: "production"
    manifestPath: "./production/target/manifest.json"
```

### CI/CD Integration

```yaml
# .github/workflows/ktx.yml
name: ktx Context Build

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  build-context:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install ktx
        run: npm install -g @kaelio/ktx
      
      - name: Build context
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: |
          ktx ingest
      
      - name: Commit updated context
        run: |
          git config user.name "ktx Bot"
          git config user.email "bot@example.com"
          git add semantic-layer/ wiki/
          git diff --quiet || git commit -m "Update ktx context"
          git push
```

## Best Practices

1. **Version Control**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/` but git-ignore `.ktx/`
2. **Read-Only Access**: Configure database connections with read-only users
3. **Regular Ingestion**: Run `ktx ingest` daily or on data model changes
4. **Metric Naming**: Use clear, consistent names (e.g., `total_revenue`, not `rev`)
5. **Documentation**: Document business logic in wiki pages, not just metrics
6. **Environment Variables**: Never commit secrets; use env vars for all credentials
7. **Testing**: Test new metrics and joins before committing to version control
8. **MCP Management**: Keep MCP server running for active agent sessions
