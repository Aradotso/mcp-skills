---
name: ktx-data-agent-context-layer
description: Install and configure ktx, a context layer that teaches AI agents to query data warehouses accurately with approved metrics and business knowledge
triggers:
  - set up ktx for data agents
  - configure ktx semantic layer
  - install ktx context layer
  - help me query my data warehouse with ktx
  - add ktx to my analytics project
  - configure ktx for Claude Code
  - build context with ktx
  - set up data agent skills with ktx
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

ktx is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It ingests metric definitions from dbt, LookML, and other sources, absorbs business knowledge from wikis and documentation, maps database schemas, detects joinable columns, and exposes everything through CLI and MCP tools for AI agents.

**Key features:**
- Automatically builds warehouse context from raw tables, dbt, MetricFlow, LookML, Looker, Metabase
- Ingests wiki content (Notion, local docs) and flags contradictions
- Maps joinable columns and resolves fan/chasm traps
- Exposes semantic layer with approved metric definitions
- Read-only by design — never writes to your warehouse
- Runs locally with your own LLM API keys

## Installation

### Global installation

```bash
npm install -g @kaelio/ktx
```

### Project-local installation

```bash
npm install --save-dev @kaelio/ktx
```

Then run via `npx ktx` or add to package.json scripts.

## Initial Setup

### Interactive setup

Run the interactive setup wizard to create or resume a ktx project:

```bash
ktx setup
```

This will:
1. Create `ktx.yaml` configuration
2. Configure LLM and embedding providers
3. Set up database connections
4. Configure context sources (dbt, LookML, etc.)
5. Build initial context
6. Install agent integration

### Check project status

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

## Project Configuration

### ktx.yaml structure

```yaml
version: "1"
project:
  name: my-analytics-project
  description: Data warehouse context for agents

llm:
  provider: anthropic
  model: claude-sonnet-4-6

embeddings:
  provider: openai
  model: text-embedding-3-small

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    # Credentials in .ktx/secrets.yaml

context_sources:
  dbt_main:
    type: dbt
    path: ./dbt
    connection: warehouse
    
  looker_prod:
    type: lookml
    path: ./looker
    connection: warehouse
    
  notion_wiki:
    type: notion
    integration_token_env: NOTION_INTEGRATION_TOKEN
    database_id_env: NOTION_DATABASE_ID
```

### Secrets management

Secrets are stored in `.ktx/secrets.yaml` (git-ignored):

```yaml
connections:
  warehouse:
    user: ${WAREHOUSE_USER}
    password: ${WAREHOUSE_PASSWORD}

providers:
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}
  openai:
    api_key: ${OPENAI_API_KEY}
```

Reference environment variables with `${VAR_NAME}` syntax.

## Building Context

### Ingest all sources

```bash
ktx ingest
```

This scans all configured connections and context sources, building:
- Raw table metadata and samples
- Joinable column detection
- Semantic layer from dbt/LookML/etc.
- Wiki pages from Notion/local docs

### Ingest specific connection

```bash
ktx ingest --connection warehouse
```

### Ingest specific source

```bash
ktx ingest --source dbt_main
```

### View ingestion status

```bash
ktx ingest --status
```

## Searching Context

### Search semantic layer

Find metrics, dimensions, and entities:

```bash
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer lifetime value"
```

Returns structured semantic entities that agents can query.

### Search wiki

Search business knowledge:

```bash
ktx wiki "refund policy"
ktx wiki "customer segmentation"
ktx wiki "revenue recognition rules"
```

Returns relevant wiki pages with context.

### Combined search

```bash
ktx search "user churn"
```

Searches both semantic layer and wiki simultaneously.

## MCP Server for Agents

### Start MCP server

```bash
ktx mcp start
```

By default, this starts an MCP server that agents can connect to. The server exposes tools for:
- `search_semantic_layer` - Query metrics and dimensions
- `search_wiki` - Search business knowledge
- `get_table_metadata` - Inspect raw tables
- `validate_query` - Check SQL against semantic layer

### Start with custom project directory

```bash
ktx mcp start --project-dir /path/to/project
```

### Configure in Claude Desktop

Add to `claude_desktop_config.json`:

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

### Configure in Codex

ktx setup automatically configures Codex integration. Verify with:

```bash
ktx status
```

Look for `Agent integration ready: yes (codex:project)`.

## TypeScript/JavaScript API

### Programmatic usage

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load project
const project = await KtxProject.load('/path/to/project');

