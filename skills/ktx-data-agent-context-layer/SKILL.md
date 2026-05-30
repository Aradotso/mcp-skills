---
name: ktx-data-agent-context-layer
description: Build and query a self-improving context layer for AI data agents with ktx - combines warehouse metadata, metrics, and wiki knowledge
triggers:
  - set up ktx for data agent context
  - configure ktx semantic layer
  - build ktx warehouse context
  - query data using ktx MCP server
  - ingest dbt models into ktx
  - search ktx wiki or metrics
  - troubleshoot ktx agent integration
  - add database connection to ktx
---

# ktx Data Agent Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query your data warehouse accurately. It automatically ingests metadata from databases, dbt, LookML, Looker, Metabase, and Notion, builds a semantic layer with approved metric definitions, and exposes everything via CLI and MCP for agent execution.

## What ktx Does

- **Learns company knowledge**: Ingests wiki content, organizes it, removes duplicates, flags contradictions
- **Maps the data stack**: Samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds semantic layer**: Combines raw tables and metrics through a join graph that resolves fan/chasm traps
- **Serves agents**: CLI and MCP tools with semantic search across wiki and semantic-layer entities

**Key benefits for agents**:
- Query with approved metric definitions instead of inventing SQL every time
- Reuse canonical business logic across questions
- Get context from scattered sources (dbt, Looker, Notion) in one searchable surface

## Installation

### Global CLI Install

```bash
npm install -g @kaelio/ktx
```

### Project-Scoped Install

```bash
npm install --save-dev @kaelio/ktx
npx ktx setup
```

### As an MCP Skill

From Claude Code, Codex, Cursor, or OpenCode:

```text
Run npx skills add Kaelio/ktx --skill ktx
```

## Quick Start

```bash
# Create or resume a ktx project
ktx setup

# Check project readiness
ktx status

# Build context from configured sources
ktx ingest

# Search semantic layer
ktx sl "revenue"

# Search wiki
ktx wiki "refund policy"

# Start MCP server for agents
ktx mcp start
```

## Project Structure

A ktx project follows this layout:

```text
my-project/
├── ktx.yaml                         # Project configuration
├── semantic-layer/<connection-id>/  # YAML semantic sources
├── wiki/global/                     # Shared business context
├── wiki/user/<user-id>/             # User-scoped notes
├── raw-sources/<connection-id>/     # Ingest artifacts and reports
└── .ktx/                            # Local state and secrets (git-ignored)
```

**Version control**: Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Keep `.ktx/` local.

## Configuration

### ktx.yaml Example

```yaml
version: 1
project:
  name: analytics
  description: Data warehouse context for product analytics

llm:
  provider: anthropic
  model: claude-sonnet-4-6
  # API key from KTX_ANTHROPIC_API_KEY or .ktx/secrets.yaml

embeddings:
  provider: openai
  model: text-embedding-3-small
  # API key from KTX_OPENAI_API_KEY or .ktx/secrets.yaml

databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    # Credentials from KTX_DATABASE_WAREHOUSE_USER/PASSWORD env vars

context_sources:
  dbt_main:
    type: dbt
    project_path: ./dbt
    profiles_dir: ~/.dbt
    target: prod
  
  notion_docs:
    type: notion
    # Token from KTX_CONTEXT_SOURCE_NOTION_DOCS_TOKEN env var
    page_ids:
      - a1b2c3d4e5f6
      - f6e5d4c3b2a1
```

### Secrets Management

Store secrets in `.ktx/secrets.yaml` or environment variables:

```yaml
# .ktx/secrets.yaml (git-ignored)
llm:
  anthropic_api_key: sk-ant-...

embeddings:
  openai_api_key: sk-...

databases:
  warehouse:
    user: readonly
    password: ...

context_sources:
  notion_docs:
    token: secret_...
```

Environment variable names follow `KTX_<SECTION>_<KEY>` pattern:
- `KTX_ANTHROPIC_API_KEY`
- `KTX_OPENAI_API_KEY`
- `KTX_DATABASE_WAREHOUSE_USER`
- `KTX_DATABASE_WAREHOUSE_PASSWORD`
- `KTX_CONTEXT_SOURCE_NOTION_DOCS_TOKEN`

