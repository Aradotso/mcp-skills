---
name: postgres-mcp-server
description: Query and analyze PostgreSQL databases through Model Context Protocol with controlled LLM access
triggers:
  - query the postgres database
  - analyze database schema
  - get table structure from postgres
  - execute sql query
  - list database tables
  - inspect postgres table
  - run database analysis
  - explore postgresql data
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

The Postgres MCP Server is a Model Context Protocol server that enables LLMs to query and analyze PostgreSQL databases through a controlled interface. It provides safe read-only access by default, with optional write operations, and exposes database schemas, tables, and query execution capabilities.

## Installation

### Quick Install (Cursor/Claude Desktop)

Add to your MCP client settings file:

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

### Configuration File Locations

- **Claude Desktop (macOS)**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Claude Desktop (Windows)**: `%APPDATA%\Claude\claude_desktop_config.json`
- **Cursor**: `.cursor/mcp.json` or Cursor settings

### Local Development Setup

```bash
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install
bun run build:js
```

Use local build in settings:

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

## Environment Variables

- **`DATABASE_URL`** (required): PostgreSQL connection string
  - Format: `postgresql://[user[:password]@][host][:port][/dbname][?param=value]`
  - Example: `postgresql://postgres:mypass@localhost:5432/production`
  
- **`DANGEROUSLY_ALLOW_WRITE_OPS`**: Enable INSERT, UPDATE, DELETE operations (default: `false`)
  - Set to `"true"` to enable writes
  
- **`DEBUG`**: Enable debug logging (default: `false`)
  
- **`PG_SSL_ROOT_CERT`**: Path to TLS CA bundle for SSL connections
  - Example: `/path/to/rds-combined-ca-bundle.pem`

## Transport Modes

### stdio (Default)

Used for local MCP clients like Claude Desktop and Cursor:

```bash
npx --yes pg-mcp-server --transport stdio
```

### HTTP

For HTTP-based MCP clients:

```bash
npx --yes pg-mcp-server --transport http
# Serves at http://localhost:3000/mcp
```

Set custom port:

```bash
PORT=8080 npx --yes pg-mcp-server --transport http
```

## Available Tools

### query

Execute SQL queries against the database.

**Parameters:**
- `sql` (string, required): The SQL query to execute

**Example Usage:**

```typescript
// Through MCP tool call
{
  "name": "query",
  "arguments": {
    "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
  }
}
```

**Read-only queries (always allowed):**

```sql
SELECT id, email, created_at FROM users WHERE role = 'admin';

SELECT p.name, COUNT(o.id) as order_count 
FROM products p 
LEFT JOIN orders o ON p.id = o.product_id 
GROUP BY p.id, p.name 
ORDER BY order_count DESC 
LIMIT 5;

SELECT 
  DATE_TRUNC('month', created_at) as month,
  COUNT(*) as user_count
FROM users
GROUP BY month
ORDER BY month DESC;
```

**Write queries (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):**

```sql
INSERT INTO users (email, name) VALUES ('user@example.com', 'John Doe');

UPDATE products SET price = 29.99 WHERE id = 123;

DELETE FROM sessions WHERE expires_at < NOW();
```

## Resources

Resources provide structured access to database metadata.

### postgres://tables

Lists all tables in the database with schema information.

**Response format:**
```json
{
  "uri": "postgres://tables",
  "mimeType": "application/json",
  "text": "[{\"schema\":\"public\",\"table\":\"users\"},{\"schema\":\"public\",\"table\":\"orders\"}]"
}
```

### postgres://table/{schema}/{table}

Get detailed schema and sample data for a specific table.

**Example URIs:**
- `postgres://table/public/users`
- `postgres://table/analytics/events`

**Response includes:**
- Column definitions (name, type, nullable, default)
- Primary keys and constraints
- Sample rows (up to 5)

```json
{
  "uri": "postgres://table/public/users",
  "mimeType": "application/json",
  "text": "{\"columns\":[{\"name\":\"id\",\"type\":\"integer\",\"nullable\":false}],\"sample\":[{\"id\":1,\"email\":\"user@example.com\"}]}"
}
```

## Common Patterns

### Data Exploration

```typescript
// Prompt: "Show me the database schema"
// Agent will:
// 1. List all tables using postgres://tables resource
// 2. Get schema for each table using postgres://table/{schema}/{table}

// Prompt: "Find all active users created in the last 7 days"
// Agent will use query tool:
{
  "sql": "SELECT id, email, created_at FROM users WHERE active = true AND created_at > NOW() - INTERVAL '7 days' ORDER BY created_at DESC"
}
```

### Aggregation & Analytics

