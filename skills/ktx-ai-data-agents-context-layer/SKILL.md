---
name: ktx-ai-data-agents-context-layer
description: Context layer for data agents that teaches AI how to query warehouses accurately with approved metrics, semantic layers, and business knowledge
triggers:
  - set up ktx for data agent queries
  - configure ktx context layer for warehouse
  - build semantic layer with ktx
  - ingest warehouse metadata with ktx
  - connect claude to data warehouse with ktx
  - create ktx project for analytics agents
  - query warehouse metrics through ktx mcp
  - configure ktx semantic layer and wiki
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

ktx is an executable context layer for data and analytics agents. It teaches AI agents (Claude Code, Codex, Cursor, etc.) how to query your data warehouse accurately by combining approved metric definitions, joinable columns, and business knowledge from your entire data stack.

## What ktx Does

ktx solves the problem of general-purpose agents struggling with data tasks. Instead of re-exploring your warehouse on every question and inventing metric logic, ktx:

- **Learns from company knowledge** — ingests wiki content, organizes it, removes duplicates, flags contradictions
- **Maps the data stack** — samples tables, captures metadata, detects joinable columns, annotates sources
- **Builds a semantic layer** — combines raw tables and high-level metrics through a join graph that resolves chasm and fan traps
- **Serves agents at execution** — exposes CLI and MCP tools with semantic search across wiki and semantic-layer entities

**Supported databases:** PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite

**Integrations:** dbt, MetricFlow, LookML, Looker, Metabase, Notion

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
- Creates or resumes a local ktx project
- Configures LLM and embedding providers
- Sets up database connections
- Configures context sources (dbt, Looker, Metabase, Notion)
- Builds initial context
- Installs agent integration (MCP server)

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

**Commit:** `ktx.yaml`, `semantic-layer/`, `wiki/`  
**Ignore:** `.ktx/` (contains secrets and local state)

## Key Commands

### Status and Health

```bash
# Check project readiness
ktx status

# Validate configuration without building
ktx validate
```

### Building Context

```bash
# Ingest all configured connections
ktx ingest

# Ingest specific connection
ktx ingest --connection-id warehouse

# Force rebuild
ktx ingest --force

# Dry run to see what would be ingested
ktx ingest --dry-run
```

### Searching Context

```bash
# Search semantic layer (metrics, dimensions, entities)
ktx sl "monthly recurring revenue"
ktx sl "customer churn rate"

# Search wiki pages
ktx wiki "refund policy"
ktx wiki "data retention rules"

# Full-text search with semantic ranking
ktx search "revenue recognition rules"
```

### MCP Server (Agent Integration)

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Check MCP server status
ktx mcp status
```

### Semantic Layer Management

```bash
# List all semantic sources
ktx sl list

# Show specific metric details
ktx sl show --entity-id mrr_metric

# Validate semantic layer definitions
ktx sl validate
```

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
  api_key_env: ANTHROPIC_API_KEY

embeddings:
  provider: openai
  model: text-embedding-3-small
  api_key_env: OPENAI_API_KEY

connections:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
    username_env: DB_USERNAME
    password_env: DB_PASSWORD
    read_only: true

context_sources:
  dbt_main:
    type: dbt
    connection_id: warehouse
    manifest_path: ./target/manifest.json
    run_results_path: ./target/run_results.json
  
  notion_docs:
    type: notion
    api_key_env: NOTION_API_KEY
    database_id_env: NOTION_DATABASE_ID
```

### Environment Variables

```bash
# LLM providers
export ANTHROPIC_API_KEY=your-key-here
export OPENAI_API_KEY=your-key-here
export GOOGLE_CLOUD_PROJECT=your-project

# Database connections
export DB_USERNAME=readonly_user
export DB_PASSWORD=secure-password

# Context sources
export NOTION_API_KEY=secret_xxx
export NOTION_DATABASE_ID=xxx

# ktx project location (optional)
export KTX_PROJECT_DIR=/path/to/project
```

## Real-World Usage Patterns

### Pattern 1: Setting Up for Analytics Team

