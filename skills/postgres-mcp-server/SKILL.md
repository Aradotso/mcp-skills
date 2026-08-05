---
name: postgres-mcp-server
description: MCP server enabling LLMs to query and analyze PostgreSQL databases through a controlled interface with read/write capabilities.
triggers:
  - query the postgres database
  - analyze database tables
  - execute sql on postgres
  - show database schema
  - get table structure from postgres
  - run database queries through mcp
  - inspect postgres database
  - connect to postgresql via mcp
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol server that provides LLMs with controlled access to PostgreSQL databases for querying, analysis, and optional write operations.

## What It Does

- **Query Execution**: Run SQL queries against PostgreSQL databases
- **Schema Inspection**: List tables and view detailed table schemas
- **Sample Data**: Automatically fetch sample rows for exploration
- **Controlled Writes**: Optional write operations (disabled by default)
- **Transport Options**: Supports both stdio (default) and HTTP transports

## Installation

### Quick Install (npx)

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
        "DATABASE_URL": "postgresql://user:password@localhost:5432/mydb"
      }
    }
  }
}
```

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | Yes | - | PostgreSQL connection string |
| `DANGEROUSLY_ALLOW_WRITE_OPS` | No | `false` | Enable INSERT/UPDATE/DELETE operations |
| `DEBUG` | No | `false` | Enable debug logging |
| `PG_SSL_ROOT_CERT` | No | - | Path to TLS CA bundle (for cloud databases) |
| `PORT` | No | `3000` | HTTP server port (when using HTTP transport) |

### Connection String Format

```
postgresql://[user[:password]@][host][:port][/dbname][?param1=value1&...]
```

Example with SSL for AWS RDS:

```json
{
  "env": {
    "DATABASE_URL": "postgresql://myuser:mypass@mydb.us-east-1.rds.amazonaws.com:5432/production?sslmode=require",
    "PG_SSL_ROOT_CERT": "/path/to/global-bundle.pem"
  }
}
```

## Transport Modes

### Stdio (Default)

```bash
npx --yes pg-mcp-server --transport stdio
```

Best for local MCP clients like Claude Desktop and Cursor.

### HTTP

```bash
npx --yes pg-mcp-server --transport http
# Server runs at http://localhost:3000/mcp
```

For clients supporting MCP Streamable HTTP protocol.

## Available Tools

### `query` - Execute SQL Queries

Execute any SQL statement. Writes require `DANGEROUSLY_ALLOW_WRITE_OPS=true`.

**Parameters:**
- `sql` (string, required): SQL query to execute

**Example Usage:**

```typescript
// Read query
{
  "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
}

// Aggregate query
{
  "sql": "SELECT status, COUNT(*) as count FROM orders GROUP BY status"
}

// Join query
{
  "sql": "SELECT u.name, o.total FROM users u JOIN orders o ON u.id = o.user_id WHERE o.created_at > NOW() - INTERVAL '7 days'"
}
```

**Write Operations** (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):

```typescript
// Insert
{
  "sql": "INSERT INTO products (name, price) VALUES ('Widget', 29.99) RETURNING id"
}

// Update
{
  "sql": "UPDATE users SET last_login = NOW() WHERE id = 123"
}

// Delete
{
  "sql": "DELETE FROM temp_data WHERE created_at < NOW() - INTERVAL '30 days'"
}
```

## Available Resources

### `postgres://tables` - List All Tables

Returns a list of all tables in the database with basic metadata.

**Response includes:**
- Table name
- Schema name
- Row count estimate
- Table size

### `postgres://table/{schema}/{table}` - Table Details

Get detailed schema information and sample data for a specific table.

**Parameters:**
- `schema`: Schema name (usually `public`)
- `table`: Table name

**Response includes:**
- Column names and types
- Constraints (primary keys, foreign keys)
- Indexes
- Sample rows (default: 5 rows)

**Example:**
```
postgres://table/public/users
postgres://table/public/order_items
```

## Common Patterns

### Exploring a New Database

```typescript
// 1. List all tables
// Use resource: postgres://tables

// 2. Inspect specific table
// Use resource: postgres://table/public/users

// 3. Query for insights
{
  "sql": "SELECT column_name, data_type, is_nullable FROM information_schema.columns WHERE table_name = 'users'"
}
```

### Data Analysis Workflow

