---
name: postgres-mcp-server
description: MCP server enabling LLMs to query and analyze PostgreSQL databases through the Model Context Protocol
triggers:
  - query postgres database
  - inspect postgres tables
  - analyze database schema
  - run sql query through mcp
  - connect to postgresql database
  - explore postgres data
  - setup postgres mcp server
  - get table structure from postgres
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol server that provides LLMs with controlled access to PostgreSQL databases. Enables querying, schema inspection, and data analysis through standardized MCP tools and resources.

## Installation

### Quick Install (npx)

Add to your MCP client configuration (e.g., `~/Library/Application Support/Claude/claude_desktop_config.json`):

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

Local configuration:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": ["/absolute/path/to/pg-mcp-server/lib/index.js", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/postgres"
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

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (default: `false`): Enable INSERT, UPDATE, DELETE operations
  - Set to `true` to allow write operations
  - **Warning**: Only enable in trusted environments

- **`DEBUG`** (default: `false`): Enable debug logging
  - Set to `true` for detailed logs

- **`PG_SSL_ROOT_CERT`**: Path to TLS CA bundle for SSL connections
  - Example: `/path/to/rds-ca-bundle.pem`
  - Useful for AWS RDS and other managed databases

### Transport Modes

The server supports two transport modes:

1. **stdio** (default): Standard input/output for local MCP clients
2. **http**: HTTP server with Streamable HTTP endpoint at `/mcp`

```bash
# stdio transport
pg-mcp-server --transport=stdio

# HTTP transport (default port 3000)
pg-mcp-server --transport=http

# Custom port
PORT=8080 pg-mcp-server --transport=http
```

## Available Tools

### query

Execute SQL queries against the database.

**Parameters:**
- `sql` (string, required): SQL query to execute

**Example:**

```typescript
// Tool invocation
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, name, email FROM users WHERE active = true LIMIT 10"
  }
}
```

**Read-only mode** (default):
- SELECT statements allowed
- INSERT, UPDATE, DELETE blocked

**Write mode** (`DANGEROUSLY_ALLOW_WRITE_OPS=true`):
- All SQL operations allowed
- Use with caution in production

## Available Resources

### postgres://tables

Lists all tables in the database with row counts.

**URI:** `postgres://tables`

**Returns:**
```json
{
  "tables": [
    {
      "schema": "public",
      "name": "users",
      "rows": 1523
    },
    {
      "schema": "public",
      "name": "orders",
      "rows": 8491
    }
  ]
}
```

### postgres://table/{schema}/{table}

Get detailed schema information and sample data for a specific table.

**URI Pattern:** `postgres://table/{schema}/{table}`

**Example:** `postgres://table/public/users`

**Returns:**
- Column definitions (name, type, nullable, default)
- Primary key information
- Foreign key relationships
- Indexes
- Sample rows (up to 5)

## Common Usage Patterns

### Exploring Database Schema

```typescript
// Ask the agent:
// "What tables are in the database?"

// Agent uses: postgres://tables resource
// Response includes all tables with row counts

// Follow-up: "Show me the structure of the users table"
// Agent uses: postgres://table/public/users resource
```

### Basic Queries

```typescript
// "Show me the first 10 active users"
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, name, email, created_at FROM users WHERE active = true ORDER BY created_at DESC LIMIT 10"
  }
}
```

### Aggregation and Analysis

```typescript
// "How many orders were placed each month this year?"
{
  "name": "query",
  "arguments": {
    "sql": "SELECT DATE_TRUNC('month', order_date) as month, COUNT(*) as order_count FROM orders WHERE EXTRACT(YEAR FROM order_date) = EXTRACT(YEAR FROM CURRENT_DATE) GROUP BY month ORDER BY month"
  }
}
```

### Joining Tables

```typescript
// "Show me recent orders with customer information"
{
  "name": "query",
  "arguments": {
    "sql": "SELECT o.id, o.order_date, o.total, u.name, u.email FROM orders o JOIN users u ON o.user_id = u.id ORDER BY o.order_date DESC LIMIT 20"
  }
}
```