## Key Commands

### Setup & Status

```bash
# Interactive setup wizard
ktx setup

# Check project readiness
ktx status

# Example output:
# ktx project: /home/user/analytics
# Project ready: yes
# LLM ready: yes (claude-sonnet-4-6)
# Embeddings ready: yes (text-embedding-3-small)
# Databases configured: yes (warehouse)
# Context sources configured: yes (dbt_main, notion_docs)
# ktx context built: yes
# Agent integration ready: yes (codex:project)
```

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific context source
ktx ingest --context-source dbt_main

# Force rebuild (skip cache)
ktx ingest --force
```

### Searching Context

```bash
# Search semantic layer for metrics/dimensions
ktx sl "monthly recurring revenue"
ktx sl "user signup date"

# Search wiki pages
ktx wiki "customer refund policy"
ktx wiki "metric calculation rules"

# JSON output for programmatic use
ktx sl "revenue" --json
ktx wiki "refunds" --json
```

### MCP Server

```bash
# Start MCP server for agent integration
ktx mcp start

# Start with custom project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status
```

### Semantic Layer Query

```bash
# Query using semantic layer (declarative)
ktx query "revenue by month for 2024"

# Raw SQL query (with context hints)
ktx raw-query "SELECT * FROM users WHERE created_at > '2024-01-01'"
```

## Agent Integration Patterns

### Claude Code / Codex

After running `ktx setup`, the MCP server is automatically configured. From your agent:

```text
Use ktx to find the definition of monthly recurring revenue
```

```text
Search the wiki for our refund policy
```

```text
Query revenue by product category for Q4 2024 using ktx
```

### Programmatic MCP Client

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

const transport = new StdioClientTransport({
  command: "ktx",
  args: ["mcp", "start", "--project-dir", "/path/to/project"],
});

const client = new Client(
  {
    name: "my-agent",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

await client.connect(transport);

// Search semantic layer
const slResults = await client.callTool({
  name: "ktx_sl_search",
  arguments: {
    query: "revenue",
  },
});

// Search wiki
const wikiResults = await client.callTool({
  name: "ktx_wiki_search",
  arguments: {
    query: "refund policy",
  },
});

// Query with semantic layer
const queryResults = await client.callTool({
  name: "ktx_query",
  arguments: {
    query: "revenue by month for 2024",
    connection_id: "warehouse",
  },
});
```

## Semantic Layer YAML

ktx builds semantic sources from ingestion and stores them as YAML:

```yaml
# semantic-layer/warehouse/users.yaml
type: source
name: users
connection_id: warehouse
table: public.users
description: User account records with signup and subscription data
columns:
  - name: id
    type: integer
    primary_key: true
  - name: email
    type: varchar
    description: User email address
  - name: created_at
    type: timestamp
    description: Account creation timestamp
  - name: plan_id
    type: integer
    foreign_key:
      table: plans
      column: id
dimensions:
  - name: signup_date
    column: created_at
    type: time
    grain: day
measures:
  - name: user_count
    aggregation: count
    description: Total number of users
```

```yaml
# semantic-layer/warehouse/mrr.yaml
type: metric
name: monthly_recurring_revenue
connection_id: warehouse
description: Total MRR from active subscriptions
sql: |
  SELECT
    DATE_TRUNC('month', s.start_date) AS month,
    SUM(p.price) AS mrr
  FROM subscriptions s
  JOIN plans p ON s.plan_id = p.id
  WHERE s.status = 'active'
  GROUP BY 1
dimensions:
  - name: month
    type: time
    grain: month
measures:
  - name: mrr
    aggregation: sum
    type: currency
```

## Wiki Pages

ktx organizes wiki content in markdown:

