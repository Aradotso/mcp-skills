---
name: postgres-mcp-server
description: MCP server for querying and analyzing PostgreSQL databases through LLMs with controlled SQL execution
triggers:
  - query postgres database
  - analyze postgresql tables
  - get database schema
  - execute sql query through mcp
  - inspect postgres table structure
  - connect to postgresql via mcp
  - run database analytics
  - explore postgres data
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Postgres MCP Server enables AI assistants to interact with PostgreSQL databases through the Model Context Protocol. It provides secure SQL query execution, schema inspection, and table browsing capabilities with optional write protection.

## Installation

### Quick Install (Cursor)

Use the one-click installer or add to your MCP client configuration:

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

```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": ["/absolute/path/to/pg-mcp-server/lib/index.js", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/mydb",
        "DANGEROUSLY_ALLOW_WRITE_OPS": "false",
        "DEBUG": "false"
      }
    }
  }
}
```

## Configuration

### Environment Variables

- **`DATABASE_URL`** (required): PostgreSQL connection string
  - Format: `postgresql://username:password@host:port/database`
  - Example: `postgresql://postgres:postgres@localhost:5432/myapp`
  
- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (optional, default: `false`): Enable INSERT/UPDATE/DELETE operations
  - Set to `"true"` to allow write operations
  - Keep `false` for read-only safety in production
  
- **`DEBUG`** (optional, default: `false`): Enable debug logging
  
- **`PG_SSL_ROOT_CERT`** (optional): Path to TLS CA bundle for SSL connections
  - Example: `/path/to/rds-ca-bundle.pem` for AWS RDS

### Transport Modes

**stdio** (default): Standard input/output communication
```bash
pg-mcp-server --transport=stdio
```

**http**: Streamable HTTP endpoint at `/mcp`
```bash
pg-mcp-server --transport=http
# Serves at http://localhost:3000/mcp (PORT env var configurable)
```

## Available Tools

### `query` Tool

Execute SQL queries against the connected database.

**Request:**
```typescript
{
  "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
}
```

**Response:**
```typescript
{
  "rows": [
    { "id": 1, "name": "Alice", "active": true },
    { "id": 2, "name": "Bob", "active": true }
  ],
  "rowCount": 2
}
```

**Read-only example:**
```sql
SELECT 
  u.id,
  u.name,
  COUNT(o.id) as order_count,
  SUM(oi.quantity * p.price) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
LEFT JOIN order_items oi ON o.id = oi.order_id
LEFT JOIN products p ON oi.product_id = p.id
WHERE u.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.id, u.name
ORDER BY total_spent DESC
LIMIT 20;
```

**Write operation (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):**
```sql
UPDATE users 
SET last_login = NOW() 
WHERE id = 123;
```

## Available Resources

### List All Tables

**Resource URI:** `postgres://tables`

Returns list of all tables in the database with their schemas.

**Response format:**
```typescript
{
  "tables": [
    { "schema": "public", "name": "users" },
    { "schema": "public", "name": "orders" },
    { "schema": "analytics", "name": "events" }
  ]
}
```

### Get Table Details

**Resource URI:** `postgres://table/{schema}/{table}`

Example: `postgres://table/public/users`

Returns table schema definition and sample data.

**Response format:**
```typescript
{
  "schema": {
    "columns": [
      { "name": "id", "type": "integer", "nullable": false },
      { "name": "email", "type": "varchar(255)", "nullable": false },
      { "name": "created_at", "type": "timestamp", "nullable": true }
    ],
    "constraints": [
      { "type": "PRIMARY KEY", "columns": ["id"] },
      { "type": "UNIQUE", "columns": ["email"] }
    ]
  },
  "sampleData": [
    { "id": 1, "email": "user@example.com", "created_at": "2024-01-15" }
  ]
}
```

## Common Usage Patterns

### Data Analysis

**Prompt:** "Show me the top 10 customers by revenue this month"

The AI will:
1. Use `postgres://tables` to discover available tables
2. Use `postgres://table/public/orders` and related tables to understand schema
3. Execute a query tool call with aggregated SQL

