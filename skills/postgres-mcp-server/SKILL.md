---
name: postgres-mcp-server
description: MCP Server for PostgreSQL databases - enables LLMs to query and analyze Postgres through a controlled interface with read/write capabilities.
triggers:
  - query the postgres database
  - connect to postgresql using mcp
  - analyze data in my postgres database
  - set up postgres mcp server
  - execute sql query through mcp
  - inspect postgres table schema
  - query database tables with llm
  - configure postgres mcp access
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

The Postgres MCP Server is a Model Context Protocol server that provides LLMs with controlled access to PostgreSQL databases. It enables natural language querying, schema inspection, and data analysis through MCP tools and resources.

## Installation

### Quick Install (Cursor/Claude)

Add to your MCP client settings (e.g., `~/Library/Application Support/Claude/claude_desktop_config.json` or `.cursor/config.json`):

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

### Environment Variables

- `DATABASE_URL` - PostgreSQL connection string (required)
- `DANGEROUSLY_ALLOW_WRITE_OPS` - Enable INSERT/UPDATE/DELETE operations (default: `false`)
- `DEBUG` - Enable debug logging (default: `false`)
- `PG_SSL_ROOT_CERT` - Path to TLS CA bundle for SSL connections (e.g., AWS RDS)

### Connection String Format

```bash
postgresql://[user[:password]@][host][:port][/dbname][?param1=value1&...]
```

Examples:
```bash
# Local development
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/myapp"

# Production with SSL
DATABASE_URL="postgresql://user:pass@db.example.com:5432/prod?sslmode=require"

# AWS RDS
DATABASE_URL="postgresql://admin:secret@mydb.rds.amazonaws.com:5432/production"
PG_SSL_ROOT_CERT="/path/to/rds-ca-bundle.pem"
```

## Transport Modes

### stdio (Default)

Standard input/output transport for local MCP clients:

```bash
npx pg-mcp-server --transport=stdio
```

### HTTP (Streamable)

HTTP server mode for remote MCP clients:

```bash
npx pg-mcp-server --transport=http
# Serves at http://localhost:3000/mcp

PORT=8080 npx pg-mcp-server --transport=http
# Custom port
```

## Available Tools

### `query` Tool

Execute SQL queries against the database.

**Parameters:**
- `sql` (string, required) - The SQL query to execute

**Example prompts:**
- "Show me the first 10 active users"
- "Count orders grouped by status"
- "Find products with price greater than $100"

**Read-only mode (default):**
```typescript
// Only SELECT queries allowed
{
  "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
}
```

**Write mode (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):**
```typescript
// INSERT
{
  "sql": "INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com') RETURNING *"
}

// UPDATE
{
  "sql": "UPDATE products SET price = price * 1.1 WHERE category = 'electronics'"
}

// DELETE
{
  "sql": "DELETE FROM sessions WHERE expires_at < NOW()"
}
```

## Available Resources

### `postgres://tables`

Lists all tables in the database with their schemas and row counts.

**Example prompt:**
- "What tables are in the database?"
- "Show me all available tables"

**Returns:**
```typescript
{
  "public": {
    "users": { "rows": 1250 },
    "products": { "rows": 450 },
    "orders": { "rows": 3200 }
  }
}
```

### `postgres://table/{schema}/{table}`

Get detailed schema information and sample data for a specific table.

**Example prompts:**
- "Show me the schema of the users table"
- "What columns does the orders table have?"
- "Give me sample data from products"

**Returns:**
- Column names, types, constraints
- First 5 rows as sample data
- Primary keys, foreign keys, indexes

## Common Usage Patterns

### Data Analysis

```typescript
// Aggregate queries
"What's the average order value by month for 2024?"
// Translates to:
{
  "sql": `
    SELECT 
      DATE_TRUNC('month', created_at) as month,
      AVG(total_amount) as avg_order_value,
      COUNT(*) as order_count
    FROM orders
    WHERE EXTRACT(year FROM created_at) = 2024
    GROUP BY DATE_TRUNC('month', created_at)
    ORDER BY month
  `
}

// Join queries
"Show me users with their order counts and total spent"
{
  "sql": `
    SELECT 
      u.id,
      u.name,
      u.email,
      COUNT(o.id) as order_count,
      COALESCE(SUM(o.total_amount), 0) as total_spent
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    GROUP BY u.id, u.name, u.email
    ORDER BY total_spent DESC
    LIMIT 20
  `
}
```

### Schema Inspection

```typescript
// Table structure
"Describe the structure of the products table"
// Uses resource: postgres://table/public/products

// Find relationships
"What foreign keys does the orders table have?"
{
  "sql": `
    SELECT
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
      AND tc.table_name = 'orders'
  `
}
```