```typescript
// Count records
{
  "sql": "SELECT COUNT(*) FROM orders WHERE status = 'completed'"
}

// Distribution analysis
{
  "sql": "SELECT DATE_TRUNC('day', created_at) as day, COUNT(*) FROM orders GROUP BY day ORDER BY day DESC LIMIT 30"
}

// Top performers
{
  "sql": "SELECT product_id, SUM(quantity * price) as revenue FROM order_items GROUP BY product_id ORDER BY revenue DESC LIMIT 10"
}
```

### Complex Queries

```typescript
// Window functions
{
  "sql": "SELECT name, department, salary, AVG(salary) OVER (PARTITION BY department) as dept_avg FROM employees"
}

// CTEs (Common Table Expressions)
{
  "sql": `
    WITH monthly_revenue AS (
      SELECT DATE_TRUNC('month', created_at) as month, SUM(total) as revenue
      FROM orders
      GROUP BY month
    )
    SELECT month, revenue, LAG(revenue) OVER (ORDER BY month) as prev_month
    FROM monthly_revenue
    ORDER BY month DESC
  `
}

// Subqueries
{
  "sql": "SELECT * FROM products WHERE price > (SELECT AVG(price) FROM products)"
}
```

## Quick Start with Docker

The repo includes a Docker setup with sample data:

```bash
# Start PostgreSQL with sample data
bun run db:start

# Tables created: users, products, orders, order_items
# Connection: postgresql://postgres:postgres@localhost:5432/postgres

# Test with MCP Inspector
bun run inspector

# Stop when done
bun run db:stop
```

## Testing the Connection

### Simple Prompts to Test

After installing, try these prompts with your AI assistant:

```
"Show me the first 5 users from the database"
"What tables are in the database?"
"Describe the structure of the orders table"
"Count how many active users we have"
"Show me recent orders with their totals"
```

### Using MCP Inspector

```bash
# Install MCP Inspector globally
npm install -g @modelcontextprotocol/inspector

# Run with your database
DATABASE_URL="postgresql://user:pass@host:5432/db" npx @modelcontextprotocol/inspector npx --yes pg-mcp-server --transport stdio
```

## Troubleshooting

### Connection Errors

**Problem:** `connection refused` or timeout errors

**Solution:**
- Verify `DATABASE_URL` is correct
- Check if PostgreSQL is running: `pg_isready -h localhost -p 5432`
- Ensure firewall allows connection
- For cloud databases, verify SSL settings and use `PG_SSL_ROOT_CERT` if needed

### Write Operations Failing

**Problem:** `Write operations are disabled`

**Solution:**
Set environment variable:
```json
{
  "env": {
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### SSL/TLS Errors with Cloud Databases

**Problem:** `self signed certificate` or SSL errors

**Solution:**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:pass@host:5432/db?sslmode=require",
    "PG_SSL_ROOT_CERT": "/path/to/ca-bundle.pem"
  }
}
```

For AWS RDS, download the global bundle:
```bash
curl -o rds-ca-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
```

### Query Timeout

**Problem:** Slow queries timing out

**Solution:**
Add timeout to connection string:
```
postgresql://user:pass@host:5432/db?statement_timeout=30000
```

Or set in query:
```typescript
{
  "sql": "SET statement_timeout = 30000; SELECT * FROM large_table"
}
```

### Debugging

Enable debug logging:

```json
{
  "env": {
    "DEBUG": "true"
  }
}
```

Or run manually:
```bash
DEBUG=true DATABASE_URL="postgresql://..." npx --yes pg-mcp-server --transport stdio
```

## Security Best Practices

1. **Use read-only credentials** when possible
2. **Never enable writes** unless absolutely necessary
3. **Use environment variables** for credentials, never hardcode
4. **Limit permissions** at the database level (GRANT SELECT only)
5. **Use SSL/TLS** for production databases
6. **Rotate credentials** regularly
7. **Monitor query logs** for suspicious activity

## Example: Creating a Read-Only User

```sql
-- Create read-only user
CREATE USER mcp_readonly WITH PASSWORD 'secure_password';

-- Grant connect
GRANT CONNECT ON DATABASE mydb TO mcp_readonly;

-- Grant schema usage
GRANT USAGE ON SCHEMA public TO mcp_readonly;

-- Grant select on all tables
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;

-- Grant select on future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_readonly;
```

Then use this user in `DATABASE_URL`:
```
postgresql://mcp_readonly:secure_password@localhost:5432/mydb
```
