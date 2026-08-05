---
name: postgres-mcp-server
description: MCP server that enables LLMs to query and analyze PostgreSQL databases through a controlled interface
triggers:
  - query my postgres database
  - analyze database tables and schemas
  - run SQL queries through MCP
  - connect to postgresql with MCP
  - inspect database structure
  - execute database queries safely
  - use postgres MCP tools
  - get table schema and data
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This skill enables AI agents to interact with PostgreSQL databases through the Model Context Protocol (MCP). The server provides tools for executing SQL queries and resources for exploring database schemas and tables.

## What It Does

- **Execute SQL queries** through MCP tools with optional write protection
- **Explore database schemas** via resources listing tables and their structures
- **Sample table data** to understand content without writing custom queries
- **Control write operations** with environment-based safety flags
- **Support both stdio and HTTP transports** for different MCP client architectures

## Installation

### Basic Setup (stdio transport)

Add to your MCP client configuration (e.g., Claude Desktop, Cursor):

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://user:password@localhost:5432/mydb"
      }
    }
  }
}
```

### HTTP Transport Setup

For clients supporting Streamable HTTP:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "http"],
      "env": {
        "DATABASE_URL": "postgresql://user:password@localhost:5432/mydb",
        "PORT": "3000"
      }
    }
  }
}
```

Connect to `http://localhost:3000/mcp` from your MCP client.

### Local Development Setup

```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": ["/absolute/path/to/pg-mcp-server/lib/index.js", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://postgres:postgres@localhost:5432/postgres",
        "DEBUG": "true"
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

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (optional, default: `false`): Enable INSERT/UPDATE/DELETE operations
  - Set to `"true"` to allow write operations
  - **Warning**: Only enable for trusted environments

- **`DEBUG`** (optional, default: `false`): Enable debug logging
  - Set to `"true"` for verbose output

- **`PG_SSL_ROOT_CERT`** (optional): Path to TLS CA bundle for SSL connections
  - Useful for AWS RDS or other managed databases requiring SSL
  - Example: `/path/to/rds-combined-ca-bundle.pem`

- **`PORT`** (optional, default: `3000`): HTTP server port (when using HTTP transport)

### Transport Modes

```bash
# stdio transport (default) - for most MCP clients
pg-mcp-server --transport=stdio

# HTTP transport - for clients supporting Streamable HTTP
pg-mcp-server --transport=http
```

## Available Tools

### `query` Tool

Execute SQL queries against the connected database.

**Parameters:**
```typescript
{
  sql: string  // SQL query to execute
}
```

**Example Usage (Read-only):**

```typescript
// Tool call
{
  "name": "query",
  "arguments": {
    "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
  }
}
```

**Example Usage (With Writes Enabled):**

```typescript
// Requires DANGEROUSLY_ALLOW_WRITE_OPS=true
{
  "name": "query",
  "arguments": {
    "sql": "INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com')"
  }
}
```

**Common Query Patterns:**

```sql
-- Get table row counts
SELECT 
  schemaname,
  tablename,
  n_live_tup as row_count
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;

-- Find tables with specific columns
SELECT 
  table_name,
  column_name,
  data_type
FROM information_schema.columns
WHERE column_name ILIKE '%email%';

-- Analyze table relationships
SELECT
  tc.table_name,
  kcu.column_name,
  ccu.table_name AS foreign_table_name,
  ccu.column_name AS foreign_column_name
FROM information_schema.table_constraints AS tc
JOIN information_schema.key_column_usage AS kcu
  ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage AS ccu
  ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY';
```

## Available Resources

### `postgres://tables` Resource

Lists all tables in the database with their schemas.

**Returns:**
- Table names
- Schema names
- Column definitions with types

**Usage Pattern:**
```typescript
// Resource URI: postgres://tables
// Returns JSON with all tables and their schemas
```

### `postgres://table/{schema}/{table}` Resource

Get detailed information about a specific table including schema and sample data.

**Parameters:**
- `{schema}`: Schema name (e.g., `public`)
- `{table}`: Table name

**Returns:**
- Complete column definitions
- Data types and constraints
- Sample rows from the table

**Usage Pattern:**
```typescript
// Resource URI: postgres://table/public/users
// Returns schema + sample data for the users table
```

## Real-World Examples

### Example 1: Exploring a New Database

```typescript
// Step 1: List all tables
// Use resource: postgres://tables

// Step 2: Examine a specific table
// Use resource: postgres://table/public/orders

// Step 3: Query aggregated data
{
  "name": "query",
  "arguments": {
    "sql": `
      SELECT 
        DATE_TRUNC('month', created_at) as month,
        COUNT(*) as order_count,
        SUM(total_amount) as revenue
      FROM orders
      WHERE created_at > NOW() - INTERVAL '6 months'
      GROUP BY DATE_TRUNC('month', created_at)
      ORDER BY month DESC
    `
  }
}
```

### Example 2: Data Analysis Workflow

```typescript
// Find top customers by revenue
{
  "name": "query",
  "arguments": {
    "sql": `
      SELECT 
        u.id,
        u.name,
        u.email,
        COUNT(DISTINCT o.id) as order_count,
        SUM(o.total_amount) as total_revenue
      FROM users u
      JOIN orders o ON u.id = o.user_id
      WHERE o.created_at > NOW() - INTERVAL '1 year'
      GROUP BY u.id, u.name, u.email
      ORDER BY total_revenue DESC
      LIMIT 20
    `
  }
}
```

### Example 3: Schema Inspection

