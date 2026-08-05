---
name: postgres-mcp-server
description: MCP server that enables LLMs to query and analyze PostgreSQL databases through a controlled interface with read/write capabilities.
triggers:
  - query my postgres database
  - analyze data in postgresql
  - get table schema from postgres
  - execute sql query through mcp
  - list all postgres tables
  - connect to postgresql with mcp
  - run database queries with llm
  - inspect postgres database structure
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Postgres MCP Server is a Model Context Protocol server that provides LLMs with controlled access to PostgreSQL databases. It supports querying, schema inspection, and optional write operations, making it ideal for database analysis, reporting, and data exploration tasks.

## Installation

### Using npx (Recommended)

Add to your MCP client settings (e.g., `~/Library/Application Support/Claude/claude_desktop_config.json` for Claude Desktop):

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://user:password@localhost:5432/dbname"
      }
    }
  }
}
```

### Local Development Build

```bash
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install
bun run build:js
```

Then reference the local build:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": ["/absolute/path/to/pg-mcp-server/lib/index.js", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

## Configuration

### Environment Variables

- **`DATABASE_URL`** (required): PostgreSQL connection string
  - Format: `postgresql://user:password@host:port/database`
  - Example: `postgresql://postgres:postgres@localhost:5432/myapp`

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (optional): Enable INSERT, UPDATE, DELETE operations
  - Default: `false`
  - Set to `true` to allow write operations

- **`DEBUG`** (optional): Enable debug logging
  - Default: `false`
  - Set to `true` for verbose logging

- **`PG_SSL_ROOT_CERT`** (optional): Path to TLS CA bundle for SSL connections
  - Example: `/path/to/rds-combined-ca-bundle.pem`

### Transport Modes

The server supports two transport modes:

**stdio (default)**: For MCP clients like Claude Desktop
```bash
pg-mcp-server --transport=stdio
```

**http**: Serves MCP Streamable HTTP endpoint at `/mcp`
```bash
pg-mcp-server --transport=http
# Server runs on PORT (default: 3000)
# Connect to: http://localhost:3000/mcp
```

## Available Tools

### `query` Tool

Execute SQL queries against the connected PostgreSQL database.

**Input Schema:**
```typescript
{
  sql: string  // SQL query to execute
}
```

**Example Usage:**
```json
{
  "sql": "SELECT id, email, created_at FROM users WHERE active = true LIMIT 10"
}
```

**Safety Features:**
- Read-only by default (SELECT, WITH queries only)
- Write operations (INSERT, UPDATE, DELETE) require `DANGEROUSLY_ALLOW_WRITE_OPS=true`
- Dangerous operations (DROP, TRUNCATE, ALTER) are blocked regardless of settings

## Available Resources

### List All Tables

**Resource URI:** `postgres://tables`

Returns a list of all tables in the database with schema information.

**Example Response:**
```
Available tables:
- public.users
- public.products
- public.orders
- public.order_items
```

### Get Table Details

**Resource URI:** `postgres://table/{schema}/{table}`

Returns detailed schema information and sample data for a specific table.

**Example:** `postgres://table/public/users`

**Response includes:**
- Column names and data types
- Primary keys
- Foreign keys
- Constraints
- Sample rows (first 5 records)

## Common Usage Patterns

### Basic Data Query

Ask the AI assistant:
```
Show me the first 10 active users from the database
```

The AI will use the `query` tool:
```json
{
  "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
}
```

### Schema Exploration

Ask the AI assistant:
```
What tables are available in the database?
```

The AI will access the `postgres://tables` resource to list all tables.

### Table Analysis

Ask the AI assistant:
```
Show me the structure of the products table
```

The AI will access `postgres://table/public/products` to get schema and sample data.

### Aggregate Queries

Ask the AI assistant:
```
How many orders were placed each month in 2024?
```

The AI will construct an appropriate GROUP BY query:
```json
{
  "sql": "SELECT DATE_TRUNC('month', created_at) as month, COUNT(*) as order_count FROM orders WHERE EXTRACT(YEAR FROM created_at) = 2024 GROUP BY DATE_TRUNC('month', created_at) ORDER BY month"
}
```

### Join Queries

Ask the AI assistant:
```
Show me order totals by user
```

The AI will construct a JOIN query:
```json
{
  "sql": "SELECT u.email, COUNT(o.id) as order_count, SUM(oi.quantity * oi.price) as total_spent FROM users u LEFT JOIN orders o ON u.id = o.user_id LEFT JOIN order_items oi ON o.id = oi.order_id GROUP BY u.id, u.email ORDER BY total_spent DESC LIMIT 20"
}
```

## TypeScript Development

### Running Locally

```bash
# Clone the repository
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server

# Install dependencies
bun install

# Run with stdio transport
bun run index.ts -- --transport=stdio

# Run with debug logging
DEBUG=true bun run index.ts -- --transport=stdio

# Run with HTTP transport
bun run index.ts -- --transport=http

# Run tests
bun test
```

### Docker Development Environment

Start a PostgreSQL instance with sample data:

```bash
# Start PostgreSQL container with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample tables included:
- `users` - User accounts
- `products` - Product catalog
- `orders` - Order records
- `order_items` - Line items for orders

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or "ECONNREFUSED"

**Solution:** Verify your `DATABASE_URL` is correct and the PostgreSQL server is running:
```bash
psql "${DATABASE_URL}" -c "SELECT version();"
```

### SSL/TLS Errors

**Problem:** SSL connection errors with cloud databases (AWS RDS, etc.)

**Solution:** Download and specify the CA bundle:
```bash
# Download AWS RDS CA bundle
curl -o /path/to/rds-ca.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
```

Update configuration:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "PG_SSL_ROOT_CERT": "/path/to/rds-ca.pem"
  }
}
```

### Write Operations Not Working

**Problem:** INSERT/UPDATE/DELETE queries fail

**Solution:** Enable write operations explicitly:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

**Note:** This is disabled by default for safety.

### Permission Errors

**Problem:** "permission denied for table" errors

**Solution:** Ensure the database user has appropriate permissions:
```sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO your_user;
-- For write operations:
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO your_user;
```

### Large Result Sets

**Problem:** Queries timeout or return too much data

**Solution:** Always use LIMIT clauses for exploratory queries:
```sql
SELECT * FROM large_table LIMIT 100;
```

For large datasets, ask the AI to use pagination or aggregation.

## Security Best Practices

1. **Use read-only database users** when possible
2. **Enable write operations only when necessary** via `DANGEROUSLY_ALLOW_WRITE_OPS`
3. **Use connection pooling** for production environments
4. **Store credentials securely** using environment variables, never in code
5. **Limit network access** to the database (firewall rules, VPC)
6. **Use SSL/TLS** for connections to remote databases

## Example Workflows

### Data Analysis Notebook

The project includes a Cursor rule for data analysis workflows at `.cursor/rules/notebooks.mdc`. Use it to:

1. Explore database schema
2. Generate sample queries
3. Visualize query results
4. Document findings in markdown notebooks

### Database Health Check

Ask the AI:
```
Check the database health: show table sizes, row counts, and any missing indexes
```

### Generating Reports

Ask the AI:
```
Create a sales report for Q4 2024 showing revenue by product category
```

The AI will construct appropriate queries and format the results.
