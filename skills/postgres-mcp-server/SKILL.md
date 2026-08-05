---
name: postgres-mcp-server
description: MCP server that enables LLMs to query and analyze PostgreSQL databases through a controlled interface with read/write capabilities.
triggers:
  - query my postgres database
  - analyze database tables and schema
  - execute sql queries through mcp
  - connect to postgresql database
  - explore database structure
  - run database queries with llm
  - set up postgres mcp server
  - interact with postgres via mcp
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol server that provides LLMs controlled access to PostgreSQL databases for querying, schema inspection, and data analysis. Supports both stdio and HTTP transports.

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

Then reference the local build:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": ["/absolute/path/to/pg-mcp-server/lib/index.js", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://user:password@localhost:5432/dbname"
      }
    }
  }
}
```

## Configuration

### Environment Variables

- **`DATABASE_URL`** (required): PostgreSQL connection string
  - Format: `postgresql://username:password@host:port/database`
  - Example: `postgresql://postgres:postgres@localhost:5432/mydb`

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (optional): Enable INSERT/UPDATE/DELETE operations
  - Default: `false`
  - Set to `true` to allow write operations (use with caution)

- **`DEBUG`** (optional): Enable debug logging
  - Default: `false`
  - Set to `true` for verbose output

- **`PG_SSL_ROOT_CERT`** (optional): Path to TLS CA bundle
  - Use for SSL connections (e.g., AWS RDS)
  - Example: `/path/to/rds-ca-bundle.pem`

### Transport Modes

**Stdio (Default)**: For local MCP clients

```bash
pg-mcp-server --transport=stdio
```

**HTTP**: For remote or web-based clients

```bash
pg-mcp-server --transport=http
# Serves at http://localhost:3000/mcp
```

Set port with `PORT` environment variable:

```bash
PORT=8080 pg-mcp-server --transport=http
```

## Available Tools

### `query` - Execute SQL Queries

Execute SELECT queries (or INSERT/UPDATE/DELETE if write ops enabled).

**Parameters:**
- `sql` (string, required): SQL query to execute

**Example:**

```typescript
// Tool call from LLM
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, email, created_at FROM users WHERE active = true LIMIT 10"
  }
}
```

**Response:**

```json
{
  "rows": [
    {"id": 1, "email": "user@example.com", "created_at": "2024-01-15T10:30:00Z"},
    {"id": 2, "email": "another@example.com", "created_at": "2024-01-16T14:22:00Z"}
  ],
  "rowCount": 2
}
```

## Available Resources

### `postgres://tables` - List All Tables

Returns all tables in the database with their schemas.

**URI:** `postgres://tables`

**Example Response:**

```json
[
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
```

### `postgres://table/{schema}/{table}` - Get Table Details

Returns table schema and sample data.

**URI Pattern:** `postgres://table/{schema}/{table}`

**Example:** `postgres://table/public/users`

**Response:**

```json
{
  "schema": "public",
  "name": "users",
  "columns": [
    {"name": "id", "type": "integer", "nullable": false},
    {"name": "email", "type": "character varying", "nullable": false},
    {"name": "created_at", "type": "timestamp", "nullable": true}
  ],
  "sampleData": [
    {"id": 1, "email": "user@example.com", "created_at": "2024-01-15T10:30:00Z"}
  ]
}
```

## Common Usage Patterns

### Exploring Database Structure

```typescript
// First, list all tables
// Resource: postgres://tables

// Then inspect specific table
// Resource: postgres://table/public/users

// Query for specific data
{
  "name": "query",
  "arguments": {
    "sql": "SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'users'"
  }
}
```

### Data Analysis Queries

```typescript
// Aggregate analysis
{
  "name": "query",
  "arguments": {
    "sql": "SELECT DATE(created_at) as date, COUNT(*) as user_count FROM users GROUP BY DATE(created_at) ORDER BY date DESC LIMIT 30"
  }
}

// Join queries
{
  "name": "query",
  "arguments": {
    "sql": "SELECT u.email, COUNT(o.id) as order_count FROM users u LEFT JOIN orders o ON u.id = o.user_id GROUP BY u.email ORDER BY order_count DESC LIMIT 10"
  }
}
```

### Write Operations (When Enabled)

```typescript
// Enable in configuration first
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}

// Insert data
{
  "name": "query",
  "arguments": {
    "sql": "INSERT INTO users (email, created_at) VALUES ('new@example.com', NOW()) RETURNING id"
  }
}

// Update data
{
  "name": "query",
  "arguments": {
    "sql": "UPDATE users SET active = false WHERE last_login < NOW() - INTERVAL '90 days'"
  }
}
```

## Development Workflow

### Quick Start with Docker

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample tables included: `users`, `products`, `orders`, `order_items`

### Running Locally

```bash
# Stdio mode
bun run index.ts -- --transport=stdio

# HTTP mode
bun run index.ts -- --transport=http

# With debug logging
DEBUG=true bun run index.ts -- --transport=stdio

# Run tests
bun test
```

### Building for Production

```bash
# Build JavaScript bundle
bun run build:js

# Output in lib/index.js
```

## Example Prompts for LLMs

When using this MCP server with an AI assistant, try these prompts:

**Basic exploration:**
- "Show me all tables in the database"
- "What's the schema of the users table?"
- "Show me the first 10 rows from the orders table"

**Analysis:**
- "How many active users do we have?"
- "Show me the top 5 products by order count"
- "What's the average order value by month?"

**Complex queries:**
- "Find users who haven't ordered in the last 30 days"
- "Show me the revenue trend for the last 6 months"
- "Which products are most frequently purchased together?"

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to database

**Solutions:**
- Verify `DATABASE_URL` format: `postgresql://username:password@host:port/database`
- Check database is running: `pg_isready -h localhost -p 5432`
- Verify credentials and permissions
- For SSL connections, set `PG_SSL_ROOT_CERT` path

### Permission Errors

**Problem:** "permission denied" errors

**Solutions:**
- Ensure database user has SELECT privileges: `GRANT SELECT ON ALL TABLES IN SCHEMA public TO username;`
- For writes, verify `DANGEROUSLY_ALLOW_WRITE_OPS=true` is set
- Check user has INSERT/UPDATE/DELETE permissions if needed

### Query Timeouts

**Problem:** Long-running queries timeout

**Solutions:**
- Add `LIMIT` clauses to queries
- Create indexes on frequently queried columns
- Use more specific WHERE clauses
- Consider pagination for large result sets

### Debug Mode

Enable verbose logging to diagnose issues:

```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DEBUG": "true"
  }
}
```

## Security Best Practices

1. **Read-only by default**: Never enable `DANGEROUSLY_ALLOW_WRITE_OPS` unless absolutely necessary
2. **Use dedicated user**: Create a database user with minimal required permissions
3. **Restrict access**: Use connection string with limited scope (specific database, read-only)
4. **SSL/TLS**: Use encrypted connections for production databases
5. **Audit queries**: Monitor and log all queries executed through the MCP server

**Example read-only user setup:**

```sql
CREATE USER mcp_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE mydb TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_readonly;
```
