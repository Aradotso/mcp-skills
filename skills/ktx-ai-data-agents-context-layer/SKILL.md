---
name: ktx-ai-data-agents-context-layer
description: Install, configure, and use ktx to build an executable context layer for AI agents to query data warehouses accurately with semantic layer, wiki, and MCP integration
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer for Claude
  - install ktx to query my warehouse
  - build context with ktx for analytics
  - integrate ktx with my data stack
  - use ktx to teach agents about my database
  - connect ktx to my dbt project
  - configure ktx MCP server for agent access
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What ktx Does

ktx is an executable context layer that teaches AI agents how to accurately query data warehouses. It automatically:

- **Builds semantic context** from your database schema, sampling tables and detecting joinable columns
- **Ingests company knowledge** from dbt, LookML, Metabase, Notion, and other sources
- **Creates a semantic layer** with approved metric definitions and automatic join resolution (fan/chasm traps)
- **Exposes MCP tools** for agents like Claude Code, Codex, and Cursor to search and query your data

Key benefits: agents stop inventing metric logic, reuse canonical SQL definitions, and query with full business context.

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite. Integrates with dbt, MetricFlow, LookML, Looker, Metabase, Notion.

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
cd your-project
npm install --save-dev @kaelio/ktx
```

### Verify Installation

```bash
ktx --version
ktx --help
```

## Initial Setup

### Interactive Setup

Run the guided setup wizard:

```bash
ktx setup
```

This will:
1. Create a `ktx.yaml` configuration file
2. Configure LLM provider (Anthropic, Google Vertex AI, or Claude Code session)
3. Configure embedding provider (OpenAI, Google, or local)
4. Set up database connections
5. Configure context sources (dbt, Looker, Metabase, Notion)
6. Run initial ingestion
7. Install agent integration

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

### Project Structure

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Important**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Add `.ktx/` to `.gitignore`.

### Basic ktx.yaml Example

```yaml
project_id: my-analytics-project
version: 1

llm:
  provider: anthropic
  model: claude-sonnet-4-20250514
  # API key stored in .ktx/secrets.yaml or ANTHROPIC_API_KEY env var

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
    # Credentials in .ktx/secrets.yaml or env vars

context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./target/manifest.json
    catalog_path: ./target/catalog.json
```

### Database Connection Configuration

#### PostgreSQL

```yaml
databases:
  warehouse:
    type: postgres
    host: ${PGHOST}
    port: 5432
    database: ${PGDATABASE}
    user: ${PGUSER}
    password: ${PGPASSWORD}
    ssl: true
    read_only: true  # ktx connections are always read-only
```

#### Snowflake

```yaml
databases:
  snowflake_prod:
    type: snowflake
    account: ${SNOWFLAKE_ACCOUNT}
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    user: ${SNOWFLAKE_USER}
    password: ${SNOWFLAKE_PASSWORD}
    role: ANALYST
```

#### BigQuery

```yaml
databases:
  bigquery_prod:
    type: bigquery
    project_id: ${GCP_PROJECT_ID}
    dataset: analytics
    credentials_path: ${GOOGLE_APPLICATION_CREDENTIALS}
```

### Context Source Configuration

#### dbt Integration

```yaml
context_sources:
  dbt_main:
    type: dbt
    manifest_path: ./dbt/target/manifest.json
    catalog_path: ./dbt/target/catalog.json
    project_dir: ./dbt
    profiles_dir: ~/.dbt
