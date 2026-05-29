---
name: ktx-ai-data-agents-context-layer
description: Use ktx to build a self-improving context layer that teaches AI agents how to query data warehouses accurately with approved metrics, wiki knowledge, and semantic layer definitions
triggers:
  - set up ktx for data agent queries
  - configure ktx context layer for warehouse
  - build semantic layer with ktx
  - integrate ktx with claude code for analytics
  - ingest warehouse metadata into ktx
  - search ktx wiki and metrics
  - configure ktx database connections
  - troubleshoot ktx mcp server
---

# ktx AI Data Agents Context Layer Skill

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

**ktx** is a self-improving context layer that teaches AI agents how to query data warehouses accurately. It combines semantic layer definitions, wiki knowledge, warehouse metadata, and joinable column detection into a unified searchable surface exposed via CLI and MCP tools. Agents get approved metric definitions, business context, and schema understanding without re-exploring the warehouse on every query.

## What ktx Does

- **Ingests company knowledge** from dbt, MetricFlow, LookML, Looker, Metabase, Notion, and local wikis
- **Maps the data stack** by sampling tables, detecting joinable columns, and resolving fan/chasm traps
- **Builds a semantic layer** combining raw tables and high-level metrics through an auto-generated join graph
- **Serves agents** via CLI and MCP with full-text and semantic search across all context sources
- **Read-only by design** — never writes to your warehouse

Supports PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite.

## Installation

```bash
# Install globally
npm install -g @kaelio/ktx

# Or use with npx
npx @kaelio/ktx setup
```

For agent integration via skills:

```bash
npx skills add Kaelio/ktx --skill ktx
```

## Initial Setup

### Interactive Setup

```bash
ktx setup
```

This command:
1. Creates or resumes a local ktx project
2. Configures LLM and embedding providers
3. Sets up database connections
4. Configures context sources (dbt, wiki, etc.)
5. Builds initial context
6. Installs agent integration

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

**Git best practices:**
- Commit: `ktx.yaml`, `semantic-layer/`, `wiki/`
- Ignore: `.ktx/` (contains secrets and local state)

## Configuration

### ktx.yaml Structure

```yaml
version: 1
llm:
  provider: anthropic
  model: claude-sonnet-4-6
  credentials:
    credentialType: env
    envVar: ANTHROPIC_API_KEY
embeddings:
  provider: openai
  model: text-embedding-3-small
  credentials:
    credentialType: env
    envVar: OPENAI_API_KEY
databases:
  - id: warehouse
    type: postgres
    credentials:
      credentialType: env
      envVar: DATABASE_URL
context:
  - type: dbt
    id: dbt_main
    path: ./dbt
    databaseId: warehouse
  - type: wiki
    id: local_wiki
    path: ./wiki
```

### Database Connection Types

**PostgreSQL:**
```yaml
databases:
  - id: warehouse
    type: postgres
    credentials:
      credentialType: env
      envVar: DATABASE_URL  # postgres://user:pass@host:5432/db
```

**Snowflake:**
```yaml
databases:
  - id: snowflake_prod
    type: snowflake
    account: myorg-myaccount
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
    credentials:
      credentialType: env
      envVar: SNOWFLAKE_PASSWORD
```

**BigQuery:**
```yaml
databases:
  - id: bigquery_prod
    type: bigquery
    projectId: my-gcp-project
    dataset: analytics
    credentials:
      credentialType: env
      envVar: GOOGLE_APPLICATION_CREDENTIALS  # path to service account JSON
```

## Core Commands

### Project Status

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

### Building Context

```bash
# Ingest all configured sources
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest specific context source
ktx ingest --source dbt_main

# Force rebuild (skip cache)
ktx ingest --force
```

### Searching Context

**Search semantic layer:**
```bash
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn rate"
```

**Search wiki:**
```bash
ktx wiki "refund policy"
ktx wiki "data retention rules"
ktx wiki "metric definitions"
```

### MCP Server

Start the MCP server for agent clients:

```bash
# Start with auto-detected project
ktx mcp start

# Start with explicit project directory
ktx mcp start --project-dir /path/to/project

# Background mode (if supported by your agent)
ktx mcp start --daemon
```

**Important:** The MCP server must be running before opening your agent client (Claude Code, Cursor, etc.).

## Context Source Configuration

### dbt Projects

```yaml
context:
  - type: dbt
    id: dbt_analytics
    path: ./dbt
    databaseId: warehouse
    manifestPath: ./dbt/target/manifest.json  # optional, auto-detected
```

ktx ingests:
- Models, sources, metrics from `manifest.json`
- Column descriptions and metadata
- Model dependencies and lineage
- dbt metrics definitions

### MetricFlow

```yaml
context:
  - type: metricflow
    id: mf_prod
    path: ./metrics
    databaseId: warehouse
```

### LookML (Looker)

```yaml
context:
  - type: lookml
    id: looker_main
    path: ./lookml
    databaseId: warehouse
```

### Wiki/Notion

