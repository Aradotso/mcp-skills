---
name: postgres-mcp-server
description: MCP server for PostgreSQL databases enabling LLMs to query and analyze databases through a controlled interface
triggers:
  - query postgres database
  - connect to postgresql via mcp
  - analyze database schema
  - run sql query through mcp
  - explore postgres tables
  - set up postgres mcp server
  - get table structure from database
  - execute database queries safely
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that provides LLMs with controlled access to PostgreSQL databases. Enables safe querying, schema inspection, and data analysis through standardized MCP tools and resources.

## Installation

### Quick Install (npx)

Add to your MCP client configuration file (e.g., `claude_desktop_config.json` or Cursor settings):

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

Use local build in MCP settings:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": ["/absolute/path/to/pg-mcp-server/lib/index.js", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/mydb"
      }
    }
  }
}
```

## Configuration

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `DATABASE_URL` | Yes | - | PostgreSQL connection string |
| `DANGEROUSLY_ALLOW_WRITE_OPS` | No | `false` | Enable INSERT/UPDATE/DELETE operations |
| `DEBUG` | No | `false` | Enable debug logging |
| `PG_SSL_ROOT_CERT` | No | - | Path to TLS CA bundle for SSL connections |

### Connection String Format

```
postgresql://[user[:password]@][host][:port][/dbname][?param1=value1&...]
```

Examples:
- Local: `postgresql://postgres:postgres@localhost:5432/myapp`
- Remote: `postgresql://user:pass@db.example.com:5432/production`
- SSL: `postgresql://user:pass@db.example.com:5432/prod?sslmode=require`

### Transport Modes

**stdio** (default) - Standard input/output for MCP clients:
```bash
pg-mcp-server --transport=stdio
```

**http** - HTTP server mode (port 3000 by default):
```bash
pg-mcp-server --transport=http
PORT=8080 pg-mcp-server --transport=http
```

HTTP endpoint available at: `http://localhost:3000/mcp`

## Available Tools

### `query` Tool

Execute SQL queries against the database. Read-only by default.

**Parameters:**
- `sql` (string, required): SQL query to execute

**Example Usage in Code:**

```typescript
// Tool call structure
{
  "name": "query",
  "arguments": {
    "sql": "SELECT id, email, created_at FROM users WHERE active = true LIMIT 10"
  }
}
```

**Common Query Patterns:**

```sql
-- Basic SELECT
SELECT * FROM products WHERE category = 'electronics' LIMIT 20;

-- JOIN queries
SELECT o.id, o.created_at, u.email, SUM(oi.quantity * oi.price) as total
FROM orders o
JOIN users u ON o.user_id = u.id
JOIN order_items oi ON oi.order_id = o.id
GROUP BY o.id, u.email
ORDER BY o.created_at DESC;

-- Aggregations
SELECT category, COUNT(*) as count, AVG(price) as avg_price
FROM products
GROUP BY category
ORDER BY count DESC;

-- WITH clauses for complex analysis
WITH monthly_sales AS (
  SELECT 
    DATE_TRUNC('month', created_at) as month,
    SUM(total_amount) as revenue
  FROM orders
  GROUP BY month
)
SELECT * FROM monthly_sales ORDER BY month DESC;
```

**Write Operations (when enabled):**

Set `DANGEROUSLY_ALLOW_WRITE_OPS=true` to enable:

```sql
-- INSERT
INSERT INTO users (email, name) VALUES ('user@example.com', 'New User');

-- UPDATE
UPDATE products SET price = price * 1.1 WHERE category = 'books';

-- DELETE
DELETE FROM temp_data WHERE created_at < NOW() - INTERVAL '30 days';
```

## Available Resources

### `postgres://tables`

Lists all tables in the database with their schemas.

**Response includes:**
- Schema name
- Table name
- Column definitions (name, type, nullable, default)
- Primary keys
- Indexes
- Foreign keys

**Usage Pattern:**

Request this resource first to understand database structure before querying.

### `postgres://table/{schema}/{table}`

Get detailed information about a specific table including:
- Full schema definition
- Sample data (first 5 rows)
- Column statistics
- Constraints

