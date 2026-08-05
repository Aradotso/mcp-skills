---
name: postgres-mcp-server
description: MCP server for PostgreSQL databases enabling LLMs to query and analyze Postgres through a controlled interface
triggers:
  - query postgres database
  - analyze postgresql data
  - connect to postgres with mcp
  - setup postgres mcp server
  - run sql queries through mcp
  - inspect postgres database schema
  - postgres database integration
  - mcp postgres tools
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol server that provides LLMs with controlled access to PostgreSQL databases. Enables SQL queries, schema inspection, and database analysis through MCP tools and resources.

## Installation

### Quick Install (via MCP Client)

Add to your MCP client configuration (e.g., Claude Desktop, Cursor):

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

### Local Development Installation

```bash
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install
bun run build:js
```

For local build in MCP client:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": ["/absolute/path/to/pg-mcp-server/lib/index.js", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://postgres:postgres@localhost:5432/postgres"
      }
    }
  }
}
```

## Configuration

### Environment Variables

- `DATABASE_URL` (required): PostgreSQL connection string
  ```
  postgresql://username:password@host:port/database
  ```

- `DANGEROUSLY_ALLOW_WRITE_OPS` (optional, default: `false`): Enable INSERT, UPDATE, DELETE operations
  ```bash
  DANGEROUSLY_ALLOW_WRITE_OPS=true
  ```

- `DEBUG` (optional, default: `false`): Enable debug logging
  ```bash
  DEBUG=true
  ```

- `PG_SSL_ROOT_CERT` (optional): Path to TLS CA bundle for SSL connections
  ```bash
  PG_SSL_ROOT_CERT=/path/to/rds-combined-ca-bundle.pem
  ```

### Transport Modes

**stdio** (default): Standard input/output for MCP client integration
```bash
pg-mcp-server --transport=stdio
```

**http**: HTTP server on port 3000 (or $PORT) at `/mcp` endpoint
```bash
pg-mcp-server --transport=http
PORT=8080 pg-mcp-server --transport=http
```

## Available Tools

### `query` - Execute SQL Queries

Execute SELECT queries (and writes if enabled) against the database.

**Parameters:**
- `sql` (string, required): SQL query to execute

**Example Usage:**

```typescript
// Read operations (always allowed)
{
  "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
}

{
  "sql": "SELECT COUNT(*) as total, status FROM orders GROUP BY status"
}

{
  "sql": "SELECT u.name, COUNT(o.id) as order_count FROM users u LEFT JOIN orders o ON u.id = o.user_id GROUP BY u.id, u.name ORDER BY order_count DESC LIMIT 5"
}

// Write operations (requires DANGEROUSLY_ALLOW_WRITE_OPS=true)
{
  "sql": "INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com')"
}

{
  "sql": "UPDATE products SET price = price * 1.1 WHERE category = 'electronics'"
}
```

## Available Resources

### `postgres://tables` - List All Tables

Lists all tables in the database with their schemas.

**Example Response:**
```json
{
  "tables": [
    {
      "schema": "public",
      "name": "users",
      "type": "table"
    },
    {
      "schema": "public",
      "name": "orders",
      "type": "table"
    }
  ]
}
```

### `postgres://table/{schema}/{table}` - Get Table Details

Retrieves schema definition and sample data for a specific table.

**Example:**
- `postgres://table/public/users`
- `postgres://table/analytics/events`

**Example Response:**
```json
{
  "schema": {
    "columns": [
      {
        "name": "id",
        "type": "integer",
        "nullable": false,
        "default": "nextval('users_id_seq'::regclass)"
      },
      {
        "name": "email",
        "type": "character varying",
        "nullable": false
      }
    ]
  },
  "sample_data": [
    { "id": 1, "email": "alice@example.com" },
    { "id": 2, "email": "bob@example.com" }
  ]
}
```

## Common Patterns

### Data Analysis Workflow