// Search semantic layer
const metrics = await project.searchSemanticLayer('revenue', {
  limit: 10,
  threshold: 0.7
});

console.log(metrics.results.map(r => ({
  name: r.entity.name,
  type: r.entity.type,
  score: r.score
})));

// Search wiki
const wikiPages = await project.searchWiki('refund policy', {
  limit: 5
});

console.log(wikiPages.results.map(r => ({
  title: r.page.title,
  excerpt: r.excerpt,
  score: r.score
})));

// Get table metadata
const table = await project.getTableMetadata('warehouse', 'public', 'users');
console.log({
  columns: table.columns.length,
  rowCount: table.rowCount,
  joinableWith: table.joinableColumns
});
```

### Query generation

```typescript
import { SemanticLayer } from '@kaelio/ktx';

const sl = await SemanticLayer.load(project);

// Generate SQL for metric query
const query = await sl.generateQuery({
  metrics: ['revenue', 'order_count'],
  dimensions: ['country', 'month'],
  filters: {
    'order_date': { gte: '2024-01-01' }
  }
});

console.log(query.sql);
console.log(query.resolvedJoins);
```

## Common Patterns

### Adding a new database connection

```typescript
// In code
await project.addConnection({
  id: 'prod_warehouse',
  type: 'snowflake',
  account: 'abc12345.us-east-1',
  warehouse: 'COMPUTE_WH',
  database: 'ANALYTICS',
  schema: 'PUBLIC',
  role: 'ANALYST'
});
```

Or via interactive setup:

```bash
ktx setup --add-connection
```

### Adding dbt as a context source

```yaml
# ktx.yaml
context_sources:
  dbt_production:
    type: dbt
    path: ./dbt
    connection: warehouse
    manifest_path: ./dbt/target/manifest.json
    profiles_yml: ./dbt/profiles.yml
```

Then ingest:

```bash
ktx ingest --source dbt_production
```

### Adding Notion wiki

```yaml
# ktx.yaml
context_sources:
  notion_docs:
    type: notion
    integration_token_env: NOTION_INTEGRATION_TOKEN
    database_id_env: NOTION_DATABASE_ID
```

Set environment variables:

```bash
export NOTION_INTEGRATION_TOKEN=secret_...
export NOTION_DATABASE_ID=abc123...
ktx ingest --source notion_docs
```

### Creating custom wiki pages

```bash
# Create global wiki page
echo "# Refund Policy\n\nCustomers can request refunds within 30 days..." > wiki/global/refund-policy.md

# Create user-scoped note
mkdir -p wiki/user/$(whoami)
echo "# Analysis Notes\n\nRevenue spike in Q4 due to..." > wiki/user/$(whoami)/q4-analysis.md

# Re-ingest to index
ktx ingest
```

### Viewing semantic layer YAML

ktx generates semantic layer definitions in `semantic-layer/<connection-id>/`:

```yaml
# semantic-layer/warehouse/metrics.yaml
metrics:
  - id: revenue
    name: Revenue
    type: sum
    sql: amount
    base_entity: orders
    description: Total order revenue
    
  - id: average_order_value
    name: Average Order Value
    type: average
    sql: amount
    base_entity: orders
```

These are auto-generated from dbt, LookML, or raw table introspection.

### Resolving contradictions

ktx flags contradictions across sources:

```bash
ktx ingest --validate
```

Review flagged items in `raw-sources/<connection-id>/contradictions.yaml`:

```yaml
contradictions:
  - type: metric_definition_mismatch
    severity: high
    description: "Revenue metric defined differently in dbt vs LookML"
    sources:
      - dbt_main:revenue (SUM(amount))
      - looker_prod:revenue (SUM(amount * quantity))
    recommendation: Align definitions or rename one metric