```yaml
context:
  - type: wiki
    id: team_wiki
    path: ./wiki
  - type: notion
    id: notion_docs
    credentials:
      credentialType: env
      envVar: NOTION_API_TOKEN
    databaseId: <notion-database-id>
```

## TypeScript Usage (Programmatic)

ktx is primarily a CLI tool, but you can use its TypeScript API:

```typescript
import { KtxProject } from '@kaelio/ktx';

async function queryContext() {
  const project = await KtxProject.load('/path/to/project');
  
  // Search semantic layer
  const metrics = await project.semanticLayer.search('revenue', {
    limit: 10,
    connectionId: 'warehouse'
  });
  
  console.log('Found metrics:', metrics);
  
  // Search wiki
  const wikiResults = await project.wiki.search('refund policy', {
    limit: 5
  });
  
  console.log('Found wiki pages:', wikiResults);
  
  // Get all semantic sources for a connection
  const sources = await project.semanticLayer.getSources('warehouse');
  
  for (const source of sources) {
    console.log(`Source: ${source.name}`);
    console.log(`Metrics: ${source.metrics?.map(m => m.name).join(', ')}`);
  }
}
```

## Agent Integration Patterns

### Claude Code / Codex

When ktx is configured, agents can:

**Query available metrics:**
```text
What revenue metrics are available in ktx?
```

Agent will run:
```bash
ktx sl "revenue"
```

**Find business context:**
```text
What's our refund policy according to ktx wiki?
```

Agent will run:
```bash
ktx wiki "refund policy"
```

**Generate queries with context:**
```text
Write a SQL query for monthly revenue using ktx approved definitions
```

Agent will:
1. Search semantic layer: `ktx sl "monthly revenue"`
2. Use the returned metric definition
3. Generate SQL that matches approved logic

### Cursor / OpenCode

Add ktx to your agent's context:

```text
Use ktx to find the approved definition of monthly recurring revenue, then write a SQL query
```

## Common Workflows

### Setting Up a New Warehouse

```bash
# 1. Initialize project
ktx setup

# 2. Configure database connection when prompted
# Select database type (postgres, snowflake, etc.)
# Enter connection details
# Connection string stored in .ktx/secrets.yaml

# 3. Add context sources
# dbt, wiki, lookml, etc.

# 4. Build context
ktx ingest

# 5. Verify
ktx status
ktx sl "test search"
```

### Adding a New Metric Definition

Create `semantic-layer/<connection-id>/metrics.yaml`:

```yaml
metrics:
  - name: monthly_recurring_revenue
    description: Total recurring revenue normalized to monthly basis
    type: simple
    typeParams:
      measure:
        name: revenue
        agg: sum
      expr: |
        CASE 
          WHEN subscription_type = 'annual' THEN amount / 12
          ELSE amount
        END
    filter: |
      status = 'active'
      AND cancelled_at IS NULL
```

Rebuild context:
```bash
ktx ingest --force
```

### Updating Wiki Knowledge

Add markdown files to `wiki/global/`:

```bash
mkdir -p wiki/global/policies
cat > wiki/global/policies/refunds.md << 'EOF'
# Refund Policy

## Standard Refunds
- Full refund within 30 days of purchase
- Prorated refund for annual subscriptions after 30 days

## Chargeback Handling
- Automatically mark customer account as suspended
- Require manual review before reactivation
EOF
```

Rebuild:
```bash
ktx ingest --source local_wiki
```

### Detecting Joinable Columns

ktx automatically detects joinable columns during ingestion. View the join graph:

```bash
# Search for a table
ktx sl "customers"

# View detected joins in semantic-layer/<connection-id>/tables.yaml
cat semantic-layer/warehouse/tables.yaml
```

Example auto-generated join:

```yaml
tables:
  - name: orders
    columns:
      - name: customer_id
        type: integer
        joinable:
          - table: customers
            column: id
            confidence: high
            evidence:
              - foreignKeyConstraint
              - nameSimilarity
              - valueSampling
```

## Troubleshooting

### MCP Server Not Starting

**Symptom:** Agent can't connect to ktx

**Solution:**
```bash
# Check status
ktx status

# If it shows "ktx mcp start --project-dir ...", run that command
ktx mcp start --project-dir /path/to/project

# Check logs
cat .ktx/mcp-server.log
```

### Ingestion Fails

**Symptom:** `ktx ingest` errors

**Solutions:**

1. **Database connection issues:**
```bash
# Test connection
ktx setup  # Re-run setup to reconfigure connection

# Check environment variables
echo $DATABASE_URL
echo $SNOWFLAKE_PASSWORD
```

2. **LLM provider issues:**
```bash
# Verify API keys
echo $ANTHROPIC_API_KEY
echo $OPENAI_API_KEY

# Check LLM configuration
grep -A 5 "^llm:" ktx.yaml
```

3. **Permission issues:**
```bash
# Ensure read access to warehouse
# ktx only needs SELECT permissions

# For Snowflake, grant minimal role:
GRANT USAGE ON WAREHOUSE COMPUTE_WH TO ROLE ktx_reader;
GRANT USAGE ON DATABASE ANALYTICS TO ROLE ktx_reader;
GRANT USAGE ON SCHEMA PUBLIC TO ROLE ktx_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA PUBLIC TO ROLE ktx_reader;
```

