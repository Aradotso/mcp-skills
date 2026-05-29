---
name: ktx-data-agent-context-layer
description: Use ktx to build semantic context for AI agents querying data warehouses with MCP integration
triggers:
  - set up ktx for data agent queries
  - configure ktx semantic layer
  - integrate ktx with claude code
  - build warehouse context with ktx
  - query data using ktx mcp tools
  - ingest dbt metrics into ktx
  - search ktx wiki and semantic layer
  - troubleshoot ktx agent connection
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is an executable context layer that teaches AI agents how to query data warehouses accurately. It automatically builds and maintains a semantic layer from your warehouse metadata, dbt models, Looker/Metabase definitions, and company wikis. Agents access this context via CLI commands or MCP tools to write correct SQL with approved metric definitions.

## What ktx Does

- **Automatic warehouse introspection**: Samples tables, detects joinable columns, maps relationships
- **Semantic layer generation**: Combines raw tables and metrics with automatic fan/chasm trap resolution
- **Knowledge integration**: Ingests wiki content, dbt docs, LookML, and flags contradictions
- **Agent execution**: Exposes MCP server with search tools across semantic sources and wiki pages
- **Read-only by design**: Never writes to your data warehouse

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Local Installation

```bash
npm install --save-dev @kaelio/ktx
```

Or install as a skill:

```bash
npx skills add Kaelio/ktx --skill ktx
```

## Initial Setup

```bash
# Create or resume a ktx project
ktx setup

# Check project readiness
ktx status
```

The setup wizard guides you through:

1. **Project location**: Creates `ktx.yaml` in the current directory
2. **LLM provider**: Configure Anthropic API, Vertex AI, or Claude Code session
3. **Embeddings provider**: Configure OpenAI, Anthropic, or Vertex AI
4. **Database connections**: Add warehouse credentials (PostgreSQL, Snowflake, BigQuery, etc.)
5. **Context sources**: Add dbt projects, Looker instances, Metabase, or Notion
6. **Initial ingestion**: Builds semantic layer and wiki from all sources
7. **Agent integration**: Installs MCP server config for Claude Desktop, Codex, etc.

Example `ktx.yaml` after setup:

```yaml
version: 1
project:
  name: analytics
  id: 3a8c9f2d-1234-5678-90ab-cdef12345678

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

contextSources:
  dbt_main:
    type: dbt
    projectDir: ./dbt
    profilesDir: ~/.dbt
    target: prod
```

## Key Commands

### Project Management

```bash
# Check project status and readiness
ktx status

# Show project configuration
ktx config

# Rebuild all context
ktx ingest

# Rebuild specific connection
ktx ingest --connection warehouse
```

### Semantic Layer Search

```bash
# Search semantic sources (metrics, dimensions, tables)
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn" --limit 5

# Get detailed source info
ktx sl:get revenue_usd
```

### Wiki Search

```bash
# Search wiki pages
ktx wiki "refund policy"
ktx wiki "pricing tiers" --limit 10

# Get page content
ktx wiki:get refund-policy

# List all pages
ktx wiki:list
```

### MCP Server

```bash
# Start MCP server (for agent clients)
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status
```

## Configuration

### Environment Variables

```bash
# LLM provider credentials
export ANTHROPIC_API_KEY=your_key_here
export OPENAI_API_KEY=your_key_here
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Database connection (if not in ktx.yaml)
export WAREHOUSE_PASSWORD=your_password
export SNOWFLAKE_ACCOUNT=xy12345.us-east-1

# Project location override
export KTX_PROJECT_DIR=/path/to/ktx/project

# Disable telemetry
export KTX_TELEMETRY_DISABLED=1
```

### Database Connections

Add connections in `ktx.yaml` or via setup:

```yaml
connections:
  warehouse:
    type: postgres
    host: db.example.com
    port: 5432
    database: analytics
    schema: public
    # Password from env var: WAREHOUSE_PASSWORD

  snowflake:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    # Password from env var: SNOWFLAKE_PASSWORD
```

### Context Sources

#### dbt Integration

```yaml
contextSources:
  dbt_main:
    type: dbt
    projectDir: ./dbt
    profilesDir: ~/.dbt
    target: prod
```

#### Looker Integration

```yaml
contextSources:
  looker_prod:
    type: looker
    baseUrl: https://company.looker.com
    clientId: your_client_id
    # Client secret from env var: LOOKER_CLIENT_SECRET
```

#### Notion Integration

```yaml
contextSources:
  team_wiki:
    type: notion
    # Token from env var: NOTION_TOKEN
    databaseIds:
      - abc123def456
```

## MCP Integration

### Claude Desktop Configuration

After `ktx setup`, the MCP server is automatically configured. Manual config at `~/Library/Application Support/Claude/claude_desktop_config.json`:

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

### Codex Configuration

In `.codex/config.json`:

```json
{
  "mcpServers": {
    "ktx": {
      "command": "npx",
      "args": ["@kaelio/ktx", "mcp", "start"]
    }
  }
}
```

### Available MCP Tools