```

## CLI Reference

### Core commands

| Command | Description |
|---------|-------------|
| `ktx setup` | Create or resume ktx project (interactive) |
| `ktx status` | Check project readiness |
| `ktx ingest` | Build context from all sources |
| `ktx sl <query>` | Search semantic layer |
| `ktx wiki <query>` | Search wiki pages |
| `ktx search <query>` | Search both semantic layer and wiki |
| `ktx mcp start` | Start MCP server for agents |

### Setup options

```bash
ktx setup                    # Interactive setup
ktx setup --add-connection   # Add new database connection
ktx setup --add-source       # Add new context source
ktx setup --reconfigure-llm  # Change LLM provider
```

### Ingest options

```bash
ktx ingest                          # Ingest all sources
ktx ingest --connection warehouse   # Ingest specific connection
ktx ingest --source dbt_main        # Ingest specific source
ktx ingest --validate               # Check for contradictions
ktx ingest --status                 # View ingestion status
ktx ingest --force                  # Re-ingest everything
```

### Search options

```bash
ktx sl "revenue" --limit 20             # More results
ktx sl "revenue" --threshold 0.8        # Higher relevance threshold
ktx wiki "policy" --connection warehouse # Scope to connection
ktx search "churn" --format json        # JSON output
```

### MCP options

```bash
ktx mcp start                           # Start with default project
ktx mcp start --project-dir /path       # Custom project directory
ktx mcp start --port 3000               # Custom port
ktx mcp stop                            # Stop running server
```

### Global options

```bash
--project-dir <path>   # Override project directory
--verbose             # Verbose logging
--quiet               # Suppress non-error output
--no-color            # Disable color output
```

## Troubleshooting

### "No LLM provider configured"

Run `ktx setup` and configure an LLM provider:

```bash
ktx setup --reconfigure-llm
```

Or set environment variables:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
export KTX_LLM_PROVIDER=anthropic
export KTX_LLM_MODEL=claude-sonnet-4-6
```

### "Database connection failed"

Check credentials in `.ktx/secrets.yaml` and test connection:

```bash
ktx ingest --connection warehouse --verbose
```

Ensure environment variables are set:

```bash
export WAREHOUSE_USER=myuser
export WAREHOUSE_PASSWORD=mypass
```

### "No context built"

Run initial ingestion:

```bash
ktx ingest
```

Check status:

```bash
ktx ingest --status
```

### "MCP server not responding"

Ensure server is running:

```bash
ktx mcp start --verbose
```

Check agent configuration points to correct project directory.

### "Semantic layer query fails"

Validate semantic layer definitions:

```bash
ktx ingest --validate
```

Check for contradictions in `raw-sources/<connection-id>/contradictions.yaml`.

### "Embedding search returns poor results"

Re-ingest to rebuild embeddings:

```bash
ktx ingest --force
```

Or adjust search threshold:

```bash
ktx sl "revenue" --threshold 0.6
```

### Project resolution issues

Explicitly set project directory:

```bash
export KTX_PROJECT_DIR=/path/to/project
```

Or pass on every command:

```bash
ktx status --project-dir /path/to/project
```

## Integration Examples

### Using ktx from an agent

Once ktx is configured, agents can use it directly:

**Claude Code/Codex:**
```text
Use ktx to search for revenue metrics and show me the definition
```

**Cursor:**
```text
@ktx what are the approved definitions for customer churn?
```

### Automated ingestion

```bash
#!/bin/bash
# daily-ingest.sh

cd /path/to/project
export ANTHROPIC_API_KEY=...
export WAREHOUSE_PASSWORD=...

ktx ingest --validate > /var/log/ktx/ingest.log 2>&1

if [ $? -ne 0 ]; then
  echo "ktx ingestion failed" | mail -s "Alert" ops@company.com
fi
```

### CI/CD integration

```yaml
# .github/workflows/ktx-validate.yml
name: Validate ktx Context
on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install -g @kaelio/ktx
      - run: ktx ingest --validate --no-build
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          WAREHOUSE_USER: ${{ secrets.WAREHOUSE_USER }}
          WAREHOUSE_PASSWORD: ${{ secrets.WAREHOUSE_PASSWORD }}
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `KTX_PROJECT_DIR` | Override project directory |
| `ANTHROPIC_API_KEY` | Anthropic API key for LLM |
| `OPENAI_API_KEY` | OpenAI API key for embeddings |
| `KTX_LLM_PROVIDER` | LLM provider (anthropic, vertex, etc.) |
| `KTX_LLM_MODEL` | LLM model name |
| `KTX_EMBEDDINGS_PROVIDER` | Embeddings provider |
| `KTX_EMBEDDINGS_MODEL` | Embeddings model name |
| `NOTION_INTEGRATION_TOKEN` | Notion integration token |
| `NOTION_DATABASE_ID` | Notion database ID |

Connection credentials can be set via environment variables referenced in `.ktx/secrets.yaml`:

```yaml
connections:
  warehouse:
    user: ${WAREHOUSE_USER}
    password: ${WAREHOUSE_PASSWORD}
```
