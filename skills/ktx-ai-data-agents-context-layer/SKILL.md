---
name: ktx-ai-data-agents-context-layer
description: Install and configure ktx, the self-improving context layer that teaches AI agents how to accurately query data warehouses with approved metrics, semantic layers, and business knowledge
triggers:
  - set up ktx for data analysis
  - configure ktx semantic layer
  - install ktx context layer for database queries
  - help me use ktx with my data warehouse
  - integrate ktx with Claude for analytics
  - build ktx context from dbt and warehouse
  - query my database using ktx semantic layer
  - configure ktx MCP server for agents
---

# ktx AI Data Agents Context Layer

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## What ktx Does

ktx is an executable context layer for data and analytics agents that teaches AI agents how to query your data warehouse accurately. It:

- **Auto-builds warehouse context** by sampling tables, capturing metadata, detecting joinable columns, and annotating sources
- **Creates a semantic layer** combining raw tables and high-level metrics through a join graph that resolves chasm and fan traps
- **Ingests company knowledge** from dbt, Looker, Metabase, Notion, and wikis, organizing it and flagging contradictions
- **Serves agents via MCP** with full-text and semantic search across wiki and semantic-layer entities
- **Enables accurate queries** by providing approved metric definitions instead of letting agents invent SQL logic

Works with PostgreSQL, Snowflake, BigQuery, ClickHouse, MySQL, SQL Server, SQLite, dbt, MetricFlow, LookML, Looker, Metabase, and Notion.

## Installation

### Global Installation

```bash
npm install -g @kaelio/ktx
```

### Project-Specific Installation

```bash
npm install @kaelio/ktx
# Or use directly with npx
npx @kaelio/ktx --help
```

## Initial Setup

### Interactive Setup

```bash
# Creates/resumes project, configures providers, builds context, installs agent integration
ktx setup
```

This command:
1. Creates `ktx.yaml` configuration file
2. Prompts for LLM provider (Anthropic, Google Vertex AI, AI Gateway, or Claude Code SDK)
3. Prompts for embedding provider (OpenAI, Voyage AI, Google, or Claude)
4. Configures database connections
5. Configures context sources (dbt, Looker, Metabase, Notion)
6. Runs initial ingestion
7. Installs MCP integration for your agent

### Check Project Status

```bash
ktx status
```

Expected output after successful setup:

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

**Commit:** `ktx.yaml`, `semantic-layer/`, `wiki/`  
**Ignore:** `.ktx/`

## Configuration

### ktx.yaml Structure

```yaml
version: 1
name: my-analytics-project

llm:
  provider: anthropic
  model: claude-sonnet-4-6

embeddings:
  provider: openai
  model: text-embedding-3-small

databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    # Credentials stored in .ktx/secrets.yaml

sources:
  dbt_main:
    type: dbt
    path: ./dbt-project
    profiles_dir: ~/.dbt
  
  looker_models:
    type: lookml
    path: ./looker-models
```

### LLM Provider Configuration

#### Anthropic API

```bash
export ANTHROPIC_API_KEY=your_key_here
```

```yaml
llm:
  provider: anthropic
  model: claude-sonnet-4-6
```

#### Google Vertex AI

```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
export GOOGLE_CLOUD_PROJECT=your-project-id
export GOOGLE_CLOUD_REGION=us-central1
```

```yaml
llm:
  provider: vertex
  model: claude-sonnet-4-6@20250514
```

#### Claude Code SDK (when running in Claude Code)

```yaml
llm:
  provider: claude-code-sdk
```

### Embedding Provider Configuration

#### OpenAI

```bash
export OPENAI_API_KEY=your_key_here
```

```yaml
embeddings:
  provider: openai
  model: text-embedding-3-small
```

#### Voyage AI

```bash
export VOYAGE_API_KEY=your_key_here
```

```yaml
embeddings:
  provider: voyage
  model: voyage-3
```

### Database Connection Examples

#### PostgreSQL

```yaml
databases:
  warehouse:
    type: postgres
    host: localhost
    port: 5432
    database: analytics
    schema: public
```

Store credentials in `.ktx/secrets.yaml`:

```yaml
databases:
  warehouse:
    user: ${DB_USER}
    password: ${DB_PASSWORD}
```

#### Snowflake

