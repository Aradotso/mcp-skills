---
name: postgres-mcp-server
description: Model Context Protocol server for querying and analyzing PostgreSQL databases with LLMs
triggers:
  - query the postgres database
  - show me database tables
  - analyze data in postgresql
  - get schema for database table
  - run sql query through mcp
  - connect to postgres with mcp
  - explore database structure
  - fetch data from postgres
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that enables LLMs to safely query and analyze PostgreSQL databases. Provides controlled access to database operations with configurable write permissions.

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

Use local build:

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

- **`DATABASE_URL`** (required): PostgreSQL connection string
  - Format: `postgresql://user:password@host:port/database`
  - Example: `postgresql://postgres:postgres@localhost:5432/mydb`

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (optional, default: `false`): Enable INSERT/UPDATE/DELETE operations
  - Set to `"true"` to allow write operations
  - **Warning**: Only enable for trusted use cases

- **`DEBUG`** (optional, default: `false`): Enable debug logging
  - Set to `"true"` for verbose output

- **`PG_SSL_ROOT_CERT`** (optional): Path to TLS CA bundle
  - Required for some cloud providers (AWS RDS, etc.)
  - Example: `/path/to/rds-ca-bundle.pem`

### Transport Modes

**stdio** (default): Standard input/output communication
```bash
pg-mcp-server --transport=stdio
```

**http**: HTTP server mode on port 3000 (or `PORT` env var)
```bash
pg-mcp-server --transport=http
```

HTTP endpoint: `http://localhost:3000/mcp`

## Tools

### query

Execute SQL queries against the PostgreSQL database.

**Parameters:**
- `sql` (string, required): SQL query to execute

**Example (read-only):**
```json
{
  "sql": "SELECT id, email, created_at FROM users WHERE active = true ORDER BY created_at DESC LIMIT 10"
}
```

**Example (write - requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):**
```json
{
  "sql": "INSERT INTO users (email, name) VALUES ('user@example.com', 'John Doe')"
}
```

**Example (joins):**
```json
{
  "sql": "SELECT o.id, u.email, o.total FROM orders o JOIN users u ON o.user_id = u.id WHERE o.status = 'completed'"
}
```

## Resources

### postgres://tables

Lists all tables in the database with their schemas.

**Returns:** Array of table objects with schema and table name.

**Usage pattern:**
```
"Show me all available tables in the database"
```

### postgres://table/{schema}/{table}

Get detailed schema information and sample data for a specific table.

**Parameters:**
- `schema`: Database schema name (typically `public`)
- `table`: Table name

**Returns:** 
- Table schema (columns, types, constraints)
- First 5 rows of sample data

**Usage pattern:**
```
"Show me the structure of the users table"
"Get sample data from the orders table"
```

## Common Patterns

### Initial Database Exploration

```typescript
// 1. List all tables
// User: "What tables are available?"
// MCP calls: postgres://tables

// 2. Examine table schema
// User: "Show me the users table structure"
// MCP calls: postgres://table/public/users

// 3. Query specific data
// User: "Show me recent users"
// MCP calls query tool with:
{
  "sql": "SELECT id, email, name, created_at FROM users ORDER BY created_at DESC LIMIT 10"
}
```

### Data Analysis Workflows

```typescript
// Aggregate queries
{
  "sql": "SELECT DATE(created_at) as date, COUNT(*) as count FROM orders WHERE created_at > NOW() - INTERVAL '30 days' GROUP BY DATE(created_at) ORDER BY date"
}

// Complex joins with aggregation
{
  "sql": "SELECT p.name, p.category, SUM(oi.quantity) as total_sold, SUM(oi.quantity * oi.price) as revenue FROM products p JOIN order_items oi ON p.id = oi.product_id JOIN orders o ON oi.order_id = o.id WHERE o.status = 'completed' GROUP BY p.id, p.name, p.category ORDER BY revenue DESC LIMIT 20"
}

// Window functions
{
  "sql": "SELECT user_id, order_date, total, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date DESC) as order_rank FROM orders WHERE order_date > NOW() - INTERVAL '90 days'"
}
```

