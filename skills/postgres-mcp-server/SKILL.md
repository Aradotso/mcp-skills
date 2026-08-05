---
name: postgres-mcp-server
description: MCP server that enables LLMs to query and analyze PostgreSQL databases through a controlled interface with read and optional write operations.
triggers:
  - query my postgres database
  - analyze database tables
  - run sql query through mcp
  - connect to postgresql database
  - explore database schema
  - get table structure from postgres
  - execute database queries safely
  - inspect postgres database
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server for PostgreSQL databases that enables LLMs to query and analyze PostgreSQL databases through a controlled, safe interface. Supports both stdio and HTTP transports, with configurable read-only or read-write modes.

## Installation

### Quick Install (Cursor/Claude Desktop)

Add to your MCP client settings (e.g., `~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://username:password@localhost:5432/database"
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

# Build for production
bun run build:js
```

Use local build in MCP settings:

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

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `DATABASE_URL` | PostgreSQL connection string | Yes | - |
| `DANGEROUSLY_ALLOW_WRITE_OPS` | Enable INSERT/UPDATE/DELETE operations | No | `false` |
| `DEBUG` | Enable debug logging | No | `false` |
| `PG_SSL_ROOT_CERT` | Path to TLS CA bundle for SSL connections | No | - |
| `PORT` | HTTP server port (HTTP transport only) | No | `3000` |

### Connection String Format

```
postgresql://[username]:[password]@[host]:[port]/[database]?[options]
```

Example with SSL:
```
postgresql://user:pass@mydb.region.rds.amazonaws.com:5432/production?sslmode=require
```

## Transport Modes

### stdio (Default)

Used with Claude Desktop, Cursor, and similar MCP clients:

```bash
pg-mcp-server --transport=stdio
```

### HTTP

Serves MCP over HTTP at `/mcp` endpoint:

```bash
pg-mcp-server --transport=http
# Server runs on http://localhost:3000/mcp
```

With custom port:

```bash
PORT=8080 pg-mcp-server --transport=http
```

## Available Tools

### `query` Tool

Execute SQL queries against the database.

**Parameters:**
- `sql` (string, required): The SQL query to execute

**Example Usage:**

```typescript
// In read-only mode (default)
{
  "sql": "SELECT id, name, email FROM users WHERE active = true LIMIT 10"
}

// With DANGEROUSLY_ALLOW_WRITE_OPS=true
{
  "sql": "UPDATE users SET last_login = NOW() WHERE id = 123"
}
```

**Response Format:**

```json
{
  "rows": [
    {"id": 1, "name": "Alice", "email": "alice@example.com"},
    {"id": 2, "name": "Bob", "email": "bob@example.com"}
  ],
  "rowCount": 2,
  "fields": [
    {"name": "id", "dataTypeID": 23},
    {"name": "name", "dataTypeID": 1043},
    {"name": "email", "dataTypeID": 1043}
  ]
}
```

## Available Resources

### List All Tables

**Resource URI:** `postgres://tables`

Returns a list of all tables in the database with their schemas.

**Response Example:**

```json
{
  "tables": [
    {"schema": "public", "table": "users"},
    {"schema": "public", "table": "orders"},
    {"schema": "analytics", "table": "events"}
  ]
}
```

### Get Table Details

**Resource URI:** `postgres://table/{schema}/{table}`

Returns detailed schema information and sample data for a specific table.

**Example:** `postgres://table/public/users`

**Response Example:**

```json
{
  "schema": "public",
  "table": "users",
  "columns": [
    {"name": "id", "type": "integer", "nullable": false},
    {"name": "email", "type": "character varying", "nullable": false},
    {"name": "created_at", "type": "timestamp with time zone", "nullable": true}
  ],
  "sampleData": [
    {"id": 1, "email": "user@example.com", "created_at": "2024-01-15T10:30:00Z"}
  ]
}
```

## Common Usage Patterns

### Read-Only Data Analysis

Default configuration is safe for production databases:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://readonly_user:password@prod-db:5432/app"
      }
    }
  }
}
```

**Example prompts:**
- "Show me the top 10 customers by order volume"
- "What's the average order value in the last 30 days?"
- "List all tables in the database"
- "Show me the schema of the users table"

### Development with Write Access

Enable writes for development/testing:

```json
{
  "mcpServers": {
    "postgres-dev": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://dev:dev@localhost:5432/dev_db",
        "DANGEROUSLY_ALLOW_WRITE_OPS": "true",
        "DEBUG": "true"
      }
    }
  }
}
```

**Example prompts:**
- "Create a test user with email test@example.com"
- "Update the status of order #123 to 'shipped'"
- "Delete all test data from the sandbox table"

### Multi-Database Setup

Connect to multiple databases simultaneously:

```json
{
  "mcpServers": {
    "postgres-prod": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://readonly:pass@prod:5432/app"
      }
    },
    "postgres-analytics": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://analyst:pass@warehouse:5432/analytics"
      }
    }
  }
}
```

### Secure Connections (AWS RDS, etc.)

For databases requiring SSL/TLS:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@db.region.rds.amazonaws.com:5432/prod?sslmode=require",
        "PG_SSL_ROOT_CERT": "/path/to/rds-combined-ca-bundle.pem"
      }
    }
  }
}
```