### Semantic Search Returns No Results

**Symptom:** `ktx sl "query"` returns empty results

**Solutions:**

1. **Check embeddings configuration:**
```yaml
embeddings:
  provider: openai
  model: text-embedding-3-small
  credentials:
    credentialType: env
    envVar: OPENAI_API_KEY
```

2. **Rebuild embeddings:**
```bash
ktx ingest --force
```

3. **Verify context was built:**
```bash
ktx status
# Should show "ktx context built: yes"

# Check semantic layer files exist
ls -la semantic-layer/*/
```

### Conflicting Metric Definitions

**Symptom:** ktx flags contradictions between dbt and Looker metrics

**Solution:**

1. **Review flagged conflicts:**
```bash
cat raw-sources/warehouse/conflicts.yaml
```

2. **Resolve in source systems or override in ktx:**
```yaml
# semantic-layer/warehouse/metrics.yaml
metrics:
  - name: monthly_revenue
    description: Canonical monthly revenue definition
    source: dbt  # explicit source preference
    overrides:
      - source: looker
        reason: "dbt includes refunds, looker does not"
```

3. **Rebuild:**
```bash
ktx ingest --force
```

### Agent Can't Access ktx Commands

**Symptom:** Agent says "ktx: command not found"

**Solutions:**

1. **Global install:**
```bash
npm install -g @kaelio/ktx
which ktx  # Should show path
```

2. **Use npx:**
```bash
npx @kaelio/ktx status
```

3. **Install via skills:**
```bash
npx skills add Kaelio/ktx --skill ktx
```

4. **Restart agent client** after installation

## Advanced Configuration

### Custom LLM Providers

**Vertex AI:**
```yaml
llm:
  provider: vertex
  model: claude-3-5-sonnet@20241022
  project: my-gcp-project
  region: us-central1
  credentials:
    credentialType: env
    envVar: GOOGLE_APPLICATION_CREDENTIALS
```

**AI Gateway:**
```yaml
llm:
  provider: ai-gateway
  baseUrl: https://gateway.ai.cloudflare.com/v1/account/gateway/
  model: anthropic/claude-sonnet-4-6
  credentials:
    credentialType: env
    envVar: AI_GATEWAY_API_KEY
```

### Multiple Database Connections

```yaml
databases:
  - id: prod_warehouse
    type: snowflake
    account: prod-account
    warehouse: PROD_WH
    credentials:
      credentialType: env
      envVar: SNOWFLAKE_PROD_PASSWORD
  - id: dev_warehouse
    type: postgres
    credentials:
      credentialType: env
      envVar: DEV_DATABASE_URL
  - id: clickhouse_events
    type: clickhouse
    host: events.example.com
    port: 9000
    database: events
    credentials:
      credentialType: env
      envVar: CLICKHOUSE_PASSWORD
```

### Sampling Configuration

Control how ktx samples tables:

```yaml
context:
  - type: dbt
    id: dbt_main
    path: ./dbt
    databaseId: warehouse
    sampling:
      maxRows: 1000  # Sample up to 1000 rows per table
      timeout: 30000  # 30 second timeout per table
      skipLargeTables: true  # Skip tables over 100M rows
```

## Environment Variables Reference

```bash
# LLM Providers
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json

# Database Connections
DATABASE_URL=postgres://user:pass@host:5432/db
SNOWFLAKE_PASSWORD=...
BIGQUERY_PROJECT=my-project-id

# ktx Configuration
KTX_PROJECT_DIR=/path/to/project  # Override project directory
KTX_TELEMETRY=false  # Disable telemetry (opt-out)

# MCP Server
KTX_MCP_PORT=3000  # Custom MCP server port
```

## Best Practices

1. **Version control:** Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Ignore `.ktx/`.

2. **Environment-specific projects:** Use separate ktx projects for dev/staging/prod:
   ```bash
   ~/analytics-dev/ktx.yaml
   ~/analytics-prod/ktx.yaml
   ```

3. **Incremental updates:** Run `ktx ingest` regularly (e.g., nightly CI job) to keep context fresh.

4. **Wiki organization:** Structure wiki with clear hierarchy:
   ```text
   wiki/global/
   ├── metrics/           # Metric definitions and business logic
   ├── policies/          # Business policies
   ├── data-dictionary/   # Column definitions
   └── guides/            # How-to guides
   ```

5. **Metric naming:** Use consistent, descriptive names:
   ```yaml
   # Good
   monthly_recurring_revenue
   customer_lifetime_value
   daily_active_users
   
   # Avoid
   mrr  # too abbreviated
   rev  # ambiguous
   ```

6. **Security:** Never commit `.ktx/secrets.yaml`. Use environment variables for credentials.

7. **Agent prompts:** Be specific when asking agents to use ktx:
   ```text
   # Good
   "Use ktx to find the approved definition of ARR, then calculate it for Q4 2024"
   
   # Less effective
   "Calculate ARR"
   ```
