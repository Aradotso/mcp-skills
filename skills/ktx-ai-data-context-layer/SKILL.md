---
name: ktx-ai-data-context-layer
description: Build and maintain an executable context layer for data and analytics agents using ktx's semantic layer, wiki knowledge, and MCP integration
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer for warehouse
  - ingest database schema into ktx context
  - search ktx wiki or semantic layer
  - connect Claude Code to warehouse with ktx
  - build ktx context from dbt and database
  - configure ktx MCP server for agents
  - troubleshoot ktx ingestion or setup
---

# ktx AI Data Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It combines approved metric definitions, joinable columns, wiki content, and business knowledge into a single searchable surface that agents can access via MCP (Model Context Protocol) or CLI.

## What ktx Does

- **Learns from company knowledge**: Ingests wiki content (Notion, Markdown), organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, captures metadata and usage patterns, detects joinable columns
- **Builds a semantic layer**: Combines raw tables and high-level metrics through a join graph that resolves fan and chasm traps
- **Serves agents at execution**: Exposes CLI and MCP tools with full-text and semantic search across wiki and semantic-layer entities

## Installation

Install globally via npm:

```bash
npm install -g @kaelio/ktx
```

Or use in a specific project:

```bash
npm install @kaelio/ktx
```

Verify installation:

```bash
ktx --version
```

## Initial Setup

Run the interactive setup wizard:

```bash
ktx setup
```

This will:
1. Create or resume a local ktx project
2. Configure LLM and embedding providers
3. Set up database connections
4. Configure context sources (dbt, Looker, Metabase, Notion)
5. Build initial context
6. Install agent integration

Check project status:

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

Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local.

## Configuration

### ktx.yaml Structure

```yaml
version: 1
project_name: analytics

# LLM configuration
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  api_key_env: ANTHROPIC_API_KEY

# Embedding configuration
embeddings:
  provider: openai
  model: text-embedding-3-small
  api_key_env: OPENAI_API_KEY

# Database connections
connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    user_env: POSTGRES_USER
    password_env: POSTGRES_PASSWORD

# Context sources
context_sources:
  - id: dbt_main
    type: dbt
    path: ./dbt_project
  - id: notion_wiki
    type: notion
    api_key_env: NOTION_API_KEY
    database_ids:
      - abc123def456
```

### LLM Provider Options

```bash
# Anthropic API
ktx config set llm.provider anthropic
ktx config set llm.api_key_env ANTHROPIC_API_KEY

# Google Vertex AI
ktx config set llm.provider vertex
ktx config set llm.project_id my-gcp-project

# AI Gateway
ktx config set llm.provider ai-gateway
ktx config set llm.gateway_url https://gateway.example.com
```

### Database Connections

Add a database connection:

```bash
ktx connection add warehouse \
  --type postgres \
  --host localhost \
  --port 5432 \
  --database analytics \
  --user-env POSTGRES_USER \
  --password-env POSTGRES_PASSWORD
```

Supported databases: PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite

Test a connection:

```bash
ktx connection test warehouse
```

## Building Context

### Ingest All Sources

Run ingestion for all configured connections and context sources:

```bash
ktx ingest
```

### Ingest Specific Connection

```bash
ktx ingest --connection warehouse
```

### Ingest with Options

```bash
# Sample more tables (default 100)
ktx ingest --sample-size 500

# Skip expensive operations
ktx ingest --skip-column-profiling

# Force re-ingest even if unchanged
ktx ingest --force
```

### What Ingestion Does

1. **Database scanning**: Samples tables, profiles columns, detects data types, patterns, and constraints
2. **Join detection**: Identifies foreign key relationships and candidate join columns via value overlap
3. **Metadata extraction**: Pulls table comments, column descriptions, and usage stats
4. **Context source parsing**: Reads dbt models, LookML views, Metabase questions, Notion pages
5. **Deduplication & validation**: Removes duplicate content, flags contradictions
6. **Embedding generation**: Creates vector embeddings for semantic search

## CLI Commands

### Search Commands

Search semantic layer (metrics, dimensions, entities):

```bash
ktx sl "revenue"
ktx sl "monthly active users"
```

Search wiki content:

```bash
ktx wiki "refund policy"
ktx wiki "how to calculate churn"
```

