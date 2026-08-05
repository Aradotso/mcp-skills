---
name: postgres-mcp-server
description: Query and analyze PostgreSQL databases through the Model Context Protocol with controlled read/write access
triggers:
  - query my postgres database
  - show me tables in the database
  - analyze database schema
  - run sql query through mcp
  - connect to postgres via mcp
  - explore my database structure
  - execute database queries safely
  - get table schema and sample data
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol server that enables LLMs to query and analyze PostgreSQL databases through a controlled, safe interface. Supports both read-only and write operations (when explicitly enabled), table inspection, and schema exploration.

## Installation

### Quick Install (Cursor/Claude Desktop)

Add to your MCP client configuration (`claude_desktop_config.json` or `.cursor/config.json`):

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

### Local Development Install

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
        "DATABASE_URL": "postgresql://postgres:postgres@localhost:5432/postgres"
      }
    }
  }
}
```

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | Yes | - | PostgreSQL connection string (format: `postgresql://user:pass@host:port/db`) |
| `DANGEROUSLY_ALLOW_WRITE_OPS` | No | `false` | Enable INSERT, UPDATE, DELETE operations |
| `DEBUG` | No | `false` | Enable debug logging |
| `PG_SSL_ROOT_CERT` | No | - | Path to TLS CA bundle (e.g., for AWS RDS) |

### Transport Modes

**stdio** (default): For MCP clients like Claude Desktop, Cursor
```bash
npx pg-mcp-server --transport=stdio
```

**http**: For web-based or HTTP-compatible MCP clients
```bash
npx pg-mcp-server --transport=http
# Serves at http://localhost:3000/mcp
PORT=8080 npx pg-mcp-server --transport=http
```

## Available Tools

### `query` - Execute SQL Queries

Execute SELECT queries (and INSERT/UPDATE/DELETE when write ops enabled).

**Parameters:**
- `sql` (string, required): SQL query to execute

**Example:**
```typescript
// Tool call from LLM
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, email, created_at FROM users WHERE active = true ORDER BY created_at DESC LIMIT 10"
  }
}
```

**Response:**
```json
{
  "rows": [
    {"id": 1, "email": "user@example.com", "created_at": "2024-01-15T10:30:00Z"},
    {"id": 2, "email": "another@example.com", "created_at": "2024-01-14T09:20:00Z"}
  ],
  "rowCount": 2
}
```

### Query Safety

By default, only **SELECT** queries are allowed. To enable writes:

```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

**Write operations example:**
```typescript
// Only works when DANGEROUSLY_ALLOW_WRITE_OPS=true
{
  "name": "query",
  "arguments": {
    "sql": "UPDATE users SET last_login = NOW() WHERE id = 123"
  }
}
```

## Available Resources

### `postgres://tables` - List All Tables

Returns a list of all tables in the database with row counts.

**URI:** `postgres://tables`

**Response:**
```json
{
  "tables": [
    {"schema": "public", "table": "users", "rows": 1523},
    {"schema": "public", "table": "products", "rows": 342},
    {"schema": "public", "table": "orders", "rows": 8912}
  ]
}
```

### `postgres://table/{schema}/{table}` - Get Table Details

Returns table schema (columns, types, constraints) and sample data.

**URI:** `postgres://table/public/users`

**Response:**
```json
{
  "schema": {
    "columns": [
      {"name": "id", "type": "integer", "nullable": false, "default": "nextval('users_id_seq')"},
      {"name": "email", "type": "character varying(255)", "nullable": false},
      {"name": "created_at", "type": "timestamp with time zone", "nullable": false, "default": "now()"}
    ],
    "primaryKey": ["id"],
    "indexes": [
      {"name": "users_pkey", "columns": ["id"], "unique": true},
      {"name": "users_email_idx", "columns": ["email"], "unique": true}
    ]
  },
  "sampleData": [
    {"id": 1, "email": "alice@example.com", "created_at": "2024-01-10T08:00:00Z"},
    {"id": 2, "email": "bob@example.com", "created_at": "2024-01-11T09:15:00Z"}
  ]
}
```

## Common Usage Patterns

### Data Exploration

```typescript
// User prompt: "Show me the structure of my database"
// Agent should:
// 1. Call resource: postgres://tables
// 2. Present overview of tables
// 3. Optionally call postgres://table/{schema}/{table} for interesting tables

// User prompt: "What columns does the users table have?"
// Agent calls: postgres://table/public/users
```

### Analytics Queries