```yaml
databases:
  snowflake_prod:
    type: snowflake
    account: xy12345.us-east-1
    warehouse: COMPUTE_WH
    database: ANALYTICS
    schema: PUBLIC
```

#### BigQuery

```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

```yaml
databases:
  bigquery_prod:
    type: bigquery
    project: my-gcp-project
    dataset: analytics
```

## Key Commands

### Building Context

```bash
# Ingest all configured connections
ktx ingest

# Ingest specific connection
ktx ingest --connection warehouse

# Ingest with verbose output
ktx ingest --verbose
```

### Searching Context

#### Search Semantic Layer

```bash
# Search for metrics, dimensions, entities
ktx sl "revenue"
ktx sl "monthly active users"
ktx sl "customer churn rate"

# With connection filter
ktx sl "revenue" --connection warehouse
```

#### Search Wiki

```bash
# Search business knowledge
ktx wiki "refund policy"
ktx wiki "revenue recognition"
ktx wiki "customer segmentation"

# Scope to user wiki
ktx wiki "my analysis notes" --scope user
```

### MCP Server

```bash
# Start MCP server for agent clients
ktx mcp start

# Start with specific project directory
ktx mcp start --project-dir /path/to/project

# Start with port override
ktx mcp start --port 3000
```

### Project Management

```bash
# Initialize new project
ktx init

# Validate configuration
ktx validate

# Clean and rebuild context
ktx clean
ktx ingest

# Update to latest ktx version
npm update -g @kaelio/ktx
```

## Agent Integration

### Claude Code / Codex

After running `ktx setup`, the MCP integration is automatically configured. Ensure the MCP server is running:

```bash
ktx mcp start --project-dir /path/to/your/project
```

Then in Claude Code or Codex, ktx tools are available:

- `search_semantic_layer` - Search metrics and dimensions
- `search_wiki` - Search business knowledge
- `get_semantic_source` - Get details of a specific metric
- `get_wiki_page` - Get full wiki page content

### Using ktx from Agents

Example prompt for Claude Code:

```text
Use ktx to find our revenue metrics, then write a query to calculate 
monthly revenue for the last 6 months broken down by product category.
```

Example prompt for Cursor:

```text
Search ktx wiki for our customer segmentation rules, then build a query 
that identifies high-value customers based on those criteria.
```

## Code Examples

### TypeScript: Programmatic Usage

```typescript
import { KtxProject } from '@kaelio/ktx';

// Load project
const project = await KtxProject.load('/path/to/project');

// Search semantic layer
const metrics = await project.searchSemanticLayer('revenue', {
  limit: 10,
  connection: 'warehouse'
});

console.log(metrics.results);

// Search wiki
const wikiResults = await project.searchWiki('refund policy', {
  scope: 'global',
  limit: 5
});

console.log(wikiResults.results);

// Get specific semantic source
const revenueMetric = await project.getSemanticSource(
  'warehouse',
  'metric',
  'total_revenue'
);

console.log(revenueMetric.definition);
```

### TypeScript: Custom Ingest Pipeline

```typescript
import { KtxProject, Ingester } from '@kaelio/ktx';

const project = await KtxProject.load('./my-project');

// Create custom ingester
const ingester = new Ingester(project, {
  connections: ['warehouse'],
  verbose: true
});

// Run ingestion with hooks
await ingester.run({
  onTableScanned: (table) => {
    console.log(`Scanned: ${table.name}`);
  },
  onMetricFound: (metric) => {
    console.log(`Found metric: ${metric.name}`);
  },
  onComplete: (stats) => {
    console.log(`Ingested ${stats.tables} tables, ${stats.metrics} metrics`);
  }
});
```

### TypeScript: Build Semantic Source

```typescript
import { SemanticSource } from '@kaelio/ktx';
import * as yaml from 'yaml';
import * as fs from 'fs';

// Define a metric
const revenueMetric: SemanticSource = {
  type: 'metric',
  name: 'total_revenue',
  label: 'Total Revenue',
  description: 'Sum of all invoice amounts',
  sql: 'SUM(invoices.amount)',
  dimensions: ['customer_id', 'product_id', 'invoice_date'],
  filters: {
    valid_only: 'invoices.status = \'paid\''
  },
  meta: {
    owner: 'finance-team',
    approved: true
  }
};