```typescript
// 1. List all available tables
// Use resource: postgres://tables

// 2. Inspect table structure
// Use resource: postgres://table/public/users

// 3. Query specific data
{
  "sql": "SELECT DATE_TRUNC('day', created_at) as day, COUNT(*) as signups FROM users WHERE created_at >= NOW() - INTERVAL '30 days' GROUP BY day ORDER BY day"
}
```

### Safe Query Patterns

```typescript
// Always use LIMIT for exploratory queries
{
  "sql": "SELECT * FROM large_table LIMIT 100"
}

// Use WHERE clauses to filter data
{
  "sql": "SELECT * FROM orders WHERE created_at >= '2024-01-01' AND status = 'completed'"
}

// Aggregate before returning large datasets
{
  "sql": "SELECT category, COUNT(*) as count, AVG(price) as avg_price FROM products GROUP BY category"
}
```

### Join Queries

```typescript
{
  "sql": "SELECT u.name, u.email, o.order_date, o.total FROM users u INNER JOIN orders o ON u.id = o.user_id WHERE o.order_date >= CURRENT_DATE - INTERVAL '7 days' ORDER BY o.order_date DESC"
}
```

### Complex Analysis

```typescript
{
  "sql": "WITH monthly_revenue AS (SELECT DATE_TRUNC('month', order_date) as month, SUM(total) as revenue FROM orders GROUP BY month) SELECT month, revenue, LAG(revenue) OVER (ORDER BY month) as prev_month, ROUND(((revenue - LAG(revenue) OVER (ORDER BY month)) / LAG(revenue) OVER (ORDER BY month) * 100)::numeric, 2) as growth_pct FROM monthly_revenue ORDER BY month DESC"
}
```

## Quick Start with Docker

The project includes a Docker setup with sample data for testing:

```bash
# Start PostgreSQL with sample database
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample tables included:
- `users`: User accounts
- `products`: Product catalog
- `orders`: Order records
- `order_items`: Order line items

## Development

### Running Locally

```bash
# stdio transport
bun run index.ts -- --transport=stdio
DEBUG=true bun run index.ts -- --transport=stdio

# http transport
bun run index.ts -- --transport=http
DEBUG=true bun run index.ts -- --transport=http
```

### Building

```bash
# Build JavaScript output
bun run build:js

# Run tests
bun test
```

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or "ECONNREFUSED"

**Solution:** Verify `DATABASE_URL` is correct and PostgreSQL is running:
```bash
psql $DATABASE_URL -c "SELECT version();"
```

### SSL/TLS Errors

**Problem:** "certificate verify failed" or SSL errors

**Solution:** Set `PG_SSL_ROOT_CERT` for managed databases (e.g., AWS RDS):
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "PG_SSL_ROOT_CERT": "/path/to/rds-combined-ca-bundle.pem"
  }
}
```

### Write Operations Blocked

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

**Warning:** Only enable writes in development or with read-only database users.

### Query Timeouts

**Problem:** Large queries hang or timeout

**Solution:** Add timeouts to your connection string:
```
postgresql://user:pass@host:5432/db?statement_timeout=30000&connect_timeout=10
```

### Debugging

Enable debug logging to see query execution:
```bash
DEBUG=true pg-mcp-server --transport=stdio
```

Or in MCP client config:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DEBUG": "true"
  }
}
```

## Example Prompts for AI Agents

When using this MCP server with an AI coding agent:

- "Show me the first 5 users from the database"
- "What tables are available in this database?"
- "Analyze the orders table and show me sales by month"
- "Find all products with price greater than $100"
- "Show me the schema for the users table"
- "Calculate the average order value for the last 30 days"
- "List the top 10 customers by total order value"

## Security Best Practices

1. **Use read-only database users** when possible
2. **Never enable `DANGEROUSLY_ALLOW_WRITE_OPS`** in production
3. **Limit database user permissions** to only necessary tables/schemas
4. **Use SSL/TLS** for database connections in production
5. **Store `DATABASE_URL`** securely (never commit to version control)
6. **Use connection pooling** for high-traffic scenarios