```typescript
// Find all indexes on a table
{
  "name": "query",
  "arguments": {
    "sql": `
      SELECT
        indexname,
        indexdef
      FROM pg_indexes
      WHERE tablename = 'users'
      AND schemaname = 'public'
    `
  }
}

// Check table size
{
  "name": "query",
  "arguments": {
    "sql": `
      SELECT
        pg_size_pretty(pg_total_relation_size('public.orders')) as total_size,
        pg_size_pretty(pg_relation_size('public.orders')) as table_size,
        pg_size_pretty(pg_total_relation_size('public.orders') - pg_relation_size('public.orders')) as indexes_size
    `
  }
}
```

### Example 4: Safe Write Operations

```typescript
// Enable writes in configuration first:
// "DANGEROUSLY_ALLOW_WRITE_OPS": "true"

// Update user status
{
  "name": "query",
  "arguments": {
    "sql": `
      UPDATE users 
      SET active = false, updated_at = NOW()
      WHERE last_login_at < NOW() - INTERVAL '2 years'
      RETURNING id, email
    `
  }
}

// Insert with conflict handling
{
  "name": "query",
  "arguments": {
    "sql": `
      INSERT INTO products (sku, name, price)
      VALUES ('SKU-001', 'Widget', 29.99)
      ON CONFLICT (sku) 
      DO UPDATE SET price = EXCLUDED.price
      RETURNING id
    `
  }
}
```

## Common Patterns

### Pattern 1: Database Discovery

```typescript
// 1. List all tables
// Resource: postgres://tables

// 2. For each interesting table, get schema + samples
// Resource: postgres://table/public/{table_name}

// 3. Query specific insights
// Tool: query with targeted SQL
```

### Pattern 2: Analytical Queries

```typescript
// Use CTEs for complex analysis
{
  "name": "query",
  "arguments": {
    "sql": `
      WITH monthly_stats AS (
        SELECT 
          DATE_TRUNC('month', created_at) as month,
          COUNT(*) as count,
          AVG(total_amount) as avg_amount
        FROM orders
        GROUP BY DATE_TRUNC('month', created_at)
      )
      SELECT 
        month,
        count,
        avg_amount,
        LAG(count) OVER (ORDER BY month) as prev_month_count,
        (count - LAG(count) OVER (ORDER BY month))::float / LAG(count) OVER (ORDER BY month) * 100 as growth_pct
      FROM monthly_stats
      ORDER BY month DESC
      LIMIT 12
    `
  }
}
```

### Pattern 3: Safe Data Exploration

```typescript
// Always use LIMIT for initial exploration
{
  "name": "query",
  "arguments": {
    "sql": "SELECT * FROM large_table LIMIT 100"
  }
}

// Use EXPLAIN for query planning
{
  "name": "query",
  "arguments": {
    "sql": "EXPLAIN ANALYZE SELECT * FROM users WHERE email LIKE '%@example.com%'"
  }
}
```

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or timeout errors

**Solutions:**
- Verify `DATABASE_URL` is correct
- Check PostgreSQL is running: `pg_isready -h localhost -p 5432`
- Verify network access and firewall rules
- For cloud databases, ensure IP allowlist includes your location

### SSL/TLS Errors

**Problem:** "SSL connection required" or certificate errors

**Solutions:**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:pass@host:5432/db?sslmode=require",
    "PG_SSL_ROOT_CERT": "/path/to/ca-bundle.pem"
  }
}
```

### Write Operations Blocked

**Problem:** INSERT/UPDATE/DELETE queries fail

**Solution:** Enable write operations explicitly:
```json
{
  "env": {
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### Permission Errors

**Problem:** "permission denied for table" errors

**Solutions:**
- Verify database user has required grants:
  ```sql
  GRANT SELECT ON ALL TABLES IN SCHEMA public TO myuser;
  -- Or for writes:
  GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO myuser;
  ```
- Check schema-level permissions
- Ensure role membership is correct

### Resource Not Found

**Problem:** Table resources return errors

**Solutions:**
- Verify schema name (usually `public`)
- Check table name spelling (PostgreSQL is case-sensitive in quotes)
- Use resource `postgres://tables` to list available tables

### Debugging

Enable debug mode for verbose logging:
```json
{
  "env": {
    "DEBUG": "true"
  }
}
```

Check logs for:
- SQL query execution
- Connection pool status
- Error stack traces

## Quick Start with Docker

For testing and development:

```bash
# Clone repository
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server

# Install dependencies
bun install

# Start PostgreSQL with sample data
bun run db:start

# Run MCP Inspector to test
bun run inspector

# Stop database
bun run db:stop
```

Sample data includes: `users`, `products`, `orders`, `order_items` tables.

## Development Commands

```bash
# Run with stdio transport
bun run index.ts -- --transport=stdio

# Run with HTTP transport
bun run index.ts -- --transport=http

# Enable debug logging
DEBUG=true bun run index.ts -- --transport=stdio

# Run tests
bun test

# Build for distribution
bun run build:js
```

## Best Practices

1. **Always use LIMIT** for exploratory queries on large tables
2. **Enable writes carefully** - only in trusted, controlled environments
3. **Use transactions** when making multiple related changes
4. **Leverage resources** (`postgres://tables`, `postgres://table/{schema}/{table}`) before writing custom queries
5. **Test queries** with EXPLAIN before running on production data
6. **Use parameterized queries** when possible (though MCP tool interface handles this)
7. **Monitor query performance** with `pg_stat_statements` extension

## Security Notes

- Read-only by default - writes require explicit opt-in
- Always use environment variables for credentials, never hardcode
- Consider read-only database users for production usage
- Use SSL/TLS for remote connections
- Audit `DANGEROUSLY_ALLOW_WRITE_OPS` usage carefully