```typescript
// After npm install -g @kaelio/ktx
// Run setup wizard
import { spawn } from 'child_process';

const setupKtx = () => {
  const ktx = spawn('ktx', ['setup'], { stdio: 'inherit' });
  
  ktx.on('close', (code) => {
    if (code === 0) {
      console.log('ktx setup complete');
      // Start MCP server for agents
      spawn('ktx', ['mcp', 'start'], { stdio: 'inherit' });
    }
  });
};
```

### Pattern 2: Automated Context Ingestion

```typescript
import { execSync } from 'child_process';

// Daily context refresh script
const refreshContext = () => {
  try {
    // Validate configuration
    execSync('ktx validate', { stdio: 'inherit' });
    
    // Ingest all sources
    execSync('ktx ingest', { stdio: 'inherit' });
    
    // Check status
    const status = execSync('ktx status', { encoding: 'utf-8' });
    console.log('Context refresh complete:', status);
  } catch (error) {
    console.error('Context refresh failed:', error);
    process.exit(1);
  }
};

refreshContext();
```

### Pattern 3: Programmatic Semantic Layer Query

```typescript
import { execSync } from 'child_process';

// Search for metrics programmatically
const findMetrics = (query: string) => {
  const result = execSync(`ktx sl "${query}" --json`, { encoding: 'utf-8' });
  return JSON.parse(result);
};

// Usage
const revenueMetrics = findMetrics('revenue');
console.log('Found metrics:', revenueMetrics.map(m => m.name));
```

### Pattern 4: Agent Integration Check

```typescript
import { execSync } from 'child_process';
import * as fs from 'fs';

// Verify ktx is ready for agent use
const verifyKtxReady = (projectDir: string) => {
  try {
    const status = execSync('ktx status', {
      cwd: projectDir,
      encoding: 'utf-8'
    });
    
    const checks = {
      projectReady: status.includes('Project ready: yes'),
      llmReady: status.includes('LLM ready: yes'),
      contextBuilt: status.includes('ktx context built: yes'),
      agentReady: status.includes('Agent integration ready: yes')
    };
    
    return Object.values(checks).every(v => v);
  } catch (error) {
    return false;
  }
};

if (!verifyKtxReady('./')) {
  console.error('ktx not ready - run: ktx setup');
  process.exit(1);
}
```

### Pattern 5: Custom Wiki Page Creation

```typescript
import * as fs from 'fs';
import * as path from 'path';

// Add business context to ktx wiki
const addWikiPage = (
  projectDir: string,
  title: string,
  content: string,
  global: boolean = true
) => {
  const wikiDir = global 
    ? path.join(projectDir, 'wiki', 'global')
    : path.join(projectDir, 'wiki', 'user', process.env.USER || 'default');
  
  fs.mkdirSync(wikiDir, { recursive: true });
  
  const filename = title.toLowerCase().replace(/\s+/g, '-') + '.md';
  const filepath = path.join(wikiDir, filename);
  
  const markdown = `---
title: ${title}
created: ${new Date().toISOString()}
---

# ${title}

${content}
`;
  
  fs.writeFileSync(filepath, markdown);
  console.log(`Wiki page created: ${filepath}`);
};

// Usage
addWikiPage('./', 'Revenue Recognition Policy', `
## Policy

Revenue is recognized when service is delivered, not when payment is received.

## Implementation

- Use the \`revenue_recognized_at\` timestamp
- Join with \`subscription_events\` table
- Filter by \`event_type = 'service_delivered'\`
`);
```

## Semantic Layer YAML

ktx builds semantic layers automatically, but you can also define custom metrics:

```yaml
# semantic-layer/warehouse/metrics.yaml
version: 1
metrics:
  - id: mrr
    name: Monthly Recurring Revenue
    description: Total monthly recurring revenue from active subscriptions
    type: metric
    sql: |
      SELECT 
        DATE_TRUNC('month', subscription_start) as month,
        SUM(monthly_value) as mrr
      FROM subscriptions
      WHERE status = 'active'
      GROUP BY 1
    dimensions:
      - month
    measures:
      - mrr
    joins:
      - entity: customers
        on: subscriptions.customer_id = customers.id
    tags:
      - revenue
      - subscription
