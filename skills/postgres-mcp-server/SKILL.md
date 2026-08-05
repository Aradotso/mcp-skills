---
name: postgres-mcp-server
description: MCP server for PostgreSQL databases that enables LLMs to query and analyze database schemas, tables, and data through a controlled interface
triggers:
  - query my postgres database
  - analyze database schema
  - explore database tables
  - run sql query through mcp
  - connect to postgresql database
  - inspect database structure
  - get table information from postgres
  - execute database queries safely
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that provides LLMs with controlled access to PostgreSQL databases. Enables querying, schema inspection, and data analysis through standardized MCP tools and resources.

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
        "DATABASE_URL": "postgresql://user:password@host:5432/dbname"
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

- **`DATABASE_URL`** (required): PostgreSQL connection string
  ```
  postgresql://username:password@hostname:port/database
  ```

- **`DANGEROUSLY_ALLOW_WRITE_OPS`** (optional, default: `false`): Enable INSERT, UPDATE, DELETE operations
  ```json
  "env": {
    "DATABASE_URL": "postgresql://...",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
  ```

- **`DEBUG`** (optional, default: `false`): Enable verbose logging
  ```json
  "env": {
    "DEBUG": "true"
  }
  ```

- **`PG_SSL_ROOT_CERT`** (optional): Path to TLS CA bundle for SSL connections (e.g., AWS RDS)
  ```json
  "env": {
    "PG_SSL_ROOT_CERT": "/path/to/rds-ca-bundle.pem"
  }
  ```

### Transport Modes

**stdio** (default): Standard input/output communication
```bash
pg-mcp-server --transport=stdio
```

**http**: Streamable HTTP endpoint at `/mcp` on port 3000 (or `PORT` env var)
```bash
pg-mcp-server --transport=http
PORT=8080 pg-mcp-server --transport=http
```

## MCP Tools

### query

Execute SQL queries against the database.

**Read-only by default** — only SELECT queries allowed unless `DANGEROUSLY_ALLOW_WRITE_OPS=true`.

**Request:**
```typescript
{
  "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
}
```

**Response:**
```json
{
  "rows": [
    {"id": 1, "email": "alice@example.com", "active": true},
    {"id": 2, "email": "bob@example.com", "active": true}
  ],
  "rowCount": 2
}
```

**Complex query example:**
```typescript
{
  "sql": `
    SELECT 
      o.id, 
      o.created_at, 
      u.email, 
      COUNT(oi.id) as item_count,
      SUM(oi.quantity * p.price) as total
    FROM orders o
    JOIN users u ON o.user_id = u.id
    JOIN order_items oi ON o.id = oi.order_id
    JOIN products p ON oi.product_id = p.id
    WHERE o.created_at > NOW() - INTERVAL '30 days'
    GROUP BY o.id, u.email
    ORDER BY total DESC
    LIMIT 20
  `
}
```

**Write operations** (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):
```typescript
{
  "sql": "UPDATE users SET last_login = NOW() WHERE id = 123"
}
```

## MCP Resources

### postgres://tables

List all tables in the database with row counts.

**URI:** `postgres://tables`

**Response:**
```typescript
{
  "contents": [
    {
      "uri": "postgres://tables",
      "mimeType": "application/json",
      "text": JSON.stringify([
        {
          "schema": "public",
          "table": "users",
          "rowCount": 1523
        },
        {
          "schema": "public",
          "table": "orders",
          "rowCount": 8421
        }
      ])
    }
  ]
}
```

### postgres://table/{schema}/{table}

Get detailed schema information and sample data for a specific table.

**URI:** `postgres://table/public/users`

**Response:**
```typescript
{
  "schema": [
    {
      "column": "id",
      "type": "integer",
      "nullable": false,
      "default": "nextval('users_id_seq'::regclass)"
    },
    {
      "column": "email",
      "type": "character varying(255)",
      "nullable": false,
      "default": null
    },
    {
      "column": "created_at",
      "type": "timestamp with time zone",
      "nullable": false,
      "default": "CURRENT_TIMESTAMP"
    }
  ],
  "sampleData": [
    {"id": 1, "email": "alice@example.com", "created_at": "2024-01-15T10:30:00Z"},
    {"id": 2, "email": "bob@example.com", "created_at": "2024-01-16T14:22:00Z"}
  ]
}
```

## Common Patterns

### Database Exploration

```typescript
// 1. List all tables
// Use resource: postgres://tables

// 2. Inspect specific table
// Use resource: postgres://table/public/users

// 3. Explore relationships
{
  "sql": `
    SELECT 
      tc.table_name, 
      kcu.column_name,
      ccu.table_name AS foreign_table_name,
      ccu.column_name AS foreign_column_name
    FROM information_schema.table_constraints tc
    JOIN information_schema.key_column_usage kcu
      ON tc.constraint_name = kcu.constraint_name
    JOIN information_schema.constraint_column_usage ccu
      ON tc.constraint_name = ccu.constraint_name
    WHERE tc.constraint_type = 'FOREIGN KEY'
      AND tc.table_schema = 'public'
  `
}
```