When connected via MCP, agents get these tools:

- `ktx_sl_search`: Search semantic sources (metrics, dimensions, tables)
- `ktx_sl_get`: Get detailed semantic source definition
- `ktx_wiki_search`: Search wiki pages
- `ktx_wiki_get`: Get full wiki page content
- `ktx_wiki_list`: List all wiki pages

## Common Patterns

### Initial Project Setup

```bash
# Navigate to your analytics project
cd ~/projects/analytics

# Initialize ktx
ktx setup

# Verify everything is ready
ktx status

# Should show:
# Project ready: yes
# LLM ready: yes
# Databases configured: yes
# Context built: yes
# Agent integration ready: yes
```

### Adding a New Database

```bash
# Run setup to add connection
ktx setup

# Or edit ktx.yaml directly
vim ktx.yaml

# Rebuild context for new connection
ktx ingest --connection new_warehouse
```

### Searching for Metrics

```bash
# Find revenue metrics
ktx sl "revenue"

# Example output:
# revenue_usd (metric)
#   Monthly recurring revenue in USD
#   SQL: SUM(amount_usd) WHERE type = 'subscription'
#   Dimensions: date, product, region
```

### Finding Business Logic

```bash
# Search wiki for policies
ktx wiki "customer lifetime value"

# Get full page
ktx wiki:get clv-calculation
```

### Agent Workflow

When an agent needs to answer a data question:

1. **Search semantic layer**: `ktx sl "metric name"` to find approved definitions
2. **Check wiki**: `ktx wiki "business term"` for context and policies
3. **Use definitions**: Write SQL using the canonical logic from semantic sources
4. **Execute query**: Connect to warehouse and run the SQL

## Troubleshooting

### MCP Server Not Starting

```bash
# Check project status
ktx status

# If it shows: "Run: ktx mcp start --project-dir ..."
# Execute that command before opening agent

# Check MCP logs
tail -f ~/.local/state/ktx/mcp.log
```

### Context Build Failures

```bash
# Check connection credentials
ktx config

# Test database connection
psql -h localhost -U user -d analytics -c "SELECT 1"

# Rebuild with verbose output
ktx ingest --verbose
```

### Missing Semantic Sources

```bash
# Verify context source configuration
cat ktx.yaml

# Check dbt project path
ls -la ./dbt/target/manifest.json

# Rebuild specific source
ktx ingest --source dbt_main
```

### LLM Provider Errors

```bash
# Verify API keys are set
echo $ANTHROPIC_API_KEY
echo $OPENAI_API_KEY

# Test LLM connection
ktx ingest --connection warehouse --verbose

# Switch provider in ktx.yaml
vim ktx.yaml
# Change llm.provider to 'anthropic' or 'vertex-ai'
```

### Search Returns No Results

```bash
# Check if context was built
ls -la semantic-layer/
ls -la wiki/

# Rebuild context
ktx ingest

# Verify embeddings provider
ktx status
# Should show: Embeddings ready: yes
```

### Agent Can't Access ktx Tools

```bash
# Restart MCP server
ktx mcp start --project-dir /path/to/project

# Verify MCP config
cat ~/Library/Application\ Support/Claude/claude_desktop_config.json

# Restart agent client (Claude Desktop, Codex)
```

### Permission Denied on Database

Ensure read-only user has proper grants:

```sql
-- PostgreSQL example
GRANT CONNECT ON DATABASE analytics TO ktx_user;
GRANT USAGE ON SCHEMA public TO ktx_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO ktx_user;
```

### Project Not Found

```bash
# Set project directory explicitly
export KTX_PROJECT_DIR=/path/to/project

# Or use flag
ktx status --project-dir /path/to/project

# Check for ktx.yaml
find . -name "ktx.yaml"
```

## Project Layout

```
my-project/
├── ktx.yaml                         # Project configuration (commit)
├── semantic-layer/
│   └── warehouse/                   # Generated semantic sources (commit)
│       ├── metrics.yaml
│       └── dimensions.yaml
├── wiki/
│   ├── global/                      # Shared wiki pages (commit)
│   └── user/                        # User-scoped notes (commit)
├── raw-sources/
│   └── warehouse/                   # Ingestion artifacts (optional commit)
│       ├── tables.json
│       └── columns.json
└── .ktx/                            # Local state and secrets (ignore)
    ├── state.db
    └── credentials.json
```

Add to `.gitignore`:

```
.ktx/
```

## Best Practices

1. **Run `ktx status` first**: Always check project readiness before debugging
2. **Commit semantic layer**: Check in `semantic-layer/` and `wiki/` directories
3. **Keep credentials in env vars**: Never commit API keys or passwords
4. **Rebuild after schema changes**: Run `ktx ingest` when warehouse schema changes
5. **Use descriptive connection IDs**: Name connections by purpose (e.g., `prod_warehouse`, `analytics_db`)
6. **Set project directory**: Use `KTX_PROJECT_DIR` env var in CI/CD
7. **Monitor MCP logs**: Check `~/.local/state/ktx/mcp.log` for connection issues