```

## Common Troubleshooting

### "ktx context not built"

```bash
# Run ingestion to build context
ktx ingest

# Check for errors
ktx validate
```

### "LLM ready: no"

```bash
# Set API key
export ANTHROPIC_API_KEY=your-key

# Or configure in ktx.yaml
ktx setup
```

### "Agent integration not ready"

```bash
# Ensure MCP server is running
ktx mcp start --project-dir .

# Check MCP status
ktx mcp status
```

### Database Connection Fails

```bash
# Test connection independently
ktx ingest --connection-id warehouse --dry-run

# Verify read-only access
# ktx never writes to your database
```

### Semantic Layer Not Found

```bash
# List available sources
ktx sl list

# Rebuild semantic layer
ktx ingest --connection-id warehouse --force
```

## Integration with AI Agents

### Claude Code / Codex / Cursor

From your project directory, tell the agent:

```
Run npx skills add Kaelio/ktx --skill ktx and use the ktx skill to install 
and configure ktx in this project.
```

Or manually:

1. Run `ktx setup` in your project
2. Ensure `ktx status` shows all ready
3. Start the agent - it will detect the MCP server automatically
4. Ask: "What metrics are available?" or "Query revenue by month"

### MCP Tools Available to Agents

When ktx MCP server is running, agents get these tools:

- `ktx_search_semantic_layer` — search metrics, dimensions, entities
- `ktx_search_wiki` — search business context and documentation
- `ktx_get_metric` — get detailed metric definition
- `ktx_list_connections` — list available database connections
- `ktx_query_warehouse` — execute read-only SQL queries

## Advanced Configuration

### Multiple Connections

```yaml
connections:
  prod_warehouse:
    type: snowflake
    account: company.us-east-1
    warehouse: ANALYTICS_WH
    database: PROD
    schema: PUBLIC
    username_env: SNOWFLAKE_USER
    password_env: SNOWFLAKE_PASSWORD
  
  staging_warehouse:
    type: postgres
    host: staging-db.internal
    port: 5432
    database: staging
    username_env: STAGING_DB_USER
    password_env: STAGING_DB_PASSWORD
```

### Custom LLM Backends

```yaml
llm:
  # Anthropic API
  provider: anthropic
  model: claude-sonnet-4-6
  api_key_env: ANTHROPIC_API_KEY

  # Google Vertex AI
  # provider: vertex
  # model: claude-sonnet-4
  # project_env: GOOGLE_CLOUD_PROJECT

  # AI Gateway
  # provider: ai_gateway
  # api_url_env: AI_GATEWAY_URL
  # api_key_env: AI_GATEWAY_KEY
```

### Telemetry Opt-Out

```bash
# Disable anonymous usage telemetry
export KTX_TELEMETRY_DISABLED=1

# Or in ktx.yaml
telemetry:
  enabled: false
```

## Best Practices

1. **Keep .ktx/ out of version control** — it contains secrets and local state
2. **Commit semantic-layer/ and wiki/** — share context across team
3. **Run ktx ingest regularly** — daily or after schema changes
4. **Use read-only database users** — ktx never writes, enforce at DB level
5. **Store API keys in environment variables** — never commit to ktx.yaml
6. **Start MCP server before opening agent** — check `ktx status` output
7. **Review contradiction flags** — when ktx finds conflicting definitions
8. **Tag metrics consistently** — helps agents find relevant context

## Resources

- [Documentation](https://docs.kaelio.com/ktx)
- [CLI Reference](https://docs.kaelio.com/ktx/docs/cli-reference/ktx)
- [Agent Quickstart](https://docs.kaelio.com/ktx/docs/ai-resources/agent-quickstart)
- [Slack Community](https://join.slack.com/t/ktxcommunity/shared_invite/zt-3y9b44m1x-LVyNNJD5nwaZHq4XS29LMQ)
- [GitHub Issues](https://github.com/Kaelio/ktx/issues)