```typescript
// User prompt: "Show me my top 10 customers by order value"
{
  "name": "query",
  "arguments": {
    "sql": `
      SELECT 
        u.id, 
        u.email, 
        COUNT(o.id) as order_count,
        SUM(o.total) as total_spent
      FROM users u
      JOIN orders o ON u.id = o.user_id
      GROUP BY u.id, u.email
      ORDER BY total_spent DESC
      LIMIT 10
    `
  }
}
```

### Complex Joins

```typescript
// User prompt: "List products that have never been ordered"
{
  "name": "query",
  "arguments": {
    "sql": `
      SELECT p.id, p.name, p.price
      FROM products p
      LEFT JOIN order_items oi ON p.id = oi.product_id
      WHERE oi.id IS NULL
      ORDER BY p.name
    `
  }
}
```

### Aggregations & Statistics

```typescript
// User prompt: "Show me monthly revenue for 2024"
{
  "name": "query",
  "arguments": {
    "sql": `
      SELECT 
        DATE_TRUNC('month', created_at) as month,
        COUNT(*) as order_count,
        SUM(total) as revenue
      FROM orders
      WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
      GROUP BY DATE_TRUNC('month', created_at)
      ORDER BY month
    `
  }
}
```

### Safe Schema Inspection

```typescript
// User prompt: "What indexes exist on the users table?"
// Agent calls: postgres://table/public/users
// Then parses the schema.indexes field

// User prompt: "Show me all foreign key relationships"
{
  "name": "query",
  "arguments": {
    "sql": `
      SELECT
        tc.table_schema, 
        tc.table_name, 
        kcu.column_name,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name
      FROM information_schema.table_constraints AS tc
      JOIN information_schema.key_column_usage AS kcu
        ON tc.constraint_name = kcu.constraint_name
      JOIN information_schema.constraint_column_usage AS ccu
        ON ccu.constraint_name = tc.constraint_name
      WHERE tc.constraint_type = 'FOREIGN KEY'
    `
  }
}
```

## Development & Testing

### Quick Start with Docker

The repository includes a Docker Compose setup with sample data:

```bash
# Start PostgreSQL with sample tables (users, products, orders, order_items)
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

### Running Locally

```bash
# Clone repository
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
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

### Building for Distribution

```bash
bun run build:js
# Output: lib/index.js
```

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or "ECONNREFUSED"
```bash
# Check DATABASE_URL format
# Correct: postgresql://user:password@localhost:5432/dbname
# Ensure PostgreSQL is running
psql $DATABASE_URL -c "SELECT 1"
```

**Problem:** SSL certificate verification failed
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "PG_SSL_ROOT_CERT": "/path/to/ca-bundle.pem"
  }
}
```

### Permission Errors

**Problem:** "permission denied for table"
```sql
-- Grant read access to specific tables
GRANT SELECT ON TABLE users, products TO your_user;

-- Grant read access to all tables in schema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO your_user;
```

### Write Operations Blocked

**Problem:** "Write operations are not allowed"
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### MCP Server Not Responding

**Problem:** Server starts but doesn't respond to requests
```bash
# Check transport mode matches client expectations
# Cursor/Claude Desktop: use --transport=stdio (default)
# Web clients: use --transport=http

# Verify with MCP Inspector
npx @modelcontextprotocol/inspector npx pg-mcp-server --transport=stdio
```

### Query Timeout

**Problem:** Long-running queries hang
```typescript
// Add query timeout to DATABASE_URL
"DATABASE_URL": "postgresql://user:pass@host:5432/db?statement_timeout=30000"
// Or set in query
{
  "sql": "SET statement_timeout = '30s'; SELECT * FROM large_table;"
}
```

### Debug Logging

Enable detailed logging for troubleshooting:

```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DEBUG": "true"
  }
}
```

## Security Best Practices

1. **Read-Only by Default**: Never enable `DANGEROUSLY_ALLOW_WRITE_OPS` in production unless absolutely necessary
2. **Use Dedicated User**: Create a PostgreSQL user with limited privileges:
   ```sql
   CREATE USER mcp_readonly WITH PASSWORD 'secure_password';
   GRANT CONNECT ON DATABASE mydb TO mcp_readonly;
   GRANT USAGE ON SCHEMA public TO mcp_readonly;
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
   ```
3. **Connection Pooling**: For high-traffic scenarios, use connection pooling (PgBouncer)
4. **Secrets Management**: Never commit `DATABASE_URL` to version control; use environment variables

## Example Prompts for Users

- "Show me the first 5 users from the database"
- "What tables exist in my database?"
- "Describe the schema of the orders table"
- "Find all products with price greater than $100"
- "Show me total revenue by month for this year"
- "List customers who haven't placed an order in the last 30 days"
- "What are the most popular products by order count?"
