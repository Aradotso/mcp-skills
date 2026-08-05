---
name: postgres-mcp-server
description: Query and analyze PostgreSQL databases through MCP protocol, enabling LLMs to interact with database schemas, tables, and data
triggers:
  - query the postgres database
  - connect to postgresql through mcp
  - analyze database schema and tables
  - execute sql queries via mcp
  - inspect postgres table structure
  - run database queries with mcp server
  - get data from postgresql database
  - explore postgres database schema
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol server that provides controlled access to PostgreSQL databases. Enables LLMs to query schemas, inspect tables, and execute SQL queries through MCP tools and resources.

## What It Does

- **Query Execution**: Run SQL queries against PostgreSQL databases
- **Schema Inspection**: List tables and view detailed schema information
- **Sample Data**: Retrieve sample rows from tables for analysis
- **Read/Write Control**: Optional write operation support via environment flag
- **Transport Options**: Supports both stdio (default) and HTTP transports

## Installation

### Quick Install (npx)

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

Point to local build:

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

- **`DATABASE_URL`** (required): PostgreSQL connection string
  ```
  postgresql://[user]:[password]@[host]:[port]/[database]
  ```

- **`DANGEROUSLY_ALLOW_WRITE_OPS`**: Enable INSERT, UPDATE, DELETE operations
  ```
  DANGEROUSLY_ALLOW_WRITE_OPS=true
  ```

- **`DEBUG`**: Enable verbose logging
  ```
  DEBUG=true
  ```

- **`PG_SSL_ROOT_CERT`**: Path to TLS CA bundle (for AWS RDS, etc.)
  ```
  PG_SSL_ROOT_CERT=/path/to/ca-bundle.crt
  ```

### Transport Modes

**Stdio** (default, for MCP clients):
```bash
pg-mcp-server --transport=stdio
```

**HTTP** (serves at `/mcp` endpoint):
```bash
pg-mcp-server --transport=http
# Connects at http://localhost:3000/mcp
```

Override port:
```bash
PORT=8080 pg-mcp-server --transport=http
```

## MCP Tools

### `query`

Execute SQL queries against the database.

**Input Schema:**
```typescript
{
  sql: string  // SQL query to execute
}
```

**Example Usage:**
```typescript
// Read query
{
  "sql": "SELECT id, email, created_at FROM users WHERE active = true LIMIT 10"
}

// Aggregation
{
  "sql": "SELECT status, COUNT(*) as count FROM orders GROUP BY status"
}

// Join query
{
  "sql": "SELECT u.email, COUNT(o.id) as order_count FROM users u LEFT JOIN orders o ON u.id = o.user_id GROUP BY u.email"
}
```

**Write Operations** (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):
```typescript
// Insert
{
  "sql": "INSERT INTO users (email, name) VALUES ('user@example.com', 'John Doe')"
}

// Update
{
  "sql": "UPDATE users SET active = false WHERE last_login < NOW() - INTERVAL '1 year'"
}

// Delete
{
  "sql": "DELETE FROM temp_data WHERE created_at < NOW() - INTERVAL '7 days'"
}
```

**Response Format:**
```typescript
{
  content: [
    {
      type: "text",
      text: string  // JSON string of query results
    }
  ]
}
```

## MCP Resources

### `postgres://tables`

List all tables in the database with row counts.

**Response:**
```json
{
  "tables": [
    {
      "schema": "public",
      "name": "users",
      "row_count": 1523
    },
    {
      "schema": "public",
      "name": "orders",
      "row_count": 8947
    }
  ]
}
```

### `postgres://table/{schema}/{table}`

Get detailed schema and sample data for a specific table.

**URI Pattern:**
```
postgres://table/public/users
postgres://table/analytics/events
```

**Response:**
```json
{
  "schema": "public",
  "table": "users",
  "columns": [
    {
      "column_name": "id",
      "data_type": "integer",
      "is_nullable": "NO",
      "column_default": "nextval('users_id_seq'::regclass)"
    },
    {
      "column_name": "email",
      "data_type": "character varying",
      "is_nullable": "NO",
      "column_default": null
    }
  ],
  "sample_rows": [
    {
      "id": 1,
      "email": "user1@example.com",
      "created_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

## Common Patterns

### Exploratory Analysis

```typescript
// 1. List all tables
// Use resource: postgres://tables

// 2. Inspect specific table
// Use resource: postgres://table/public/users

// 3. Run analytical query
{
  "sql": `
    SELECT 
      DATE_TRUNC('month', created_at) as month,
      COUNT(*) as new_users
    FROM users
    WHERE created_at >= NOW() - INTERVAL '12 months'
    GROUP BY month
    ORDER BY month DESC
  `
}
```

### Data Validation

```typescript
// Check for duplicates
{
  "sql": "SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1"
}

// Find missing relationships
{
  "sql": "SELECT o.* FROM orders o LEFT JOIN users u ON o.user_id = u.id WHERE u.id IS NULL"
}

// Identify data quality issues
{
  "sql": "SELECT COUNT(*) FROM products WHERE price <= 0 OR name IS NULL"
}
```

### Complex Joins

```typescript
{
  "sql": `
    SELECT 
      u.email,
      COUNT(DISTINCT o.id) as total_orders,
      SUM(oi.quantity * oi.unit_price) as total_spent
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    LEFT JOIN order_items oi ON o.id = oi.order_id
    WHERE o.created_at >= NOW() - INTERVAL '30 days'
    GROUP BY u.id, u.email
    HAVING SUM(oi.quantity * oi.unit_price) > 100
    ORDER BY total_spent DESC
    LIMIT 20
  `
}
```

### Prompt Examples

Ask your AI agent:

- "Show me the schema of the users table"
- "List all tables in the database"
- "Find the top 10 customers by order value this month"
- "Check if there are any orphaned records in the orders table"
- "Analyze the distribution of order statuses"
- "Get sample data from the products table"

## Troubleshooting

### Connection Issues

**Problem**: `connection refused` or timeout errors

**Solutions**:
```bash
# Test connection manually
psql "$DATABASE_URL"

# Check host/port accessibility
telnet localhost 5432

# Verify DATABASE_URL format
echo $DATABASE_URL
# Should be: postgresql://user:pass@host:port/dbname
```

### SSL/TLS Errors

**Problem**: SSL connection errors with cloud providers (AWS RDS, etc.)

**Solution**:
```bash
# Download CA bundle (AWS RDS example)
curl -o rds-ca-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

# Set in environment
export PG_SSL_ROOT_CERT=/path/to/rds-ca-bundle.pem
```

Or add to connection string:
```
postgresql://user:pass@host:5432/db?sslmode=require&sslrootcert=/path/to/ca.pem
```

### Write Operations Blocked

**Problem**: INSERT/UPDATE/DELETE queries return permission errors

**Solution**: Enable write operations explicitly
```json
{
  "env": {
    "DATABASE_URL": "postgresql://localhost:5432/mydb",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

⚠️ **Warning**: Only enable writes for trusted databases. Consider using read-only database users.

### Debug Mode

Enable verbose logging:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://localhost:5432/mydb",
    "DEBUG": "true"
  }
}
```

Or when running directly:
```bash
DEBUG=true pg-mcp-server --transport=stdio
```

### Testing with Docker

Quick local PostgreSQL setup:
```bash
# Clone repo and start test database
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun run db:start  # Starts PostgreSQL with sample data

# Test with MCP Inspector
bun run inspector

# Clean up
bun run db:stop
```

Sample schema includes: `users`, `products`, `orders`, `order_items` tables.
