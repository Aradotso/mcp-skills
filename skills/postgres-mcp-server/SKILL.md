---
name: postgres-mcp-server
description: Query and analyze PostgreSQL databases through MCP with LLMs using controlled SQL execution and schema inspection
triggers:
  - query my postgres database
  - analyze postgresql data
  - execute sql through mcp
  - inspect database schema
  - run database queries with llm
  - connect to postgres via mcp
  - get postgresql table structure
  - explore database with ai
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Postgres MCP Server is a Model Context Protocol (MCP) server that provides LLMs with controlled access to PostgreSQL databases. It enables querying, schema inspection, and data analysis through a safe, structured interface with optional write protection.

## Installation

### Quick Install (Cursor/Claude Desktop)

Add to your MCP client configuration file:

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

### Configuration Files Locations

- **Cursor**: `~/.cursor/mcp.json` (macOS/Linux) or `%APPDATA%\Cursor\mcp.json` (Windows)
- **Claude Desktop**: `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)

## Configuration Options

### Environment Variables

- `DATABASE_URL` **(required)**: PostgreSQL connection string
  - Format: `postgresql://username:password@host:port/database`
  - Example: `postgresql://postgres:postgres@localhost:5432/myapp`
- `DANGEROUSLY_ALLOW_WRITE_OPS` (optional): Enable INSERT/UPDATE/DELETE operations
  - Default: `false` (read-only mode)
  - Set to `"true"` to enable writes
- `DEBUG` (optional): Enable debug logging
  - Default: `false`
  - Set to `"true"` for verbose output
- `PG_SSL_ROOT_CERT` (optional): Path to SSL/TLS CA certificate bundle
  - Useful for cloud providers like AWS RDS
  - Example: `/path/to/rds-ca-bundle.pem`

### Enable Write Operations

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://postgres:postgres@localhost:5432/myapp",
        "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
      }
    }
  }
}
```

### SSL/TLS Configuration

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@rds.amazonaws.com:5432/prod",
        "PG_SSL_ROOT_CERT": "/path/to/rds-global-bundle.pem"
      }
    }
  }
}
```

## Available Tools

### `query` - Execute SQL Queries

Execute SQL queries against the database. Read-only by default unless `DANGEROUSLY_ALLOW_WRITE_OPS` is enabled.

**Parameters:**
- `sql` (string, required): The SQL query to execute

**Example Usage:**

```typescript
// Through MCP tool call
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, email, created_at FROM users WHERE active = true LIMIT 10"
  }
}
```

**Common Query Patterns:**

```sql
-- List all tables
SELECT table_name 
FROM information_schema.tables 
WHERE table_schema = 'public';

-- Get row counts
SELECT COUNT(*) FROM users;

-- Aggregate analysis
SELECT 
  status, 
  COUNT(*) as count,
  AVG(total) as avg_total
FROM orders
GROUP BY status;

-- Join multiple tables
SELECT 
  u.email,
  COUNT(o.id) as order_count,
  SUM(o.total) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.email
ORDER BY total_spent DESC
LIMIT 20;
```

## Available Resources

### `postgres://tables` - List All Tables

Returns a list of all tables in the database with their schemas.

**Example Response:**
```
public.users
public.products
public.orders
public.order_items
```

### `postgres://table/{schema}/{table}` - Table Schema and Sample Data

Get detailed information about a specific table including columns, types, constraints, and sample rows.

**Example:**
```
postgres://table/public/users
```

**Returns:**
- Column names and data types
- Primary keys and constraints
- Indexes
- Sample data (first few rows)

## Natural Language Prompts

Once configured, you can use natural language to interact with your database:

### Query Examples

```
Show me the first 10 active users
```

```
What's the total revenue by product category this month?
```

```
Find all orders that haven't been fulfilled yet
```

```
Show me the database schema for the products table
```

### Analysis Examples

```
Analyze user signup trends over the last 6 months
```

```
Which products have the highest return rate?
```

```
Show me customers who haven't ordered in 90 days
```

## Transport Modes

### STDIO (Default)

Standard input/output mode for MCP clients like Cursor and Claude Desktop:

```bash
npx pg-mcp-server --transport stdio
```

### HTTP (Streamable)

For clients supporting Streamable HTTP:

```bash
npx pg-mcp-server --transport http
# Server runs on http://localhost:3000/mcp
```

Custom port:
```bash
PORT=8080 npx pg-mcp-server --transport http
```

## Development and Testing

### Local Development Setup

```bash
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install

# Run with stdio
bun run index.ts -- --transport=stdio

# Run with HTTP
bun run index.ts -- --transport=http

# Enable debug mode
DEBUG=true bun run index.ts -- --transport=stdio
```

### Using Local Build in MCP Client

```bash
# Build JavaScript distribution
bun run build:js
```

Update MCP configuration:
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

### Docker Testing Environment

Quick start with sample database:

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample tables created:
- `users` - User accounts
- `products` - Product catalog
- `orders` - Customer orders
- `order_items` - Order line items

## Common Patterns

### Safe Data Exploration

Start with schema inspection before querying:

1. **List all tables:**
   ```
   Show me all tables in the database
   ```

2. **Inspect specific table:**
   ```
   Show me the schema for the orders table
   ```

3. **Query with limits:**
   ```
   Get 10 sample rows from orders
   ```

### Analytical Queries

```sql
-- Time-series analysis
SELECT 
  DATE_TRUNC('day', created_at) as date,
  COUNT(*) as signups
FROM users
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY date
ORDER BY date;

-- Cohort analysis
SELECT 
  DATE_TRUNC('month', u.created_at) as cohort,
  COUNT(DISTINCT o.user_id) as active_users
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY cohort
ORDER BY cohort;
```

### Data Quality Checks

```sql
-- Find null values
SELECT 
  COUNT(*) FILTER (WHERE email IS NULL) as missing_email,
  COUNT(*) FILTER (WHERE phone IS NULL) as missing_phone
FROM users;

-- Duplicate detection
SELECT email, COUNT(*) 
FROM users 
GROUP BY email 
HAVING COUNT(*) > 1;
```

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to database

**Solutions:**
- Verify `DATABASE_URL` format: `postgresql://user:pass@host:port/db`
- Check database is running: `psql $DATABASE_URL -c "SELECT 1"`
- Ensure network connectivity and firewall rules
- For SSL/TLS errors, set `PG_SSL_ROOT_CERT` environment variable

### Write Operations Blocked

**Problem:** INSERT/UPDATE/DELETE queries fail

**Solution:** Enable write mode (use with caution):
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### Permission Errors

**Problem:** "permission denied for table X"

**Solutions:**
- Grant necessary privileges: `GRANT SELECT ON ALL TABLES IN SCHEMA public TO myuser;`
- Use a database user with appropriate permissions
- For read-only operations, create a restricted user:
  ```sql
  CREATE USER readonly WITH PASSWORD 'password';
  GRANT CONNECT ON DATABASE mydb TO readonly;
  GRANT USAGE ON SCHEMA public TO readonly;
  GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
  ```

### MCP Server Not Detected

**Problem:** Client doesn't recognize the server

**Solutions:**
- Restart your MCP client (Cursor/Claude Desktop)
- Verify JSON configuration syntax (no trailing commas)
- Check absolute paths in configuration
- Review client logs for error messages

### Query Timeout

**Problem:** Long-running queries timeout

**Solutions:**
- Add `LIMIT` clauses to queries
- Create indexes on frequently queried columns
- Optimize query with `EXPLAIN ANALYZE`
- Increase client timeout if supported

## Security Best Practices

1. **Use Read-Only Users:** Create dedicated database users with SELECT-only permissions
2. **Keep Writes Disabled:** Only enable `DANGEROUSLY_ALLOW_WRITE_OPS` when absolutely necessary
3. **Use SSL/TLS:** Always connect to production databases with SSL enabled
4. **Limit Network Access:** Use firewalls and VPNs to restrict database access
5. **Avoid Root Users:** Never use superuser accounts for application access
6. **Store Credentials Securely:** Use environment variables, never hardcode passwords