```markdown
<!-- wiki/global/refund-policy.md -->
---
title: Customer Refund Policy
tags: [policy, customer-service, finance]
---

# Customer Refund Policy

## Eligibility

Customers can request refunds within 30 days of purchase if:
- Product defect
- Service unavailability > 24 hours
- Accidental duplicate purchase

## Processing

Refunds are processed within 5-7 business days.
Finance team approval required for amounts > $500.

## Metric Impact

Refunds reduce `net_revenue` but not `gross_revenue`.
Track via `refund_rate` metric in semantic layer.
```

## Database Connectors

Supported databases:

| Type | Configuration |
|------|--------------|
| PostgreSQL | `type: postgres` |
| Snowflake | `type: snowflake` |
| BigQuery | `type: bigquery` |
| ClickHouse | `type: clickhouse` |
| MySQL | `type: mysql` |
| SQL Server | `type: mssql` |
| SQLite | `type: sqlite` |

Example PostgreSQL configuration:

```yaml
databases:
  warehouse:
    type: postgres
    host: db.example.com
    port: 5432
    database: analytics
    schema: public
    # Credentials from env or secrets.yaml
```

Example Snowflake configuration:

```yaml
databases:
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    role: READONLY
```

## Context Source Integrations

### dbt

```yaml
context_sources:
  dbt_main:
    type: dbt
    project_path: ./dbt
    profiles_dir: ~/.dbt
    target: prod
```

Ingests:
- Model definitions and lineage
- Column descriptions
- Metric definitions (dbt Metrics or MetricFlow)
- Tests and constraints

### Looker

```yaml
context_sources:
  looker_prod:
    type: looker
    api_url: https://looker.example.com:19999
    # Client ID/secret from env vars
```

### LookML

```yaml
context_sources:
  lookml_repo:
    type: lookml
    repo_path: ./lookml
```

### Metabase

```yaml
context_sources:
  metabase:
    type: metabase
    url: https://metabase.example.com
    # API key from env var
```

### Notion

```yaml
context_sources:
  notion_docs:
    type: notion
    page_ids:
      - root-page-id-1
      - root-page-id-2
```

## Common Workflows

### Initial Setup for a New Project

```bash
# 1. Install ktx
npm install -g @kaelio/ktx

# 2. Navigate to your project
cd ~/analytics-project

# 3. Run interactive setup
ktx setup
# - Select LLM provider (Anthropic/Google/AI Gateway)
# - Configure embeddings provider (OpenAI/Google)
# - Add database connection (Postgres/Snowflake/etc)
# - Add context sources (dbt/Looker/Notion/etc)

# 4. Build initial context
ktx ingest

# 5. Verify setup
ktx status

# 6. Test search
ktx sl "revenue"
ktx wiki "business rules"
```

### Adding a New Database Connection

```bash
# Edit ktx.yaml
# Add new database under 'databases:' section

databases:
  new_warehouse:
    type: postgres
    host: new-db.example.com
    port: 5432
    database: prod

# Add credentials to .ktx/secrets.yaml or env vars
export KTX_DATABASE_NEW_WAREHOUSE_USER=readonly
export KTX_DATABASE_NEW_WAREHOUSE_PASSWORD=secret

# Ingest the new connection
ktx ingest --connection new_warehouse

# Verify
ktx status
```

### Updating Context After Schema Changes

```bash
# Force rebuild of all context
ktx ingest --force

# Or rebuild specific connection
ktx ingest --connection warehouse --force

# Or rebuild specific context source
ktx ingest --context-source dbt_main --force
```

### Using with Claude Code

```bash
# 1. Ensure MCP server is configured
ktx status
# If output shows: "Run ktx mcp start --project-dir ..."
# Copy and run that command before opening Claude Code

# 2. From Claude Code, ask:
```

```text
Search ktx for the definition of customer lifetime value
```

```text
Use ktx to query monthly active users for the last 6 months
```

```text
Check the ktx wiki for our data retention policy
```

## Troubleshooting

### "Project ready: no"

```bash
ktx status
# Check which component is not ready

# Missing ktx.yaml?
ktx setup

# Missing secrets?
# Add to .ktx/secrets.yaml or export env vars
```

### "LLM ready: no"