### Data Exploration

```typescript
// Distribution analysis
"Show me the distribution of users by signup date"
{
  "sql": `
    SELECT 
      DATE_TRUNC('day', created_at) as signup_date,
      COUNT(*) as user_count
    FROM users
    GROUP BY DATE_TRUNC('day', created_at)
    ORDER BY signup_date DESC
    LIMIT 30
  `
}

// Top N queries
"Who are the top 10 customers by revenue?"
{
  "sql": `
    SELECT 
      u.name,
      u.email,
      SUM(o.total_amount) as total_revenue
    FROM users u
    JOIN orders o ON u.id = o.user_id
    GROUP BY u.id, u.name, u.email
    ORDER BY total_revenue DESC
    LIMIT 10
  `
}
```

## Write Operations (Advanced)

Enable write operations with caution:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://user:password@localhost:5432/myapp",
        "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
      }
    }
  }
}
```

**Example write operations:**

```typescript
// Batch insert
"Add test users for QA"
{
  "sql": `
    INSERT INTO users (name, email, role)
    VALUES 
      ('Test User 1', 'test1@example.com', 'user'),
      ('Test User 2', 'test2@example.com', 'user')
    RETURNING *
  `
}

// Conditional update
"Mark orders as shipped if they're paid and ready"
{
  "sql": `
    UPDATE orders
    SET status = 'shipped', shipped_at = NOW()
    WHERE status = 'paid' 
      AND ready_for_shipping = true
    RETURNING id, order_number, status
  `
}

// Cleanup old data
"Delete expired sessions older than 30 days"
{
  "sql": `
    DELETE FROM sessions
    WHERE expires_at < NOW() - INTERVAL '30 days'
  `
}
```

## Development Setup

### Local Development

```bash
# Clone repository
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server

# Install dependencies
bun install

# Run with stdio transport
bun run index.ts -- --transport=stdio

# Run with HTTP transport
bun run index.ts -- --transport=http

# Enable debug logging
DEBUG=true bun run index.ts -- --transport=stdio
```

### Testing with Docker

Start a PostgreSQL instance with sample data:

```bash
# Start database with sample tables
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop database
bun run db:stop
```

Sample tables included:
- `users` - User accounts with profiles
- `products` - Product catalog
- `orders` - Order records
- `order_items` - Line items for orders

### Using Local Build

```bash
# Build JavaScript bundle
bun run build:js

# Test the build
bun test
```

Update MCP config to use local build:

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

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or timeout errors

**Solution:**
- Verify `DATABASE_URL` is correct
- Check PostgreSQL is running: `pg_isready -h localhost -p 5432`
- Test connection manually: `psql $DATABASE_URL`
- Check firewall rules for remote connections
- Verify SSL settings match server requirements

### SSL/TLS Errors

**Problem:** SSL certificate verification failed

**Solution:**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:pass@host:5432/db?sslmode=require",
    "PG_SSL_ROOT_CERT": "/path/to/ca-bundle.pem"
  }
}
```

For AWS RDS:
```bash
# Download RDS CA bundle
curl -o rds-ca-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

# Reference in config
"PG_SSL_ROOT_CERT": "/path/to/rds-ca-bundle.pem"
```

### Permission Errors

**Problem:** "permission denied for table" errors

**Solution:**
- Ensure database user has SELECT permissions: `GRANT SELECT ON ALL TABLES IN SCHEMA public TO username;`
- For write ops: `GRANT INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO username;`
- Check schema permissions: `GRANT USAGE ON SCHEMA public TO username;`

### Query Performance

**Problem:** Slow query responses

**Solution:**
- Add LIMIT clauses to large result sets
- Create indexes on frequently queried columns
- Use EXPLAIN to analyze query plans
- Consider connection pooling for high-traffic scenarios

### Debug Mode

Enable detailed logging:

```bash
DEBUG=true npx pg-mcp-server --transport=stdio
```

Or in config:
```json
{
  "env": {
    "DATABASE_URL": "...",
    "DEBUG": "true"
  }
}
```

## Security Best Practices

1. **Read-only by default** - Only enable writes when absolutely necessary
2. **Use dedicated user** - Create a PostgreSQL user with minimal required permissions
3. **Network isolation** - Restrict database access to trusted networks
4. **Environment variables** - Never hardcode credentials in config files
5. **SSL/TLS** - Always use encrypted connections for production databases
6. **Audit logging** - Enable PostgreSQL query logging for write operations

Example restricted user:

```sql
-- Create read-only user
CREATE USER mcp_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE myapp TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_readonly;
```