// Write to semantic layer
const yamlContent = yaml.stringify(revenueMetric);
fs.writeFileSync(
  './semantic-layer/warehouse/total_revenue.yaml',
  yamlContent
);
```

### Python: Query Planning with ktx-sl

```python
from ktx_sl import QueryPlanner, SemanticLayer

# Load semantic layer
sl = SemanticLayer.from_directory("./semantic-layer/warehouse")

# Create query planner
planner = QueryPlanner(sl)

# Plan a query
query = planner.build_query(
    metrics=["total_revenue", "order_count"],
    dimensions=["customer_segment", "month"],
    filters={
        "order_date": "BETWEEN '2024-01-01' AND '2024-12-31'"
    }
)

print(query.sql)
print(query.join_path)
```

## Common Patterns

### Pattern 1: dbt + ktx Integration

```yaml
# ktx.yaml
sources:
  dbt_analytics:
    type: dbt
    path: ./dbt-project
    profiles_dir: ~/.dbt
    manifest_path: ./dbt-project/target/manifest.json
```

```bash
# Build dbt, then ingest into ktx
dbt build
ktx ingest --connection dbt_analytics
```

ktx automatically:
- Discovers dbt models, metrics, and tests
- Maps column lineage
- Imports metric definitions
- Detects joinable columns

### Pattern 2: Wiki-Driven Analytics

Create wiki pages in `wiki/global/`:

```markdown
<!-- wiki/global/revenue-recognition.md -->
# Revenue Recognition Policy

Revenue is recognized when:
- Invoice is marked as 'paid'
- Payment date is within fiscal period
- No active disputes

## Key Metrics

- `total_revenue`: SUM(invoices.amount) WHERE status = 'paid'
- `arr`: Annual Recurring Revenue (subscription revenue × 12)
```

```bash
# Ingest wiki knowledge
ktx ingest

# Search from agent
ktx wiki "revenue recognition"
```

### Pattern 3: Multi-Warehouse Setup

```yaml
# ktx.yaml
databases:
  warehouse_prod:
    type: snowflake
    account: prod.us-east-1
    database: ANALYTICS
  
  warehouse_dev:
    type: postgres
    host: localhost
    database: analytics_dev

sources:
  dbt_prod:
    type: dbt
    path: ./dbt-project
    target: prod
    connection: warehouse_prod
  
  dbt_dev:
    type: dbt
    path: ./dbt-project
    target: dev
    connection: warehouse_dev
```

```bash
# Ingest both
ktx ingest

# Query specific warehouse from agent
ktx sl "revenue" --connection warehouse_prod
```

### Pattern 4: Continuous Context Updates

```bash
#!/bin/bash
# scheduled-ingest.sh

cd /path/to/ktx-project

# Pull latest dbt changes
git pull origin main

# Build dbt
dbt build --target prod

# Ingest into ktx
ktx ingest --verbose

# Commit updated semantic layer
git add semantic-layer/ wiki/
git commit -m "chore: update ktx context $(date +%Y-%m-%d)"
git push origin main
```

Schedule with cron:

```cron
0 2 * * * /path/to/scheduled-ingest.sh
```

## Troubleshooting

### MCP Server Won't Start

**Issue:** `ktx mcp start` fails or agent can't connect

**Solutions:**

```bash
# Check status first
ktx status

# If it says to run mcp start command, copy and run it exactly
ktx mcp start --project-dir /path/shown/in/status

# Verify project is valid
ktx validate

# Check logs
ktx mcp start --verbose
```

### Ingestion Fails

**Issue:** `ktx ingest` errors or hangs

**Solutions:**

```bash
# Check database connection
ktx validate --connection warehouse

# Run with verbose output
ktx ingest --verbose

# Try single connection
ktx ingest --connection warehouse

# Check credentials in .ktx/secrets.yaml
cat .ktx/secrets.yaml

# Test database connectivity manually
psql -h localhost -U user -d analytics -c "SELECT 1"
```

### LLM API Errors

**Issue:** "API key not found" or rate limit errors

**Solutions:**

```bash
# Verify environment variables
echo $ANTHROPIC_API_KEY
echo $OPENAI_API_KEY

# Check ktx.yaml configuration
cat ktx.yaml | grep -A 3 "llm:"

# Test with explicit provider
ktx setup --llm-provider anthropic