```bash
# Check API key is set
echo $KTX_ANTHROPIC_API_KEY

# Or add to secrets.yaml
cat > .ktx/secrets.yaml << EOF
llm:
  anthropic_api_key: sk-ant-...
EOF

# Verify provider in ktx.yaml
# llm:
#   provider: anthropic  # or google, ai_gateway
```

### "Databases configured: no"

```bash
# Check ktx.yaml has databases section
cat ktx.yaml | grep -A 5 databases

# Test connection credentials
export KTX_DATABASE_WAREHOUSE_USER=readonly
export KTX_DATABASE_WAREHOUSE_PASSWORD=secret

# Try ingest
ktx ingest --connection warehouse
```

### Ingest Fails

```bash
# Check connection details
ktx ingest --connection warehouse --verbose

# For dbt sources, verify project path
# context_sources:
#   dbt_main:
#     project_path: ./dbt  # Must contain dbt_project.yml

# For API sources (Looker/Metabase/Notion), verify tokens
echo $KTX_CONTEXT_SOURCE_LOOKER_PROD_CLIENT_ID
echo $KTX_CONTEXT_SOURCE_LOOKER_PROD_CLIENT_SECRET
```

### MCP Server Not Found by Agent

```bash
# 1. Check ktx status output for MCP command
ktx status

# 2. Run the command it shows
ktx mcp start --project-dir /path/to/project

# 3. Keep that terminal open while using agent

# For persistent setup, configure agent's MCP settings:
# Claude Code: Add to mcp.json
# Cursor: Add to .cursor/mcp.json
```

### Search Returns No Results

```bash
# Ensure context is built
ktx ingest

# Check semantic layer files exist
ls -la semantic-layer/*/

# Check wiki files exist
ls -la wiki/global/

# Try broader search
ktx sl "revenue"  # vs "monthly_recurring_revenue"

# Use JSON output to debug
ktx sl "revenue" --json | jq .
```

### Read-Only Database Error

ktx only reads from your database. If you see write errors:

```bash
# Check database user permissions
# User should have SELECT only

# In Postgres:
# GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

# In Snowflake:
# GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE READONLY;
# GRANT SELECT ON ALL TABLES IN SCHEMA PUBLIC TO ROLE READONLY;
```

### Telemetry Opt-Out

ktx collects anonymous usage data. To disable:

```bash
# Environment variable
export KTX_TELEMETRY_DISABLED=1

# Or in ktx.yaml
telemetry:
  enabled: false
```

## Advanced Usage

### Custom Project Directory

```bash
# All commands accept --project-dir
ktx status --project-dir /path/to/project
ktx ingest --project-dir /path/to/project
ktx sl "revenue" --project-dir /path/to/project

# Or set environment variable
export KTX_PROJECT_DIR=/path/to/project
ktx status
```

### Multiple Projects

```bash
# Project A
cd ~/project-a
ktx setup
ktx ingest

# Project B
cd ~/project-b
ktx setup
ktx ingest

# Query project A from anywhere
ktx sl "revenue" --project-dir ~/project-a

# Query project B from anywhere
ktx wiki "policy" --project-dir ~/project-b
```

### Programmatic Query Execution

```typescript
import { execSync } from 'child_process';

// Execute ktx command and parse JSON output
function ktxSearch(query: string, type: 'sl' | 'wiki'): any {
  const cmd = type === 'sl' 
    ? `ktx sl "${query}" --json`
    : `ktx wiki "${query}" --json`;
  
  const output = execSync(cmd, { encoding: 'utf-8' });
  return JSON.parse(output);
}

const metrics = ktxSearch('revenue', 'sl');
const policies = ktxSearch('refund', 'wiki');

console.log('Found metrics:', metrics.length);
console.log('Found wiki pages:', policies.length);
```

## Project Resolution

ktx finds your project in this order:

1. `--project-dir` flag
2. `KTX_PROJECT_DIR` environment variable
3. Nearest `ktx.yaml` in current or parent directories
4. Current working directory

Always use `--project-dir` or `KTX_PROJECT_DIR` in scripts for reliability.
