---
name: postgres-mcp-server
description: MCP Server for querying and analyzing PostgreSQL databases through LLM interfaces with controlled read/write operations
triggers:
  - query my postgres database
  - connect to postgresql through mcp
  - analyze database tables with llm
  - run sql queries safely
  - inspect postgres schema
  - list database tables
  - execute database query with ai
  - get postgres table structure
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that enables LLMs to safely query and analyze PostgreSQL databases. Provides controlled access to database operations through MCP tools and resources, supporting both read-only and optional write operations.

## What It Does

- **Query Execution**: Run SQL queries against PostgreSQL databases through MCP tools
- **Schema Inspection**: Browse tables, view structure, and sample data via MCP resources
- **Safe by Default**: Read-only mode unless explicitly enabled
- **Dual Transport**: Supports both stdio (default) and HTTP transports
- **SSL/TLS Support**: Optional certificate configuration for secure connections

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

### Local Development Install

```bash
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install
bun run build:js
```

Then use in MCP client:

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
  - Format: `postgresql://user:password@host:port/database`
  - Example: `postgresql://postgres:postgres@localhost:5432/mydb`

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (optional): Enable write operations
  - Default: `false` (read-only)
  - Set to `true` to allow INSERT, UPDATE, DELETE, etc.

- **`DEBUG`** (optional): Enable debug logging
  - Default: `false`
  - Set to `true` for verbose output

- **`PG_SSL_ROOT_CERT`** (optional): Path to TLS CA bundle
  - Useful for cloud providers (AWS RDS, etc.)
  - Example: `/path/to/rds-combined-ca-bundle.pem`

### Transport Modes

**stdio** (default): For MCP clients that communicate via stdin/stdout
```bash
pg-mcp-server --transport=stdio
```

**http**: For clients supporting MCP Streamable HTTP
```bash
pg-mcp-server --transport=http
# Serves at http://localhost:3000/mcp
```

Set custom port:
```bash
PORT=8080 pg-mcp-server --transport=http
```

## MCP Tools

### `query` Tool

Execute SQL queries against the database.

**Parameters:**
- `sql` (string): The SQL query to execute

**Example Tool Call:**
```typescript
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, email, created_at FROM users WHERE active = true LIMIT 10"
  }
}
```

**Read-Only Queries:**
```typescript
// List active users
{
  "sql": "SELECT * FROM users WHERE status = 'active' ORDER BY created_at DESC LIMIT 20"
}

// Join tables
{
  "sql": "SELECT o.id, o.total, u.email FROM orders o JOIN users u ON o.user_id = u.id WHERE o.status = 'completed'"
}

// Aggregations
{
  "sql": "SELECT product_id, COUNT(*) as order_count, SUM(quantity) as total_qty FROM order_items GROUP BY product_id ORDER BY order_count DESC"
}
```

**Write Queries (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):**
```typescript
// Insert
{
  "sql": "INSERT INTO users (email, name) VALUES ('user@example.com', 'John Doe') RETURNING id"
}

// Update
{
  "sql": "UPDATE products SET price = 29.99 WHERE id = 123"
}

// Delete
{
  "sql": "DELETE FROM sessions WHERE expires_at < NOW()"
}
```

## MCP Resources

### `postgres://tables`

Lists all tables in the database with schema information.

**URI:** `postgres://tables`

**Returns:**
- List of all tables grouped by schema
- Table names and schemas

**Example Usage:**
```
User: "Show me all tables in the database"
Agent: Reads postgres://tables resource
```

### `postgres://table/{schema}/{table}`

Get detailed information about a specific table including structure and sample data.

**URI Pattern:** `postgres://table/{schema}/{table}`

**Returns:**
- Column definitions (name, type, nullable, default)
- Primary keys and constraints
- Sample rows (up to 5)

**Example:**
```
URI: postgres://table/public/users

Returns:
- Columns: id (integer), email (varchar), created_at (timestamp)
- Sample data from the table
```

## Common Patterns

### Database Analysis Workflow

```typescript
// 1. Discover available tables
// User: "What tables do I have?"
// Agent reads: postgres://tables

// 2. Inspect table structure
// User: "Show me the users table structure"
// Agent reads: postgres://table/public/users

// 3. Query data
// User: "Find all users created in the last week"
// Agent calls: query tool
{
  "sql": "SELECT * FROM users WHERE created_at > NOW() - INTERVAL '7 days'"
}
```

### Joining Multiple Tables

```typescript
// User: "Show me recent orders with customer details"
{
  "sql": `
    SELECT 
      o.id, 
      o.created_at, 
      o.total,
      u.email, 
      u.name
    FROM orders o
    JOIN users u ON o.user_id = u.id
    WHERE o.created_at > NOW() - INTERVAL '1 day'
    ORDER BY o.created_at DESC
    LIMIT 50
  `
}
```

### Aggregation and Reporting

```typescript
// User: "What are our top-selling products this month?"
{
  "sql": `
    SELECT 
      p.name,
      SUM(oi.quantity) as units_sold,
      SUM(oi.quantity * oi.price) as revenue
    FROM order_items oi
    JOIN products p ON oi.product_id = p.id
    JOIN orders o ON oi.order_id = o.id
    WHERE o.created_at >= DATE_TRUNC('month', CURRENT_DATE)
    GROUP BY p.id, p.name
    ORDER BY revenue DESC
    LIMIT 10
  `
}
```

### Safe Pagination

```typescript
// User: "Show me the next page of users"
{
  "sql": "SELECT * FROM users ORDER BY id LIMIT 20 OFFSET 20"
}
```

## Development & Testing

### Start Local PostgreSQL with Sample Data

```bash
# Start Docker container with sample schema
bun run db:start

# Database includes: users, products, orders, order_items tables
```

### Run MCP Inspector

```bash
# Test tools and resources interactively
bun run inspector
```

### Stop PostgreSQL

```bash
bun run db:stop
```

### Run Tests

```bash
bun test
```

### Debug Mode

```bash
DEBUG=true bun run index.ts -- --transport=stdio
```

## Troubleshooting

### Connection Issues

**Problem:** `connection refused` or timeout errors

**Solution:**
- Verify `DATABASE_URL` is correct
- Check PostgreSQL is running: `pg_isready -h localhost -p 5432`
- Ensure network access (firewalls, security groups)

### SSL/TLS Errors

**Problem:** `SSL SYSCALL error` or certificate verification failures

**Solution:**
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

**Solution:**
Enable write operations explicitly:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### Query Timeouts

**Problem:** Long-running queries hang

**Solution:**
Add timeout to connection string:
```
postgresql://user:pass@host:5432/db?statement_timeout=30000
```

### Permission Denied

**Problem:** Cannot access certain tables or schemas

**Solution:**
- Check database user permissions
- Grant necessary privileges:
  ```sql
  GRANT SELECT ON ALL TABLES IN SCHEMA public TO your_user;
  ```

## Example Prompts

```
"Show me the first 5 users from the database"
"What tables are in my database?"
"Analyze the orders table structure"
"Find all products with price > 100"
"Show me users who registered this week"
"What's the total revenue by product category?"
"List tables in the public schema"
```

## Security Best Practices

1. **Use Read-Only by Default**: Only enable `DANGEROUSLY_ALLOW_WRITE_OPS` when necessary
2. **Limited User Privileges**: Create a dedicated database user with minimal permissions
3. **Connection String Security**: Store `DATABASE_URL` in secure environment configuration
4. **SSL/TLS**: Use encrypted connections for production databases
5. **Query Review**: Always review generated SQL before execution in write mode
