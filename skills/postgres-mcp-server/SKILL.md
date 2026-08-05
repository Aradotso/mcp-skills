---
name: postgres-mcp-server
description: MCP server that enables LLMs to query and analyze PostgreSQL databases through a controlled interface with read-only default mode
triggers:
  - query my postgres database
  - analyze database tables
  - get postgres schema information
  - execute sql query through mcp
  - inspect database structure
  - run database analysis
  - connect to postgresql via mcp
  - explore postgres tables
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that provides LLMs with controlled access to PostgreSQL databases. Supports both stdio and HTTP transports, with read-only operations by default and optional write operations through configuration.

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
        "DATABASE_URL": "postgresql://username:password@localhost:5432/dbname"
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

- **`DATABASE_URL`** (required): PostgreSQL connection string
  - Format: `postgresql://user:password@host:port/database`
  - Example: `postgresql://postgres:postgres@localhost:5432/mydb`

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (optional, default: `false`): Enable INSERT, UPDATE, DELETE operations
  - Set to `"true"` to allow writes
  - **Warning**: Only enable in trusted environments

- **`DEBUG`** (optional, default: `false`): Enable debug logging
  - Set to `"true"` for verbose output

- **`PG_SSL_ROOT_CERT`** (optional): Path to TLS CA bundle
  - Useful for AWS RDS or other SSL-required connections
  - Example: `/path/to/rds-ca-bundle.pem`

### Transport Modes

**stdio (default)**: Standard input/output for MCP clients like Claude Desktop

```bash
pg-mcp-server --transport=stdio
```

**HTTP**: Streamable HTTP endpoint at `/mcp` on port 3000 (or `PORT` env var)

```bash
pg-mcp-server --transport=http
# Or with custom port
PORT=8080 pg-mcp-server --transport=http
```

## Available Tools

### query

Execute SQL queries against the PostgreSQL database.

**Parameters:**
- `sql` (string, required): The SQL query to execute

**Read-only mode** (default): Only SELECT queries allowed

```typescript
// Example tool call
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, name, email FROM users WHERE active = true LIMIT 10"
  }
}
```

**Write mode** (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):

```typescript
{
  "name": "query",
  "arguments": {
    "sql": "INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com')"
  }
}
```

### Common Query Patterns

**List all tables:**
```sql
SELECT table_schema, table_name 
FROM information_schema.tables 
WHERE table_schema NOT IN ('pg_catalog', 'information_schema')
ORDER BY table_schema, table_name;
```

**Get table row count:**
```sql
SELECT COUNT(*) as total FROM users;
```

**Analyze data distribution:**
```sql
SELECT status, COUNT(*) as count 
FROM orders 
GROUP BY status 
ORDER BY count DESC;
```

**Join multiple tables:**
```sql
SELECT 
  u.name as user_name,
  o.order_date,
  SUM(oi.quantity * oi.price) as total
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
GROUP BY u.name, o.order_date
ORDER BY total DESC
LIMIT 20;
```

## Available Resources

Resources provide structured access to database metadata.

### postgres://tables

Lists all tables in the database with schema information.

```typescript
// Access pattern in MCP client
// Request resource: postgres://tables
// Returns: List of all tables with schema names
```

### postgres://table/{schema}/{table}

Get detailed schema information and sample data for a specific table.

```typescript
// Example: postgres://table/public/users
// Returns:
// - Column definitions (name, type, nullable, default)
// - Primary keys and constraints
// - Sample rows (limited to prevent large responses)
```

## Code Examples

### TypeScript/Node.js Usage