### Schema Exploration

**Prompt:** "What columns are in the products table?"

The AI will fetch `postgres://table/public/products` and describe the schema.

### Complex Joins

```typescript
// TypeScript example for building queries
const analysisQuery = `
  WITH monthly_sales AS (
    SELECT 
      DATE_TRUNC('month', o.created_at) as month,
      p.category,
      SUM(oi.quantity * oi.unit_price) as revenue
    FROM orders o
    JOIN order_items oi ON o.id = oi.order_id
    JOIN products p ON oi.product_id = p.id
    WHERE o.status = 'completed'
    GROUP BY 1, 2
  )
  SELECT 
    month,
    category,
    revenue,
    LAG(revenue) OVER (PARTITION BY category ORDER BY month) as prev_month_revenue,
    ROUND(
      100.0 * (revenue - LAG(revenue) OVER (PARTITION BY category ORDER BY month)) / 
      NULLIF(LAG(revenue) OVER (PARTITION BY category ORDER BY month), 0),
      2
    ) as growth_rate
  FROM monthly_sales
  ORDER BY month DESC, revenue DESC;
`;
```

### Data Quality Checks

```sql
-- Find duplicate emails
SELECT email, COUNT(*) as count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Check for orphaned records
SELECT o.id, o.user_id
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;

-- Find NULL values in required fields
SELECT 
  'users' as table_name,
  SUM(CASE WHEN email IS NULL THEN 1 ELSE 0 END) as null_emails,
  SUM(CASE WHEN name IS NULL THEN 1 ELSE 0 END) as null_names
FROM users;
```

## Development Workflow

### Local Setup with Docker

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

### Building and Testing

```bash
# Install dependencies
bun install

# Run in development (stdio)
bun run index.ts -- --transport=stdio

# Run with debug logging
DEBUG=true bun run index.ts -- --transport=stdio

# Run HTTP server
bun run index.ts -- --transport=http

# Run tests
bun test

# Build for production
bun run build:js
```

### Testing Queries

Use the MCP Inspector tool to test queries interactively:

```bash
bun run inspector
```

Then execute test queries:
```
> call query {"sql": "SELECT version();"}
> read postgres://tables
> read postgres://table/public/users
```

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or timeout errors

**Solution:**
- Verify `DATABASE_URL` is correct
- Check PostgreSQL is running: `pg_isready -h localhost -p 5432`
- Test connection: `psql $DATABASE_URL`
- For SSL issues, set `PG_SSL_ROOT_CERT` environment variable

### Permission Errors

**Problem:** "permission denied for table X"

**Solution:**
```sql
-- Grant read access
GRANT SELECT ON ALL TABLES IN SCHEMA public TO your_user;

-- Grant write access (if DANGEROUSLY_ALLOW_WRITE_OPS=true)
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO your_user;
```

### Write Operations Blocked

**Problem:** "Write operations are disabled"

**Solution:** Set `DANGEROUSLY_ALLOW_WRITE_OPS=true` in your MCP configuration. Use with caution in production.

### Query Timeout

**Problem:** Long-running queries hang

**Solution:**
```sql
-- Set statement timeout in query
SET statement_timeout = '30s';
SELECT * FROM large_table;

-- Or optimize with LIMIT
SELECT * FROM large_table LIMIT 1000;
```

### Schema Not Found

**Problem:** Cannot find tables in specific schema

**Solution:**
```sql
-- List all schemas
SELECT schema_name FROM information_schema.schemata;

-- Set search path
SET search_path TO myschema, public;

-- Or use fully qualified names
SELECT * FROM myschema.mytable;
```

## Best Practices

1. **Always use LIMIT** for exploratory queries on large tables
2. **Use transactions** for multiple write operations (when enabled)
3. **Index frequently queried columns** for better performance
4. **Use EXPLAIN ANALYZE** to optimize slow queries
5. **Keep write operations disabled** unless absolutely necessary
6. **Use prepared statement patterns** to prevent SQL injection in dynamic queries
7. **Monitor query performance** with `pg_stat_statements` extension

## License

MIT License - see project repository for details.