**Example:**
```
postgres://table/public/users
postgres://table/public/orders
```

## Common Workflows

### 1. Database Discovery

```typescript
// Step 1: List all tables
// Request resource: postgres://tables

// Step 2: Examine specific table
// Request resource: postgres://table/public/users

// Step 3: Run exploratory query
{
  "name": "query",
  "arguments": {
    "sql": "SELECT COUNT(*), MAX(created_at), MIN(created_at) FROM users"
  }
}
```

### 2. Data Analysis Pattern

```sql
-- User activity analysis
WITH user_stats AS (
  SELECT 
    u.id,
    u.email,
    COUNT(DISTINCT o.id) as order_count,
    SUM(o.total_amount) as lifetime_value,
    MAX(o.created_at) as last_order_date
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  GROUP BY u.id, u.email
)
SELECT 
  CASE 
    WHEN order_count = 0 THEN 'No Orders'
    WHEN order_count < 3 THEN 'Low Activity'
    WHEN order_count < 10 THEN 'Medium Activity'
    ELSE 'High Activity'
  END as segment,
  COUNT(*) as user_count,
  AVG(lifetime_value) as avg_ltv
FROM user_stats
GROUP BY segment;
```

### 3. Schema Inspection

```sql
-- Find all foreign key relationships
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
WHERE tc.constraint_type = 'FOREIGN KEY';

-- List all indexes
SELECT
    schemaname,
    tablename,
    indexname,
    indexdef
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY tablename, indexname;
```

## Development & Testing

### Quick Start with Docker

```bash
# Start test PostgreSQL with sample data
bun run db:start

# Run MCP inspector for testing
bun run inspector

# Stop database
bun run db:stop
```

Sample schema includes: `users`, `products`, `orders`, `order_items`

### Running Tests

```bash
bun test
```

### Debug Mode

```bash
DEBUG=true bun run index.ts -- --transport=stdio
```

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or timeout errors

**Solutions:**
- Verify `DATABASE_URL` format is correct
- Check database is running: `pg_isready -h localhost -p 5432`
- Test connection with psql: `psql $DATABASE_URL`
- Check firewall/security groups allow connections

### SSL/TLS Errors

**Problem:** SSL certificate verification failed

**Solutions:**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://host/db?sslmode=require",
    "PG_SSL_ROOT_CERT": "/path/to/ca-bundle.crt"
  }
}
```

For AWS RDS:
```bash
# Download RDS CA bundle
curl -o rds-ca-bundle.crt https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
```

### Write Operations Blocked

**Problem:** INSERT/UPDATE/DELETE queries return permission errors

**Solution:** Enable write operations (use with caution):
```json
{
  "env": {
    "DATABASE_URL": "postgresql://localhost/db",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### Query Timeouts

**Problem:** Long-running queries timeout

**Solutions:**
- Add `LIMIT` clauses to queries
- Use `EXPLAIN ANALYZE` to check query performance
- Add appropriate indexes
- Use connection pooling parameters: `?pool_timeout=30&query_timeout=60000`

### Schema Not Found

**Problem:** Tables not visible even though they exist

**Solutions:**
```sql
-- Check current search path
SHOW search_path;

-- List all schemas
SELECT schema_name FROM information_schema.schemata;

-- Fully qualify table names
SELECT * FROM myschema.mytable;
```

## Example Prompts for AI Agents

Use these prompts to test the MCP server:

- "Show me the first 10 users from the database"
- "What tables exist in this database?"
- "Analyze the order patterns from last month"
- "Show me the schema for the products table"
- "Find all users who haven't made an order"
- "Calculate total revenue by product category"
- "What are the foreign key relationships in this database?"

## Security Best Practices

1. **Use read-only database users** when possible
2. **Never enable `DANGEROUSLY_ALLOW_WRITE_OPS`** in production environments
3. **Use SSL/TLS** for remote connections
4. **Limit database user permissions** to only required schemas/tables
5. **Set connection limits** in PostgreSQL configuration
6. **Monitor query logs** for suspicious activity

## License

MIT - See project repository for full license text.