### Context Management

View semantic sources:

```bash
ktx semantic list
```

Show specific source details:

```bash
ktx semantic show users
```

Edit a semantic source:

```bash
ktx semantic edit users
```

### Wiki Management

Create a wiki page:

```bash
ktx wiki create --title "Metric Definitions" --content-file metrics.md
```

List wiki pages:

```bash
ktx wiki list
```

Edit a wiki page:

```bash
ktx wiki edit "Metric Definitions"
```

### MCP Server

Start the MCP server for agent clients:

```bash
ktx mcp start
```

Start with specific project directory:

```bash
ktx mcp start --project-dir /path/to/analytics
```

## MCP Integration

### Agent Configuration

For **Claude Code**, **Codex**, **Cursor**, or **OpenCode**:

1. Install ktx skill:

```bash
npx skills add Kaelio/ktx --skill ktx
```

2. Start MCP server (if not auto-started):

```bash
ktx mcp start --project-dir /path/to/project
```

3. Ask your agent to use ktx tools:

```text
Use ktx to search for revenue metrics and show me the SQL definition
```

### MCP Tools Available

- `search_semantic_layer`: Search metrics, dimensions, entities
- `search_wiki`: Search wiki pages and business context
- `get_table_schema`: Get column details and sample data
- `get_metric_definition`: Get canonical SQL for a metric
- `detect_joins`: Find joinable paths between tables
- `validate_query`: Check a SQL query against schema and metrics

## Code Examples

### TypeScript: Using ktx CLI Programmatically

```typescript
import { spawn } from 'child_process';

async function searchSemanticLayer(query: string): Promise<string> {
  return new Promise((resolve, reject) => {
    const ktx = spawn('ktx', ['sl', query]);
    let output = '';
    
    ktx.stdout.on('data', (data) => {
      output += data.toString();
    });
    
    ktx.on('close', (code) => {
      if (code === 0) {
        resolve(output);
      } else {
        reject(new Error(`ktx exited with code ${code}`));
      }
    });
  });
}

// Usage
const revenueMetrics = await searchSemanticLayer('revenue');
console.log(revenueMetrics);
```

### TypeScript: MCP Client Integration

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

async function initKtxMcpClient() {
  const transport = new StdioClientTransport({
    command: 'ktx',
    args: ['mcp', 'start', '--project-dir', process.cwd()]
  });
  
  const client = new Client({
    name: 'my-data-agent',
    version: '1.0.0'
  }, {
    capabilities: {}
  });
  
  await client.connect(transport);
  return client;
}

async function searchMetrics(client: Client, query: string) {
  const result = await client.callTool({
    name: 'search_semantic_layer',
    arguments: {
      query,
      limit: 10
    }
  });
  
  return result;
}

// Usage
const client = await initKtxMcpClient();
const metrics = await searchMetrics(client, 'monthly revenue');
console.log(metrics);
```

### Python: Querying Semantic Layer

```python
import subprocess
import json

def search_semantic_layer(query: str) -> dict:
    """Search ktx semantic layer via CLI."""
    result = subprocess.run(
        ['ktx', 'sl', query, '--json'],
        capture_output=True,
        text=True,
        check=True
    )
    return json.loads(result.stdout)

def get_metric_sql(metric_name: str) -> str:
    """Get canonical SQL for a metric."""
    result = subprocess.run(
        ['ktx', 'semantic', 'show', metric_name, '--sql-only'],
        capture_output=True,
        text=True,
        check=True
    )
    return result.stdout.strip()

# Usage
metrics = search_semantic_layer('revenue')
for metric in metrics.get('results', []):
    print(f"{metric['name']}: {metric['description']}")
    
sql = get_metric_sql('monthly_revenue')
print(f"SQL:\n{sql}")
```

### Semantic Layer YAML Example

Create a metric definition in `semantic-layer/warehouse/revenue.yaml`:

```yaml
type: metric
name: monthly_revenue
description: Total revenue aggregated by month
sql: |
  SELECT
    DATE_TRUNC('month', order_date) as month,
    SUM(amount) as revenue
  FROM orders
  WHERE status = 'completed'
  GROUP BY 1
dimensions:
  - month
  - customer_segment
  - region
measures:
  - revenue
  - order_count
