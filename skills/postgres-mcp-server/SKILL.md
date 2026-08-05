---
name: postgres-mcp-server
description: Query and analyze PostgreSQL databases through MCP with controlled SQL execution and schema inspection
triggers:
  - query my postgres database
  - analyze data in postgresql
  - show me database tables and schemas
  - execute sql query through mcp
  - inspect postgres table structure
  - get sample data from database
  - connect to postgresql with mcp
  - explore database schema
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol server that enables LLMs to query and analyze PostgreSQL databases through a controlled interface. Supports schema inspection, SQL query execution, and safe read-only operations by default.

## Installation

### NPX (Recommended)

Add to your MCP client settings (e.g., Claude Desktop, Cursor):

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

### Local Development

```bash
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install
bun run build:js
```

Then configure with absolute path:

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

- **`DATABASE_URL`** (required) - PostgreSQL connection string
  ```
  postgresql://username:password@host:port/database
  ```

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (default: `false`) - Enable INSERT, UPDATE, DELETE operations
  ```json
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
  ```

- **`DEBUG`** (default: `false`) - Enable debug logging
  ```json
  "env": {
    "DEBUG": "true"
  }
  ```

- **`PG_SSL_ROOT_CERT`** (optional) - Path to TLS CA bundle for SSL connections (e.g., AWS RDS)
  ```json
  "env": {
    "PG_SSL_ROOT_CERT": "/path/to/rds-ca-bundle.pem"
  }
  ```

### Transport Modes

```bash
# stdio (default, for MCP clients)
pg-mcp-server --transport=stdio

# HTTP (serves endpoint at /mcp on port 3000)
pg-mcp-server --transport=http
PORT=8080 pg-mcp-server --transport=http
```

## Tools

### `query` - Execute SQL Queries

Execute SELECT statements (or writes if enabled):

```typescript
// Tool call
{
  "sql": "SELECT id, email, created_at FROM users WHERE active = true LIMIT 10"
}
```

**Read-only mode (default):**
- Only SELECT statements allowed
- Returns error for INSERT/UPDATE/DELETE/DROP/etc.

**Write mode (DANGEROUSLY_ALLOW_WRITE_OPS=true):**
```typescript
{
  "sql": "INSERT INTO users (email, name) VALUES ('user@example.com', 'John Doe')"
}

{
  "sql": "UPDATE products SET price = 99.99 WHERE id = 123"
}
```

**Response format:**
```json
{
  "rows": [
    {"id": 1, "email": "user@example.com", "created_at": "2024-01-15T10:30:00Z"},
    {"id": 2, "email": "admin@example.com", "created_at": "2024-01-16T11:45:00Z"}
  ],
  "rowCount": 2
}
```

## Resources

### `postgres://tables` - List All Tables

Returns all tables in the database with their schemas:

```typescript
// Resource URI
"postgres://tables"

// Response
{
  "tables": [
    {"schema": "public", "table": "users"},
    {"schema": "public", "table": "products"},
    {"schema": "analytics", "table": "events"}
  ]
}
```

### `postgres://table/{schema}/{table}` - Get Table Details

Retrieve complete table schema and sample data:

```typescript
// Resource URI
"postgres://table/public/users"

// Response
{
  "schema": {
    "columns": [
      {"name": "id", "type": "integer", "nullable": false},
      {"name": "email", "type": "character varying", "nullable": false},
      {"name": "created_at", "type": "timestamp with time zone", "nullable": true}
    ],
    "indexes": [
      {"name": "users_pkey", "columns": ["id"], "unique": true},
      {"name": "users_email_idx", "columns": ["email"], "unique": true}
    ]
  },
  "sampleData": [
    {"id": 1, "email": "user@example.com", "created_at": "2024-01-15T10:30:00Z"}
  ]
}
```

## Common Usage Patterns

### Analyzing User Data

```
Show me the first 10 active users sorted by creation date
```

Agent will use:
```typescript
{
  "sql": "SELECT id, email, name, created_at FROM users WHERE active = true ORDER BY created_at DESC LIMIT 10"
}
```

### Exploring Database Schema

```
What tables are in my database and what do they contain?
```

Agent will:
1. Call resource `postgres://tables`
2. Inspect key tables with `postgres://table/public/users`
3. Summarize structure and relationships

### Aggregate Queries

```
How many orders were placed each month in 2024?
```

```typescript
{
  "sql": "SELECT DATE_TRUNC('month', order_date) as month, COUNT(*) as order_count FROM orders WHERE EXTRACT(YEAR FROM order_date) = 2024 GROUP BY month ORDER BY month"
}
```

### Joins and Complex Queries

```
Show me the top 5 customers by total order value
```

```typescript
{
  "sql": "SELECT u.id, u.email, u.name, SUM(o.total_amount) as total_spent FROM users u JOIN orders o ON u.id = o.user_id GROUP BY u.id, u.email, u.name ORDER BY total_spent DESC LIMIT 5"
}
```

### Data Analysis Workflow

1. **List tables**: Use `postgres://tables` resource
2. **Inspect schema**: Use `postgres://table/{schema}/{table}` for relevant tables
3. **Query data**: Use `query` tool with appropriate SQL
4. **Iterate**: Refine queries based on results

## Quick Start with Docker

Test the server with sample data:

```bash
# Clone and setup
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install

# Start PostgreSQL with sample data (users, products, orders, order_items)
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop database
bun run db:stop
```

## Development

### Running Locally

```bash
# stdio transport
bun run index.ts -- --transport=stdio
DEBUG=true bun run index.ts -- --transport=stdio

# HTTP transport
bun run index.ts -- --transport=http
DEBUG=true bun run index.ts -- --transport=http
```

### Testing

```bash
bun test
```

### Building

```bash
bun run build:js
```

## Troubleshooting

### Connection Issues

**Problem**: "Connection refused" or "ECONNREFUSED"

**Solution**: Verify DATABASE_URL and ensure PostgreSQL is running:
```bash
psql $DATABASE_URL -c "SELECT 1"
```

### SSL/TLS Errors

**Problem**: "SSL SYSCALL error" or certificate verification failures

**Solution**: Set PG_SSL_ROOT_CERT for managed databases:
```json
"env": {
  "DATABASE_URL": "postgresql://...",
  "PG_SSL_ROOT_CERT": "/path/to/ca-bundle.pem"
}
```

For AWS RDS:
```bash
wget https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
```

### Write Operations Blocked

**Problem**: "Write operations are disabled"

**Solution**: Enable writes explicitly (use with caution):
```json
"env": {
  "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
}
```

### Query Timeout

**Problem**: Long-running queries hang

**Solution**: Add timeout to connection string:
```
postgresql://user:pass@host:5432/db?statement_timeout=30000
```

### Schema/Table Not Found

**Problem**: "relation does not exist"

**Solution**: 
- Check schema name (default is `public`)
- Use fully qualified names: `schema.table`
- List available tables with `postgres://tables` resource

### Debug Logging

Enable detailed logs:
```json
"env": {
  "DEBUG": "true"
}
```

## Example Prompts

- "Show me the first 5 users from the database"
- "What's the schema of the orders table?"
- "How many products are in stock?"
- "Find all orders from the last 30 days"
- "What indexes exist on the users table?"
- "Calculate average order value by month"
- "Show me tables in the analytics schema"

## Security Best Practices

1. **Use read-only by default** - Only enable writes when necessary
2. **Limit permissions** - Use database users with minimal required privileges
3. **Connection pooling** - For production, use connection pooling proxy
4. **Secrets management** - Store DATABASE_URL in secure environment variables
5. **SSL/TLS** - Always use encrypted connections in production