# For Claude Code SDK, ensure running in Claude Code context
# Check provider is set to claude-code-sdk in ktx.yaml
```

### Search Returns No Results

**Issue:** `ktx sl` or `ktx wiki` returns empty

**Solutions:**

```bash
# Verify context was built
ktx status

# Rebuild context
ktx ingest

# Check semantic layer files exist
ls -la semantic-layer/

# Check wiki files exist
ls -la wiki/global/

# Search with verbose flag
ktx sl "revenue" --verbose
```

### Join Path Errors in Queries

**Issue:** Agent-generated queries fail due to invalid joins

**Solutions:**

1. Check join graph detection:

```bash
ktx ingest --verbose | grep "detected join"
```

2. Manually define joins in semantic source:

```yaml
# semantic-layer/warehouse/orders.yaml
type: dimension
name: orders
primary_key: order_id
relationships:
  - entity: customers
    type: many_to_one
    join_keys:
      - order.customer_id = customer.customer_id
```

3. Review ingestion report:

```bash
cat raw-sources/warehouse/ingestion-report.json
```

### Performance Issues

**Issue:** Queries or searches are slow

**Solutions:**

```bash
# Use smaller embedding model
# In ktx.yaml:
embeddings:
  provider: openai
  model: text-embedding-3-small  # Instead of text-embedding-3-large

# Limit ingestion scope
ktx ingest --connection warehouse --tables "sales_*,revenue_*"

# Clean and rebuild indexes
ktx clean
ktx ingest

# Check vector database size
du -sh .ktx/vector-store/
```

## Advanced Usage

### Custom Semantic Sources

Create YAML files in `semantic-layer/<connection>/`:

```yaml
# semantic-layer/warehouse/monthly_recurring_revenue.yaml
type: metric
name: monthly_recurring_revenue
label: Monthly Recurring Revenue (MRR)
description: Sum of active subscription values normalized to monthly
sql: |
  SUM(
    CASE subscription_interval
      WHEN 'monthly' THEN amount
      WHEN 'yearly' THEN amount / 12
    END
  )
dimensions:
  - customer_id
  - subscription_plan
  - month
filters:
  active_only: status = 'active'
  exclude_trials: is_trial = false
meta:
  owner: revenue-team
  approved_by: finance
  last_validated: 2024-05-15
```

### Programmatic Wiki Updates

```typescript
import { KtxProject, WikiPage } from '@kaelio/ktx';
import * as fs from 'fs';
import * as path from 'path';

const project = await KtxProject.load('./my-project');

// Create wiki page programmatically
const page: WikiPage = {
  title: 'Customer Segmentation',
  content: `
# Customer Segmentation

## Segments

### High Value
- Total revenue > $10,000
- Active for > 12 months

### Growth
- Revenue $1,000-$10,000
- Active for 3-12 months

### New
- Active for < 3 months
  `,
  scope: 'global',
  tags: ['customers', 'segmentation', 'analysis']
};

// Write to wiki
const wikiPath = path.join(
  project.projectDir,
  'wiki/global/customer-segmentation.md'
);
fs.writeFileSync(wikiPath, page.content);

// Re-ingest to index
await project.ingest();
```

### Environment-Specific Configuration

```yaml
# ktx.yaml (base configuration)
version: 1
name: analytics-project

# Use environment variables for sensitive values
databases:
  warehouse:
    type: postgres
    host: ${DB_HOST}
    port: ${DB_PORT:-5432}
    database: ${DB_NAME}
```

```bash
# .env.production
DB_HOST=prod-warehouse.example.com
DB_PORT=5432
DB_NAME=analytics_prod

# .env.development
DB_HOST=localhost
DB_PORT=5433
DB_NAME=analytics_dev
```

```bash
# Load environment and run
source .env.production
ktx ingest
```

## Best Practices

1. **Version control** - Commit `ktx.yaml`, `semantic-layer/`, and `wiki/`. Ignore `.ktx/`
2. **Keep wiki updated** - Document business logic changes in wiki pages
3. **Validate metrics** - Use `meta.approved: true` for production metrics
4. **Regular ingestion** - Schedule `ktx ingest` to keep context fresh
5. **Read-only connections** - Use database users with SELECT-only privileges
6. **Environment separation** - Use different ktx projects for dev/staging/prod
7. **Test semantic sources** - Validate generated SQL before marking metrics approved
