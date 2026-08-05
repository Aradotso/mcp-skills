---
name: postgres-mcp-server
description: MCP server that enables AI to query and analyze PostgreSQL databases through a controlled interface with tools, resources, and optional write operations.
triggers:
  - query my postgres database
  - show me tables in the database
  - get schema for this table
  - analyze data in postgres
  - execute sql query on database
  - list all database tables
  - get sample data from table
  - connect to postgresql database
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

This MCP server provides AI assistants with controlled access to PostgreSQL databases. It exposes tools for executing SQL queries, resources for browsing database schemas and tables, and configurable permissions for read-only or read-write operations.

## Installation

### Quick Install (Claude Desktop / Cursor)

Add to your MCP settings file:

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

### Configuration Options

Environment variables:

- **`DATABASE_URL`** (required): PostgreSQL connection string
  - Format: `postgresql://username:password@host:port/database`
  - Example: `postgresql://postgres:postgres@localhost:5432/myapp`
- **`DANGEROUSLY_ALLOW_WRITE_OPS`**: Enable INSERT/UPDATE/DELETE operations (default: `false`)
- **`DEBUG`**: Enable debug logging (default: `false`)
- **`PG_SSL_ROOT_CERT`**: Path to TLS CA bundle for SSL connections (e.g., AWS RDS)

### Transport Modes

**stdio** (default):
```json
{
  "command": "npx",
  "args": ["--yes", "pg-mcp-server", "--transport", "stdio"]
}
```

**HTTP** (Streamable HTTP endpoint at `/mcp`):
```json
{
  "command": "npx",
  "args": ["--yes", "pg-mcp-server", "--transport", "http"],
  "env": {
    "PORT": "3000",
    "DATABASE_URL": "postgresql://postgres:postgres@localhost:5432/postgres"
  }
}
```

## Available Tools

### `query`

Execute SQL queries against the database.

**Parameters:**
```typescript
{
  sql: string  // SQL query to execute
}
```

**Example usage:**
```typescript
// Tool call from AI
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, email, created_at FROM users WHERE active = true LIMIT 10"
  }
}
```

**Read-only mode (default):**
- Only SELECT statements allowed
- INSERT, UPDATE, DELETE, DROP will be rejected

**Write mode (DANGEROUSLY_ALLOW_WRITE_OPS=true):**
```typescript
// Insert example
{
  "name": "query",
  "arguments": {
    "sql": "INSERT INTO users (email, name) VALUES ('user@example.com', 'John Doe') RETURNING id"
  }
}

// Update example
{
  "name": "query",
  "arguments": {
    "sql": "UPDATE products SET price = 19.99 WHERE id = 42"
  }
}
```

## Available Resources

### `postgres://tables`

Lists all tables in the database with their schemas.

**Returns:**
- Array of table names with schema information
- Useful for discovering what data is available

### `postgres://table/{schema}/{table}`

Get detailed information about a specific table.

**URI format:**
```
postgres://table/public/users
postgres://table/myschema/products
```

**Returns:**
- Column definitions (name, type, nullable, default)
- Primary keys and indexes
- Foreign key relationships
- Sample data (up to 5 rows)

**Example usage:**
```typescript
// Resource URI
"postgres://table/public/orders"

// Returns schema + sample data
{
  "columns": [
    { "name": "id", "type": "integer", "nullable": false, "primary_key": true },
    { "name": "user_id", "type": "integer", "nullable": false },
    { "name": "total", "type": "numeric", "nullable": false },
    { "name": "created_at", "type": "timestamp", "nullable": false }
  ],
  "sample_data": [ /* up to 5 rows */ ]
}
```

## Common Patterns

### Data Exploration

**List all tables:**
```typescript
// Use resource: postgres://tables
// AI will see all available tables and schemas
```

**Inspect table structure:**
```typescript
// Use resource: postgres://table/public/users
// Review columns, types, constraints, and sample data before querying
```

**Count records:**
```typescript
{
  "name": "query",
  "arguments": {
    "sql": "SELECT COUNT(*) as total FROM users"
  }
}
```

### Data Analysis

**Find active users created in last 30 days:**
```typescript
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, email, created_at FROM users WHERE active = true AND created_at > NOW() - INTERVAL '30 days' ORDER BY created_at DESC"
  }
}
```

**Aggregate sales by month:**
```typescript
{
  "name": "query",
  "arguments": {
    "sql": "SELECT DATE_TRUNC('month', created_at) as month, SUM(total) as revenue FROM orders WHERE created_at > '2024-01-01' GROUP BY month ORDER BY month"
  }
}
```

**Join tables for detailed view:**
```typescript
{
  "name": "query",
  "arguments": {
    "sql": "SELECT o.id, o.total, u.email, u.name FROM orders o JOIN users u ON o.user_id = u.id WHERE o.created_at > '2024-01-01' LIMIT 100"
  }
}
```

### Safe Data Modification (Write Mode)

**Enable write operations:**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:password@host:5432/db",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

**Insert new record:**
```typescript
{
  "name": "query",
  "arguments": {
    "sql": "INSERT INTO products (name, price, category) VALUES ('Widget', 29.99, 'hardware') RETURNING id, name, price"
  }
}
```

**Update existing records:**
```typescript
{
  "name": "query",
  "arguments": {
    "sql": "UPDATE users SET last_login = NOW() WHERE email = 'user@example.com'"
  }
}
```

**Transactional operations:**
```typescript
// Use BEGIN/COMMIT for multi-statement transactions
{
  "name": "query",
  "arguments": {
    "sql": "BEGIN; UPDATE accounts SET balance = balance - 100 WHERE id = 1; UPDATE accounts SET balance = balance + 100 WHERE id = 2; COMMIT;"
  }
}
```

## Development & Local Testing

### Docker Quick Start

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

### Build from Source

```bash
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install

# Run with stdio transport
bun run index.ts -- --transport=stdio

# Run with HTTP transport
bun run index.ts -- --transport=http

# Build for distribution
bun run build:js
```

### Use Local Build in MCP Client

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

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to database

**Solutions:**
- Verify `DATABASE_URL` format: `postgresql://user:pass@host:port/db`
- Check PostgreSQL is running and accepting connections
- Verify network access (firewall, security groups)
- For SSL connections, set `PG_SSL_ROOT_CERT` to CA bundle path

### Permission Errors

**Problem:** "Write operations not allowed"

**Solution:**
- Set `DANGEROUSLY_ALLOW_WRITE_OPS=true` in environment
- Note: Only enable for trusted environments

### Query Failures

**Problem:** SQL syntax errors

**Solutions:**
- Check PostgreSQL version compatibility
- Verify table/column names exist (use `postgres://tables` resource)
- Use `postgres://table/{schema}/{table}` to inspect schema first

### Debug Mode

Enable detailed logging:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:pass@host:port/db",
    "DEBUG": "true"
  }
}
```

### SSL/TLS Issues

For AWS RDS or other SSL-required databases:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:pass@rds-host:5432/db?ssl=true",
    "PG_SSL_ROOT_CERT": "/path/to/rds-ca-bundle.pem"
  }
}
```

## Security Best Practices

1. **Read-only by default:** Never enable write operations in production without careful consideration
2. **Least privilege:** Use database users with minimal required permissions
3. **Connection strings:** Store `DATABASE_URL` in secure environment variables, never in code
4. **Query review:** Always review AI-generated queries before execution in production
5. **Network isolation:** Restrict database access to necessary networks only