grain: month
filters:
  - status = 'completed'
  - amount > 0
source_tables:
  - orders
  - customers
```

## Common Patterns

### Setting Up for a New Project

```bash
# Navigate to project
cd /path/to/analytics-project

# Initialize ktx
ktx setup

# Configure database
ktx connection add warehouse \
  --type postgres \
  --host $DB_HOST \
  --database analytics \
  --user-env POSTGRES_USER \
  --password-env POSTGRES_PASSWORD

# Add dbt context source
ktx context-source add dbt_models \
  --type dbt \
  --path ./dbt_project

# Build context
ktx ingest

# Verify
ktx status
```

### Daily Context Updates

```bash
# Re-ingest to pick up schema changes
ktx ingest --connection warehouse

# Add a new wiki page about metric changes
ktx wiki create --title "Q1 Metric Updates" --content-file q1-updates.md

# Search to verify new content
ktx wiki "Q1 metric"
```

### Agent Workflow

1. Agent searches semantic layer:

```text
Search ktx for customer retention metrics
```

2. ktx returns metrics with canonical SQL

3. Agent uses SQL definition to build query

4. Agent validates query against ktx schema

5. Agent executes query (read-only)

### Debugging Semantic Layer

```bash
# List all semantic sources
ktx semantic list

# Show detailed metric definition
ktx semantic show monthly_revenue

# Validate metric SQL
ktx semantic validate monthly_revenue

# Check join paths
ktx semantic joins orders customers
```

## Troubleshooting

### "ktx mcp start --project-dir ..." Message

If `ktx status` prints this message, run the command before opening your agent:

```bash
ktx mcp start --project-dir /path/to/project
```

This ensures the MCP server is ready for agent connections.

### Ingestion Fails

Check connection:

```bash
ktx connection test warehouse
```

Run with verbose logging:

```bash
ktx ingest --verbose
```

Skip problematic tables:

```bash
ktx ingest --exclude-tables table1,table2
```

### LLM Provider Errors

Verify API key is set:

```bash
echo $ANTHROPIC_API_KEY
```

Test with a different provider:

```bash
ktx config set llm.provider vertex
ktx ingest
```

### Semantic Search Returns No Results

Rebuild embeddings:

```bash
ktx ingest --force --rebuild-embeddings
```

Check embedding configuration:

```bash
ktx config get embeddings.provider
ktx config get embeddings.model
```

### Join Detection Missing Relationships

Increase sample size:

```bash
ktx ingest --sample-size 1000
```

Manually define joins in semantic layer YAML:

```yaml
type: entity
name: orders
primary_key: order_id
relationships:
  - entity: customers
    join_column: customer_id
    relationship_type: many_to_one
```

### Contradictions Flagged

Review flagged items:

```bash
ktx wiki search --contradictions-only
```

Resolve by editing wiki pages:

```bash
ktx wiki edit "Conflicting Metric Definition"
```

### Agent Can't Find ktx Tools

Ensure MCP server is running:

```bash
ps aux | grep 'ktx mcp'
```

Restart agent client after starting MCP server.

Check project directory is correct:

```bash
ktx status --project-dir /path/to/project
```

## Environment Variables

ktx uses environment variables for secrets and configuration:

- `ANTHROPIC_API_KEY`: Anthropic API key
- `OPENAI_API_KEY`: OpenAI API key (for embeddings)
- `POSTGRES_USER`, `POSTGRES_PASSWORD`: Database credentials
- `SNOWFLAKE_ACCOUNT`, `SNOWFLAKE_USER`, `SNOWFLAKE_PASSWORD`: Snowflake credentials
- `NOTION_API_KEY`: Notion integration token
- `KTX_PROJECT_DIR`: Override project directory resolution

Never hardcode secrets. Always reference environment variables in `ktx.yaml`:

```yaml
connections:
  warehouse:
    user_env: POSTGRES_USER
    password_env: POSTGRES_PASSWORD
```

## Resources

- **Documentation**: https://docs.kaelio.com/ktx
- **CLI Reference**: https://docs.kaelio.com/ktx/docs/cli-reference/ktx
- **Agent Setup Guide**: https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart
- **Slack Community**: https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ
- **GitHub Issues**: https://github.com/Kaelio/ktx/issues