```typescript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

// Connect to MCP server
const transport = new StdioClientTransport({
  command: 'npx',
  args: ['--yes', 'pg-mcp-server', '--transport', 'stdio'],
  env: {
    DATABASE_URL: process.env.DATABASE_URL,
    DEBUG: 'true'
  }
});

const client = new Client({
  name: 'postgres-client',
  version: '1.0.0'
}, {
  capabilities: {}
});

await client.connect(transport);

// Execute a query
const result = await client.callTool({
  name: 'query',
  arguments: {
    sql: 'SELECT * FROM users WHERE created_at > NOW() - INTERVAL \'7 days\''
  }
});

console.log(result);

// List all tables
const tables = await client.readResource({
  uri: 'postgres://tables'
});

console.log(tables);

// Get specific table schema
const userSchema = await client.readResource({
  uri: 'postgres://table/public/users'
});

console.log(userSchema);
```

### Development & Testing

```bash
# Start local PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Run tests
bun test

# Stop PostgreSQL
bun run db:stop
```

### Sample Data Structure

The `db:start` command creates these sample tables:

```sql
-- users table
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100) UNIQUE,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);

-- products table
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  price DECIMAL(10,2),
  stock INTEGER
);

-- orders table
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  order_date TIMESTAMP DEFAULT NOW(),
  status VARCHAR(20)
);

-- order_items table
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER REFERENCES orders(id),
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER,
  price DECIMAL(10,2)
);
```

## Common Patterns

### Database Analysis

**Identify slow queries or table sizes:**
```sql
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;
```

**Find duplicate records:**
```sql
SELECT email, COUNT(*) as count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

**Time-series aggregation:**
```sql
SELECT 
  DATE_TRUNC('day', created_at) as day,
  COUNT(*) as new_users
FROM users
WHERE created_at >= NOW() - INTERVAL '30 days'
GROUP BY day
ORDER BY day DESC;
```

### Data Exploration

**Sample random rows:**
```sql
SELECT * FROM products ORDER BY RANDOM() LIMIT 10;
```

**Get column statistics:**
```sql
SELECT 
  MIN(price) as min_price,
  MAX(price) as max_price,
  AVG(price)::DECIMAL(10,2) as avg_price,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) as median_price
FROM products;
```

## Troubleshooting

### Connection Issues

**Problem**: `ECONNREFUSED` or connection timeout

**Solution**: Verify `DATABASE_URL` format and network access
```bash
# Test connection manually
psql "postgresql://user:pass@host:port/db"

# Check if PostgreSQL is running
pg_isready -h localhost -p 5432
```

**Problem**: SSL/TLS certificate errors

**Solution**: Set `PG_SSL_ROOT_CERT` or adjust connection string
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:pass@host:5432/db?sslmode=require",
    "PG_SSL_ROOT_CERT": "/path/to/ca-bundle.pem"
  }
}
```

### Query Execution

**Problem**: "Write operations are disabled" error

**Solution**: Enable writes only if needed and in trusted environments
```json
{
  "env": {
    "DATABASE_URL": "...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

**Problem**: Query timeout or large result sets

**Solution**: Use `LIMIT` clauses and optimize queries
```sql
-- Add LIMIT to prevent huge responses
SELECT * FROM large_table LIMIT 100;

-- Use indexes for WHERE clauses
CREATE INDEX idx_users_email ON users(email);
```

### MCP Server Not Responding

**Problem**: Server starts but doesn't respond to tools/resources

**Solution**: Enable debug logging
```json
{
  "env": {
    "DATABASE_URL": "...",
    "DEBUG": "true"
  }
}
```

Check MCP client logs for error details. Restart the MCP client after configuration changes.

### Transport Mode Issues

**Problem**: HTTP transport not accessible

**Solution**: Verify port and network binding
```bash
# Check if port is in use
lsof -i :3000

# Start with custom port
PORT=8080 pg-mcp-server --transport=http
```

## Security Best Practices

1. **Use read-only mode** unless writes are absolutely necessary
2. **Limit database user permissions** — create a dedicated MCP user with minimal grants
3. **Use connection pooling limits** to prevent resource exhaustion
4. **Validate connection strings** don't expose credentials in logs
5. **Monitor query patterns** for unexpected or malicious SQL

Example restricted database user:

```sql
CREATE USER mcp_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE mydb TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_readonly;
```