### Safe Data Modification (with write ops enabled)

```typescript
// Update with WHERE clause
{
  "sql": "UPDATE users SET last_login = NOW() WHERE id = 123"
}

// Conditional insert
{
  "sql": "INSERT INTO audit_log (user_id, action, timestamp) SELECT id, 'export', NOW() FROM users WHERE id = 456"
}

// Transactional pattern (single query)
{
  "sql": "WITH updated AS (UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 789 RETURNING *) SELECT * FROM updated"
}
```

### Performance Optimization

```typescript
// Use EXPLAIN to analyze queries
{
  "sql": "EXPLAIN ANALYZE SELECT * FROM large_table WHERE indexed_column = 'value'"
}

// Limit results for exploration
{
  "sql": "SELECT * FROM large_table LIMIT 100"
}

// Use indexes effectively
{
  "sql": "SELECT id, email FROM users WHERE email = 'specific@example.com'"  // Assumes index on email
}
```

## TypeScript Development

### Local Development Setup

```bash
# Start local PostgreSQL with Docker
bun run db:start

# Run MCP server in development
DEBUG=true bun run index.ts -- --transport=stdio

# Test with MCP Inspector
bun run inspector

# Run tests
bun test

# Build for production
bun run build:js
```

### Example Integration

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

// Initialize MCP client
const transport = new StdioClientTransport({
  command: "npx",
  args: ["--yes", "pg-mcp-server", "--transport", "stdio"],
  env: {
    DATABASE_URL: process.env.DATABASE_URL,
  },
});

const client = new Client({
  name: "postgres-client",
  version: "1.0.0",
}, {
  capabilities: {},
});

await client.connect(transport);

// Call query tool
const result = await client.callTool({
  name: "query",
  arguments: {
    sql: "SELECT * FROM users LIMIT 5",
  },
});

console.log(result);
```

## Troubleshooting

### Connection Issues

**Problem:** `Error: connect ECONNREFUSED`

**Solution:** Verify PostgreSQL is running and `DATABASE_URL` is correct:
```bash
psql $DATABASE_URL -c "SELECT 1"
```

**Problem:** SSL/TLS errors with cloud databases

**Solution:** Set `PG_SSL_ROOT_CERT` environment variable:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:pass@aws-rds-host:5432/db?sslmode=require",
    "PG_SSL_ROOT_CERT": "/path/to/rds-ca-bundle.pem"
  }
}
```

### Query Execution Issues

**Problem:** `Error: permission denied for table`

**Solution:** Ensure database user has appropriate permissions:
```sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO your_user;
```

**Problem:** Write operations fail

**Solution:** Enable write operations:
```json
{
  "env": {
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### Performance Issues

**Problem:** Queries timeout or run slowly

**Solution:** 
- Add `LIMIT` clauses to large queries
- Use `EXPLAIN ANALYZE` to identify bottlenecks
- Ensure proper indexes exist
- Consider pagination for large result sets

### MCP Inspector for Testing

```bash
# Install inspector
npm install -g @modelcontextprotocol/inspector

# Run against your server
bun run inspector
# or
npx @modelcontextprotocol/inspector npx --yes pg-mcp-server --transport stdio
```

## Security Best Practices

1. **Read-only by default**: Never enable `DANGEROUSLY_ALLOW_WRITE_OPS` in production unless absolutely necessary
2. **Use limited database users**: Create dedicated users with minimal required permissions
3. **Environment variables**: Never commit `DATABASE_URL` with credentials to version control
4. **Network security**: Use SSL/TLS connections for cloud databases
5. **Query validation**: LLMs may generate unexpected queries—monitor and log all operations

## Example Prompts for LLMs

```
"Show me the first 5 users from the database"
"What tables are available in this database?"
"Analyze sales trends for the last 30 days"
"Find all orders with status 'pending' grouped by user"
"Show me the schema for the products table"
"Count active users by registration date"
"Get the top 10 products by revenue"
```