### Data Validation

```typescript
// "Find users with invalid email formats"
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, email FROM users WHERE email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Z|a-z]{2,}$'"
  }
}
```

## Development & Testing

### Running Locally with TypeScript

```bash
# stdio mode
bun run index.ts -- --transport=stdio

# HTTP mode
bun run index.ts -- --transport=http

# Debug mode
DEBUG=true bun run index.ts -- --transport=stdio
```

### Docker Quick Start

```bash
# Start PostgreSQL with sample data (users, products, orders tables)
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

### Testing with MCP Inspector

```bash
# Install inspector globally
npm install -g @modelcontextprotocol/inspector

# Run inspector
npx @modelcontextprotocol/inspector node /path/to/pg-mcp-server/lib/index.js --transport stdio
```

## Real-World Examples

### E-commerce Analytics

```typescript
// "What's the total revenue by product category this quarter?"
{
  "name": "query",
  "arguments": {
    "sql": "SELECT p.category, SUM(oi.quantity * oi.price) as revenue FROM order_items oi JOIN products p ON oi.product_id = p.id JOIN orders o ON oi.order_id = o.id WHERE o.order_date >= DATE_TRUNC('quarter', CURRENT_DATE) GROUP BY p.category ORDER BY revenue DESC"
  }
}
```

### User Activity Analysis

```typescript
// "Find users who haven't logged in for 90 days"
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, name, email, last_login FROM users WHERE last_login < CURRENT_DATE - INTERVAL '90 days' AND active = true ORDER BY last_login"
  }
}
```

### Performance Monitoring

```typescript
// "Show tables larger than 1GB"
{
  "name": "query",
  "arguments": {
    "sql": "SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size FROM pg_tables WHERE pg_total_relation_size(schemaname||'.'||tablename) > 1073741824 ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC"
  }
}
```

## Troubleshooting

### Connection Issues

**Problem:** Cannot connect to database

```bash
# Verify DATABASE_URL format
echo $DATABASE_URL
# Should be: postgresql://user:password@host:port/database

# Test connection directly
psql $DATABASE_URL

# Check SSL requirements (for managed databases)
export PG_SSL_ROOT_CERT=/path/to/ca-bundle.pem
```

### Permission Errors

**Problem:** "permission denied for table"

```sql
-- Grant read access to schema
GRANT USAGE ON SCHEMA public TO your_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO your_user;

-- For write operations
GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO your_user;
```

### Write Operations Blocked

**Problem:** INSERT/UPDATE/DELETE not working

```json
{
  "mcpServers": {
    "postgres": {
      "env": {
        "DATABASE_URL": "postgresql://...",
        "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
      }
    }
  }
}
```

### Debug Mode

Enable detailed logging:

```json
{
  "env": {
    "DEBUG": "true"
  }
}
```

Or when running directly:

```bash
DEBUG=true bun run index.ts -- --transport=stdio
```

### HTTP Transport Issues

**Problem:** Cannot connect on HTTP mode

```bash
# Check port availability
lsof -i :3000

# Use custom port
PORT=8080 pg-mcp-server --transport=http

# Verify endpoint
curl http://localhost:3000/mcp
```

## Security Best Practices

1. **Use read-only credentials** when possible
2. **Never enable write operations** in production without understanding risks
3. **Use SSL/TLS** for remote database connections
4. **Limit database user permissions** to minimum required
5. **Store DATABASE_URL** in environment variables, never in code
6. **Audit SQL queries** executed through the server

## Integration with AI Coding Agents

### Example Prompts

- "Show me the database schema"
- "What are the top 10 products by sales?"
- "Analyze user signup trends by month"
- "Find duplicate email addresses in the users table"
- "Show me the structure of the orders table"
- "Calculate average order value by customer segment"

### Notebook Analysis

For data exploration, the project includes a Cursor rule for using MCP server with notebooks. Check `.cursor/rules/notebooks.mdc` in the repository for integration patterns.