```typescript
// Prompt: "What are the top-selling products this month?"
{
  "sql": "SELECT p.id, p.name, SUM(oi.quantity) as total_sold, SUM(oi.quantity * oi.price) as revenue FROM products p JOIN order_items oi ON p.id = oi.product_id JOIN orders o ON oi.order_id = o.id WHERE o.created_at >= DATE_TRUNC('month', CURRENT_DATE) GROUP BY p.id, p.name ORDER BY total_sold DESC LIMIT 10"
}
```

### Schema Introspection

```typescript
// Prompt: "What's the structure of the orders table?"
// Agent fetches resource: postgres://table/public/orders

// Prompt: "Show me all foreign key relationships"
{
  "sql": "SELECT tc.table_schema, tc.table_name, kcu.column_name, ccu.table_schema AS foreign_table_schema, ccu.table_name AS foreign_table_name, ccu.column_name AS foreign_column_name FROM information_schema.table_constraints AS tc JOIN information_schema.key_column_usage AS kcu ON tc.constraint_name = kcu.constraint_name AND tc.table_schema = kcu.table_schema JOIN information_schema.constraint_column_usage AS ccu ON ccu.constraint_name = tc.constraint_name WHERE tc.constraint_type = 'FOREIGN KEY'"
}
```

### Safe Data Modification (with writes enabled)

```typescript
// When DANGEROUSLY_ALLOW_WRITE_OPS=true

// Prompt: "Mark order #123 as shipped"
{
  "sql": "UPDATE orders SET status = 'shipped', shipped_at = NOW() WHERE id = 123 RETURNING *"
}

// Prompt: "Create a new product category"
{
  "sql": "INSERT INTO categories (name, slug, description) VALUES ('Electronics', 'electronics', 'Electronic devices and accessories') RETURNING *"
}
```

## Configuration Examples

### Production Database (Read-only)

```json
{
  "mcpServers": {
    "postgres-prod": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://readonly_user:${DB_PASSWORD}@prod-db.example.com:5432/production",
        "PG_SSL_ROOT_CERT": "/path/to/ca-bundle.pem"
      }
    }
  }
}
```

### Development Database (With Writes)

```json
{
  "mcpServers": {
    "postgres-dev": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://postgres:devpass@localhost:5432/myapp_dev",
        "DANGEROUSLY_ALLOW_WRITE_OPS": "true",
        "DEBUG": "true"
      }
    }
  }
}
```

### Multiple Database Connections

```json
{
  "mcpServers": {
    "postgres-analytics": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://analyst:${ANALYTICS_DB_PASSWORD}@analytics.example.com:5432/analytics"
      }
    },
    "postgres-app": {
      "command": "npx",
      "args": ["--yes", "pg-mcp-server", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://app_user:${APP_DB_PASSWORD}@app-db.example.com:5432/app_production"
      }
    }
  }
}
```

## Testing with Docker

Quick start with sample database:

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample tables created:
- `users`: User accounts
- `products`: Product catalog
- `orders`: Order records
- `order_items`: Order line items

## Troubleshooting

### Connection Errors

**Problem**: `connection refused` or `timeout`

```bash
# Verify PostgreSQL is running
psql $DATABASE_URL -c "SELECT 1"

# Check connection parameters
echo $DATABASE_URL

# Test with explicit host
DATABASE_URL="postgresql://user:pass@127.0.0.1:5432/dbname"
```

### SSL/TLS Issues

**Problem**: `SSL connection required`

```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:pass@host:5432/db?sslmode=require",
    "PG_SSL_ROOT_CERT": "/path/to/ca-certificate.crt"
  }
}
```

### Write Operations Blocked

**Problem**: `Write operations are disabled`

Enable writes explicitly:

```json
{
  "env": {
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### Debug Mode

Enable verbose logging:

```json
{
  "env": {
    "DEBUG": "true"
  }
}
```

### Permission Errors

**Problem**: `permission denied for table`

Grant appropriate permissions:

```sql
-- Read-only access
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;
GRANT USAGE ON SCHEMA public TO readonly_user;

-- Read-write access
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
```

### Resource Not Found

Ensure schema and table names are correct:

```typescript
// Correct format
"postgres://table/public/users"

// Not
"postgres://table/users"  // Missing schema
```

## Security Best Practices

1. **Use read-only credentials** for production databases
2. **Enable writes only when necessary** (`DANGEROUSLY_ALLOW_WRITE_OPS`)
3. **Use SSL/TLS** for remote connections
4. **Limit database user permissions** to minimum required
5. **Store credentials in environment variables**, never in config files
6. **Use separate database users** for different environments

## Example Prompts for AI Agents

- "Show me the first 5 users from the database"
- "What tables are in this database?"
- "Analyze the orders table structure"
- "Find customers who haven't placed an order in 30 days"
- "Calculate total revenue by product category"
- "Show me the relationships between tables"
- "What's the average order value this quarter?"