### Data Analysis

```typescript
// Aggregation query
{
  "sql": `
    SELECT 
      DATE_TRUNC('day', created_at) as date,
      COUNT(*) as orders,
      AVG(total_amount) as avg_amount,
      SUM(total_amount) as revenue
    FROM orders
    WHERE created_at > NOW() - INTERVAL '90 days'
    GROUP BY DATE_TRUNC('day', created_at)
    ORDER BY date DESC
  `
}
```

### Performance Investigation

```typescript
// Table sizes
{
  "sql": `
    SELECT 
      schemaname,
      tablename,
      pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
    FROM pg_tables
    WHERE schemaname = 'public'
    ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
  `
}

// Index usage
{
  "sql": `
    SELECT 
      schemaname,
      tablename,
      indexname,
      idx_scan as scans,
      idx_tup_read as tuples_read,
      idx_tup_fetch as tuples_fetched
    FROM pg_stat_user_indexes
    ORDER BY idx_scan ASC
  `
}
```

### Safe Data Sampling

```typescript
// Random sample (efficient for large tables)
{
  "sql": `
    SELECT * FROM large_table 
    TABLESAMPLE BERNOULLI(1) 
    LIMIT 100
  `
}

// Diverse sample across time ranges
{
  "sql": `
    WITH buckets AS (
      SELECT 
        WIDTH_BUCKET(
          EXTRACT(EPOCH FROM created_at),
          EXTRACT(EPOCH FROM MIN(created_at) OVER ()),
          EXTRACT(EPOCH FROM MAX(created_at) OVER ()),
          10
        ) as bucket
      FROM events
    )
    SELECT e.* 
    FROM events e
    JOIN buckets b ON true
    WHERE RANDOM() < 0.1
    LIMIT 100
  `
}
```

## Development & Testing

### Quick Start with Docker

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample schema includes: `users`, `products`, `orders`, `order_items`

### Running Locally

```bash
# Clone repository
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server

# Install dependencies
bun install

# Run in stdio mode
bun run index.ts -- --transport=stdio

# Run in http mode
bun run index.ts -- --transport=http

# With debug logging
DEBUG=true bun run index.ts -- --transport=stdio

# Run tests
bun test
```

### Building for Distribution

```bash
# Build JavaScript bundle
bun run build:js

# Output: lib/index.js
```

## Troubleshooting

### Connection Issues

**Problem:** `Error: connect ECONNREFUSED`

**Solution:** Verify `DATABASE_URL` format and database accessibility:
```bash
psql "postgresql://user:password@host:5432/dbname"
```

**Problem:** SSL/TLS errors with cloud databases

**Solution:** Use `PG_SSL_ROOT_CERT` for AWS RDS or similar:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://...",
    "PG_SSL_ROOT_CERT": "/path/to/rds-combined-ca-bundle.pem"
  }
}
```

### Query Restrictions

**Problem:** Write queries fail with "Write operations are disabled"

**Solution:** Enable writes explicitly (use with caution):
```json
{
  "env": {
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### Performance Issues

**Problem:** Queries timeout or consume too much memory

**Solution:** Add `LIMIT` clauses and use pagination:
```sql
SELECT * FROM large_table 
ORDER BY id 
LIMIT 1000 OFFSET 0
```

**Problem:** Complex aggregations slow

**Solution:** Use materialized views or pre-computed tables:
```sql
CREATE MATERIALIZED VIEW daily_stats AS
SELECT DATE_TRUNC('day', created_at) as date, COUNT(*) 
FROM events 
GROUP BY date;
```

### Transport Mode Issues

**Problem:** MCP client doesn't connect via HTTP

**Solution:** Ensure client supports Streamable HTTP and connects to `http://localhost:3000/mcp` (or your configured `PORT`).

**Problem:** stdio mode shows no output

**Solution:** Ensure client is configured for stdio transport and check `DEBUG=true` logs.

## Security Best Practices

1. **Read-only by default**: Never enable `DANGEROUSLY_ALLOW_WRITE_OPS` in production
2. **Dedicated database user**: Create a user with minimal permissions
3. **Connection pooling**: Use connection string parameters like `?pool_min=1&pool_max=5`
4. **Query validation**: LLM should avoid user-controlled input in SQL
5. **SSL/TLS**: Always use encrypted connections for remote databases

Example restricted user:
```sql
CREATE USER mcp_readonly WITH PASSWORD 'strong_password';
GRANT CONNECT ON DATABASE mydb TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
  GRANT SELECT ON TABLES TO mcp_readonly;
```