## Development & Testing

### Quick Start with Docker

The repository includes a Docker setup with sample data:

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample tables: `users`, `products`, `orders`, `order_items`

### Running Locally

```bash
# Clone repository
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install

# Run with stdio transport
bun run index.ts -- --transport=stdio

# Run with HTTP transport
bun run index.ts -- --transport=http

# Run with debug logging
DEBUG=true bun run index.ts -- --transport=stdio

# Run tests
bun test
```

### Building for Distribution

```bash
# Build JavaScript output
bun run build:js

# Test built version
node lib/index.js --transport=stdio
```

## Troubleshooting

### Connection Refused

**Problem:** Cannot connect to PostgreSQL database

**Solutions:**
- Verify `DATABASE_URL` is correct
- Check PostgreSQL is running: `pg_isready -h localhost -p 5432`
- Ensure network access (firewall, security groups)
- For Docker: use `host.docker.internal` instead of `localhost`

### SSL/TLS Errors

**Problem:** SSL certificate verification failed

**Solutions:**
- Add `?sslmode=require` to connection string
- Set `PG_SSL_ROOT_CERT` to CA bundle path
- For development only: `?sslmode=no-verify` (not recommended)

### Permission Denied on Queries

**Problem:** User lacks permissions to execute queries

**Solutions:**
- Grant SELECT permissions: `GRANT SELECT ON ALL TABLES IN SCHEMA public TO username;`
- For writes: `GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO username;`
- Check role memberships: `\du` in psql

### Write Operations Blocked

**Problem:** INSERT/UPDATE/DELETE queries fail

**Solution:**
Set `DANGEROUSLY_ALLOW_WRITE_OPS=true` in environment variables. Use only in development environments.

### MCP Client Not Detecting Server

**Problem:** Server not appearing in MCP client

**Solutions:**
- Restart MCP client after configuration changes
- Check JSON syntax in config file
- Verify `command` path is correct
- Check client logs for errors
- For Cursor: `Cmd+Shift+P` → "Reload Window"

### High Memory Usage

**Problem:** Server consuming excessive memory

**Solutions:**
- Add `LIMIT` clauses to queries
- Avoid `SELECT *` on large tables
- Use pagination for large result sets
- Monitor query execution time

## Example Queries

### Data Analysis

```sql
-- Top customers by revenue
SELECT 
  u.name,
  COUNT(o.id) as order_count,
  SUM(o.total) as total_revenue
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.created_at > NOW() - INTERVAL '30 days'
GROUP BY u.id, u.name
ORDER BY total_revenue DESC
LIMIT 10;
```

### Schema Exploration

```sql
-- List all indexes on a table
SELECT
  indexname,
  indexdef
FROM pg_indexes
WHERE tablename = 'orders'
  AND schemaname = 'public';
```

### Performance Monitoring

```sql
-- Find slow queries
SELECT
  query,
  calls,
  total_time,
  mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

## Best Practices

1. **Use read-only users for production** - Create dedicated users with minimal permissions
2. **Enable SSL for remote connections** - Always use encrypted connections
3. **Add LIMIT clauses** - Prevent accidentally returning millions of rows
4. **Use specific column names** - Avoid `SELECT *` for better performance
5. **Test queries in development first** - Validate before running on production
6. **Monitor query performance** - Watch for slow or expensive queries
7. **Keep DATABASE_URL secure** - Never commit credentials to version control

## License

MIT - See LICENSE file in repository