```

#### LookML Integration

```yaml
context_sources:
  lookml_main:
    type: lookml
    project_path: ./lookml-project
    model_files:
      - views/*.view.lkml
      - models/*.model.lkml
```

#### Metabase Integration

```yaml
context_sources:
  metabase:
    type: metabase
    url: https://metabase.company.com
    api_key: ${METABASE_API_KEY}
    database_id: 1
```

#### Notion Integration

```yaml
context_sources:
  notion_docs:
    type: notion
    api_key: ${NOTION_API_KEY}
    database_id: ${NOTION_DATABASE_ID}
    # Or specific pages
    page_ids:
      - abc123
      - def456
```

### LLM Provider Configuration

#### Anthropic API

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-20250514
  api_key: ${ANTHROPIC_API_KEY}
  max_tokens: 8192
```

#### Google Vertex AI

```yaml
llm:
  provider: google
  model: claude-sonnet-4-6@20250514
  project_id: ${GCP_PROJECT_ID}
  location: us-central1
  credentials_path: ${GOOGLE_APPLICATION_CREDENTIALS}
```

#### Claude Code Session (Local)

```yaml
llm:
  provider: claude-session
  # Automatically uses the local Claude Code session
```

## Core Commands

### Building Context

#### Full Ingestion

Build context from all configured sources:

```bash
ktx ingest
```

#### Ingest Specific Connection

```bash
ktx ingest --connection warehouse
```

#### Ingest Specific Source

```bash
ktx ingest --source dbt_main
```

#### Force Re-ingestion

```bash
ktx ingest --force
```

### Searching Context

#### Search Semantic Layer

```bash
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn rate"
```

Example output:
```text
Found 3 semantic sources matching "revenue":

1. monthly_revenue (metric)
   Description: Total revenue by month
   SQL: SELECT date_trunc('month', order_date) as month, SUM(amount) as revenue
   FROM orders GROUP BY 1

2. arr (metric)
   Description: Annual Recurring Revenue
   Tags: finance, subscription
   ...
```

#### Search Wiki

```bash
ktx wiki "refund policy"
ktx wiki "data quality standards"
ktx wiki "metric definitions"
```

#### Combined Search

```bash
ktx search "customer lifetime value"
```

### Managing Wiki Content

#### Add Wiki Page

```bash
ktx wiki add --title "Metric Definitions" --content "Our standard metrics..."
ktx wiki add --file ./docs/data-guide.md
```

#### Edit Wiki Page

```bash
ktx wiki edit <page-id>
```

#### List Wiki Pages

```bash
ktx wiki list
ktx wiki list --scope global
ktx wiki list --scope user
```

### MCP Server

#### Start MCP Server

```bash
ktx mcp start
```

For a specific project:

```bash
ktx mcp start --project-dir /path/to/project
```

#### Check MCP Status

```bash
ktx mcp status
```

#### Stop MCP Server

```bash
ktx mcp stop
```

### Project Management

#### Initialize New Project

```bash
ktx init
```

#### Update Existing Project

```bash
ktx setup
```

#### Validate Configuration

```bash
ktx validate
```

#### Export Context

```bash
ktx export --format json --output context.json
```

## Real-World Examples

### Example 1: Setting Up ktx with PostgreSQL and dbt

```bash
# Navigate to your analytics project
cd ~/projects/analytics

# Install ktx globally
npm install -g @kaelio/ktx

# Run interactive setup
ktx setup

# When prompted, configure:
# - LLM: Anthropic Claude Sonnet 4
# - Embeddings: OpenAI text-embedding-3-small
# - Database: PostgreSQL (localhost:5432)
# - Context source: dbt (./target/manifest.json)

# Verify setup
ktx status

# Build initial context
ktx ingest

# Search for metrics
ktx sl "monthly revenue"

# Start MCP server for agent access
ktx mcp start
```

### Example 2: TypeScript Agent Integration

```typescript
import { MCPClient } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

async function queryWithKtx() {
  // Connect to ktx MCP server
  const transport = new StdioClientTransport({
    command: 'ktx',
    args: ['mcp', 'start', '--project-dir', '/path/to/project']
  });

  const client = new MCPClient({
    name: 'analytics-agent',
    version: '1.0.0'
  }, {
    capabilities: {
      tools: {}
    }
  });

  await client.connect(transport);

  // Search semantic layer
  const searchResult = await client.callTool({
    name: 'ktx_search_semantic_layer',
    arguments: {
      query: 'monthly active users',
      limit: 5
    }
  });

  console.log('Semantic sources:', searchResult);

  // Search wiki
  const wikiResult = await client.callTool({
    name: 'ktx_search_wiki',
    arguments: {
      query: 'metric calculation rules',
      scope: 'global'
    }
  });

  console.log('Wiki pages:', wikiResult);

  await client.close();
}
```

### Example 3: Python Data Analysis with ktx Context

```python
import subprocess
import json

def get_metric_definition(metric_name: str) -> dict:
    """Fetch metric definition from ktx semantic layer."""
    result = subprocess.run(
        ['ktx', 'sl', metric_name, '--format', 'json'],
        capture_output=True,
        text=True
    )
    return json.loads(result.stdout)

def query_with_context(question: str) -> str:
    """Query database using ktx context."""
    # Get relevant semantic sources
    search_result = subprocess.run(
        ['ktx', 'search', question, '--format', 'json'],
        capture_output=True,
        text=True
    )
    
    context = json.loads(search_result.stdout)
    
    # Build query using context
    # ... your query logic here
    
    return context

# Example usage
revenue_def = get_metric_definition('monthly_revenue')
print(f"Revenue metric SQL: {revenue_def['sql']}")

context = query_with_context('What is our customer retention rate?')
```

### Example 4: Programmatic Configuration

```typescript
import { writeFile } from 'fs/promises';
import { dump } from 'js-yaml';

interface KtxConfig {
  project_id: string;
  version: number;
  llm: {
    provider: string;
    model: string;
  };
  databases: Record<string, any>;
  context_sources: Record<string, any>;
}

async function createKtxConfig(projectDir: string) {
  const config: KtxConfig = {
    project_id: 'analytics-prod',
    version: 1,
    llm: {
      provider: 'anthropic',
      model: 'claude-sonnet-4-20250514'
    },
    databases: {
      warehouse: {
        type: 'postgres',
        host: process.env.PGHOST,
        port: 5432,
        database: process.env.PGDATABASE,
        read_only: true
      }
    },
    context_sources: {
      dbt_main: {
        type: 'dbt',
        manifest_path: './target/manifest.json',
        catalog_path: './target/catalog.json'
      }
    }
  };

  const yamlContent = dump(config);
  await writeFile(`${projectDir}/ktx.yaml`, yamlContent);
  
  console.log('ktx.yaml created successfully');
}
```

### Example 5: CI/CD Integration

```bash
#!/bin/bash
# .github/workflows/ktx-ingest.sh

set -e

# Install ktx
npm install -g @kaelio/ktx

# Set project directory
export KTX_PROJECT_DIR=/workspace/analytics

# Validate configuration
ktx validate

# Run ingestion
ktx ingest --force

# Export context for artifacts
ktx export --format json --output ktx-context.json

# Verify semantic layer
ktx sl "revenue" --format json | jq '.[] | .name'
```

## Common Patterns

### Pattern 1: Daily Context Refresh

```bash
#!/bin/bash
# scripts/refresh-ktx-context.sh

cd /path/to/analytics

# Pull latest dbt changes
git pull origin main

# Rebuild dbt artifacts
dbt compile
dbt docs generate

# Ingest updated context
ktx ingest --source dbt_main

# Restart MCP server
ktx mcp stop
ktx mcp start
```

### Pattern 2: Multi-Warehouse Setup

```yaml
# ktx.yaml for multiple warehouses
databases:
  prod_warehouse:
    type: snowflake
    account: ${SNOWFLAKE_PROD_ACCOUNT}
    warehouse: PROD_WH
    database: ANALYTICS
  
  dev_warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: dev_analytics

context_sources:
  dbt_prod:
    type: dbt
    manifest_path: ./prod/target/manifest.json
    database: prod_warehouse
  
  dbt_dev:
    type: dbt
    manifest_path: ./dev/target/manifest.json
    database: dev_warehouse
```

```bash
# Ingest only production
ktx ingest --connection prod_warehouse

# Search across all sources
ktx search "customer metrics"
```

### Pattern 3: Team Wiki Synchronization

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

async function syncNotionToWiki() {
  // Ingest from Notion
  await execAsync('ktx ingest --source notion_docs');
  
  // List new wiki pages
  const { stdout } = await execAsync('ktx wiki list --format json');
  const pages = JSON.parse(stdout);
  
  console.log(`Synced ${pages.length} wiki pages from Notion`);
  
  // Search for conflicts
  const conflicts = await execAsync('ktx wiki conflicts');
  if (conflicts.stdout) {
    console.warn('Found conflicting definitions:', conflicts.stdout);
  }
}
```

### Pattern 4: Semantic Layer Validation

```bash
#!/bin/bash
# Validate semantic layer after changes

# Check for undefined metrics
ktx sl --list | while read metric; do
  result=$(ktx sl "$metric" --format json 2>&1)
  if echo "$result" | grep -q "error"; then
    echo "ERROR: Metric $metric has issues"
    echo "$result"
  fi
done

# Check for orphaned sources
ktx validate --strict
```

## Troubleshooting

### Issue: "ktx mcp start" command not found

**Solution**: Install MCP server dependencies:

```bash
npm install -g @modelcontextprotocol/sdk
ktx setup --repair
```

### Issue: Database connection fails

**Symptoms**: `ktx ingest` errors with connection timeout or authentication failure

**Solutions**:

1. Verify credentials in `.ktx/secrets.yaml`:
```bash
cat .ktx/secrets.yaml
```

2. Test connection manually:
```bash
ktx validate --connection warehouse
```

3. Check environment variables:
```bash
echo $PGHOST $PGUSER $PGDATABASE
```

4. Ensure read-only user has necessary permissions:
```sql
-- PostgreSQL
GRANT CONNECT ON DATABASE analytics TO ktx_user;
GRANT USAGE ON SCHEMA public TO ktx_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ktx_user;
```

### Issue: Ingestion is slow or hangs

**Solutions**:

1. Limit table sampling:
```yaml
databases:
  warehouse:
    sampling:
      max_tables: 100
      max_rows_per_table: 1000
```

2. Skip large tables:
```yaml
databases:
  warehouse:
    exclude_tables:
      - raw_logs
      - events_archive
```

3. Run ingestion incrementally:
```bash
ktx ingest --incremental
```

### Issue: Agent can't find ktx MCP server

**Symptoms**: Agent returns "No MCP server found" or "ktx tools not available"

**Solutions**:

1. Verify MCP server is running:
```bash
ktx mcp status
```

2. Check agent configuration. For Claude Code, ensure `ktx mcp start` is in project instructions:
```text
Before queries, ensure ktx MCP server is running:
ktx mcp start --project-dir $(pwd)
```

3. Restart agent client after starting MCP server

### Issue: Semantic layer returns no results

**Solutions**:

1. Check if context is built:
```bash
ktx status
```

2. Rebuild context:
```bash
ktx ingest --force
```

3. Verify semantic sources exist:
```bash
ls -la semantic-layer/
```

4. Search with broader query:
```bash
ktx sl "" --limit 50
```

### Issue: Wiki conflicts detected

**Symptoms**: `ktx ingest` warns about contradictory definitions

**Solution**: Review conflicts and resolve manually:

```bash
# List conflicts
ktx wiki conflicts

# View specific conflict
ktx wiki show <conflict-id>

# Resolve by editing wiki page
ktx wiki edit <page-id>

# Or mark as resolved
ktx wiki resolve <conflict-id>
```

### Issue: Out of memory during ingestion

**Solutions**:

1. Increase Node.js heap size:
```bash
export NODE_OPTIONS="--max-old-space-size=4096"
ktx ingest
```

2. Process connections one at a time:
```bash
ktx ingest --connection warehouse
ktx ingest --connection analytics_db
```

3. Reduce sampling:
```yaml
databases:
  warehouse:
    sampling:
      max_rows_per_table: 100
```

### Issue: LLM API rate limits

**Symptoms**: Ingestion fails with 429 errors

**Solutions**:

1. Add rate limiting configuration:
```yaml
llm:
  provider: anthropic
  rate_limit:
    requests_per_minute: 50
    retry_attempts: 3
    retry_delay: 1000  # ms
```

2. Use different model tier:
```yaml
llm:
  model: claude-haiku-4-20250514  # Faster, cheaper
```

### Issue: ktx.yaml not found

**Symptoms**: Commands fail with "No ktx project found"

**Solutions**:

1. Specify project directory explicitly:
```bash
ktx status --project-dir /path/to/project
```

2. Set environment variable:
```bash
export KTX_PROJECT_DIR=/path/to/project
ktx status
```

3. Initialize project if missing:
```bash
ktx init
```

## Best Practices

1. **Commit semantic layer and wiki**: Always version control `ktx.yaml`, `semantic-layer/`, and `wiki/global/`

2. **Keep .ktx/ local**: Add to `.gitignore` to avoid committing secrets

3. **Use environment variables**: Never hardcode API keys in `ktx.yaml`

4. **Regular ingestion**: Set up cron/CI to run `ktx ingest` after dbt runs

5. **Start MCP before agent**: Ensure `ktx mcp start` runs before opening agent client

6. **Validate before commit**: Run `ktx validate` in CI to catch configuration errors

7. **Scope wiki appropriately**: Use `wiki/global/` for team knowledge, `wiki/user/` for personal notes

8. **Monitor context size**: Large semantic layers slow search; use `exclude_tables` to prune
