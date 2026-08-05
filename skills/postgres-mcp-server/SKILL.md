---
name: postgres-mcp-server
description: MCP server that enables LLMs to query and analyze PostgreSQL databases through a controlled interface with read/write operations.
triggers:
  - query my postgres database
  - analyze database tables
  - execute SQL query through MCP
  - connect to postgresql database
  - inspect database schema
  - run database queries with MCP
  - set up postgres MCP server
  - get table structure from database
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that provides LLMs with controlled access to PostgreSQL databases. Supports both read-only and write operations, schema inspection, and SQL query execution through stdio or HTTP transport.

## What It Does

- **SQL Query Execution**: Run SELECT, INSERT, UPDATE, DELETE queries through the `query` tool
- **Schema Inspection**: List all tables and get detailed table schemas with sample data
- **Transport Flexibility**: Supports both stdio (default) and HTTP transports
- **Safety Controls**: Write operations disabled by default, must be explicitly enabled
- **TLS Support**: Optional SSL/TLS certificate configuration for secure connections

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

### Local Development Setup

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

# Build for production
bun run build:js
```

### Using Local Build

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
| `DATABASE_URL` | Yes | - | PostgreSQL connection string (format: `postgresql://user:password@host:port/database`) |
| `DANGEROUSLY_ALLOW_WRITE_OPS` | No | `false` | Enable INSERT/UPDATE/DELETE operations |
| `DEBUG` | No | `false` | Enable debug logging |
| `PG_SSL_ROOT_CERT` | No | - | Path to TLS CA bundle (e.g., for AWS RDS) |
| `PORT` | No | `3000` | Port for HTTP transport mode |

### Example Configurations

**Read-only (default):**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://readonly:password@prod-db.example.com:5432/analytics"
  }
}
```

**With write operations enabled:**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://admin:password@localhost:5432/dev_db",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

**With SSL/TLS for AWS RDS:**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://user:password@db.us-east-1.rds.amazonaws.com:5432/production",
    "PG_SSL_ROOT_CERT": "/path/to/global-bundle.pem"
  }
}
```

**Debug mode:**
```json
{
  "env": {
    "DATABASE_URL": "postgresql://postgres:postgres@localhost:5432/test",
    "DEBUG": "true"
  }
}
```

## Transport Modes

### stdio Transport (Default)

Used for local MCP clients like Claude Desktop and Cursor:

```bash
pg-mcp-server --transport=stdio
```

### HTTP Transport

Serves MCP over HTTP at `/mcp` endpoint:

```bash
pg-mcp-server --transport=http
# Server runs on http://localhost:3000/mcp
```

With custom port:
```bash
PORT=8080 pg-mcp-server --transport=http
```

## Tools

### `query` - Execute SQL Queries

Execute any SQL statement (read-only by default).

**Input Schema:**
```typescript
{
  sql: string;  // SQL query to execute
}
```

**Example - SELECT Query:**
```json
{
  "sql": "SELECT id, name, email FROM users WHERE active = true LIMIT 10"
}
```

**Example - JOIN Query:**
```json
{
  "sql": "SELECT o.id, u.name, o.total FROM orders o JOIN users u ON o.user_id = u.id WHERE o.status = 'completed' ORDER BY o.created_at DESC LIMIT 20"
}
```

**Example - Aggregation:**
```json
{
  "sql": "SELECT category, COUNT(*) as count, AVG(price) as avg_price FROM products GROUP BY category ORDER BY count DESC"
}
```

**Example - INSERT (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):**
```json
{
  "sql": "INSERT INTO users (name, email, active) VALUES ('John Doe', 'john@example.com', true) RETURNING id"
}
```

**Example - UPDATE (requires `DANGEROUSLY_ALLOW_WRITE_OPS=true`):**
```json
{
  "sql": "UPDATE products SET price = price * 0.9 WHERE category = 'electronics' AND stock > 10"
}
```

## Resources

### `postgres://tables` - List All Tables

Returns a list of all tables in the database with their schemas.

**Access:**
```
Resource URI: postgres://tables
```

**Response includes:**
- Table names
- Schema names
- Column definitions
- Primary keys
- Foreign keys

### `postgres://table/{schema}/{table}` - Get Table Details

Get detailed schema and sample data for a specific table.

**Access:**
```
Resource URI: postgres://table/public/users
```

**Response includes:**
- Complete column definitions (name, type, nullable, default)
- Primary key constraints
- Foreign key relationships
- Up to 5 sample rows

**Example URI patterns:**
```
postgres://table/public/orders
postgres://table/analytics/events
postgres://table/auth/sessions
```

## Common Usage Patterns

### Data Analysis Workflow

1. **List available tables:**
   - Request resource: `postgres://tables`
   
2. **Inspect table schema:**
   - Request resource: `postgres://table/public/users`
   
3. **Query data:**
   - Use `query` tool with SELECT statement

### Example Prompts for AI Agents

```
"Show me the first 5 users from the database"
→ Uses query tool: SELECT * FROM users LIMIT 5

"What tables are available in the database?"
→ Requests postgres://tables resource

"Analyze sales by category for the last 30 days"
→ Uses query tool with aggregation and date filtering

"Show me the schema for the orders table"
→ Requests postgres://table/public/orders resource
```

### Safe Data Exploration

```typescript
// Read-only exploration (default config)
// Agent can safely analyze data without modification risk

// Query 1: Explore table structure
{
  "sql": "SELECT column_name, data_type, is_nullable FROM information_schema.columns WHERE table_name = 'users' ORDER BY ordinal_position"
}

// Query 2: Sample data distribution
{
  "sql": "SELECT status, COUNT(*) as count FROM orders GROUP BY status"
}

// Query 3: Recent activity
{
  "sql": "SELECT DATE(created_at) as date, COUNT(*) as signups FROM users WHERE created_at >= NOW() - INTERVAL '7 days' GROUP BY DATE(created_at) ORDER BY date"
}
```

### Write Operations (Use with Caution)

```typescript
// Only works when DANGEROUSLY_ALLOW_WRITE_OPS=true

// Insert new record
{
  "sql": "INSERT INTO products (name, price, category, stock) VALUES ('New Product', 29.99, 'electronics', 100) RETURNING id, name"
}

// Update existing records
{
  "sql": "UPDATE users SET last_login = NOW() WHERE id = 123"
}

// Transactional operations (use BEGIN/COMMIT)
{
  "sql": "BEGIN; UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 456; INSERT INTO order_items (order_id, product_id, quantity) VALUES (789, 456, 1); COMMIT;"
}
```

## Testing & Development

### Quick Start with Docker

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample database includes: `users`, `products`, `orders`, `order_items` tables.

### Running Tests

```bash
bun test
```

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or timeout errors

**Solutions:**
- Verify `DATABASE_URL` format: `postgresql://user:password@host:port/database`
- Check PostgreSQL server is running: `psql $DATABASE_URL -c "SELECT 1"`
- Verify network access and firewall rules
- For cloud databases (RDS, etc.), check security group settings

### Write Operations Blocked

**Problem:** INSERT/UPDATE/DELETE fails with permission error

**Solution:**
Set `DANGEROUSLY_ALLOW_WRITE_OPS=true` in environment configuration. Only enable for trusted environments.

### SSL/TLS Certificate Errors

**Problem:** "SSL certificate verify failed"

**Solution:**
Download CA bundle and set path:
```bash
# For AWS RDS
wget https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem

# Add to config
"PG_SSL_ROOT_CERT": "/path/to/global-bundle.pem"
```

### Transport Mode Issues

**Problem:** MCP client can't connect

**Solutions:**
- For local clients (Claude Desktop, Cursor): Use `--transport=stdio`
- For HTTP clients: Verify server is running on correct port
- Check MCP client configuration matches transport mode

### Debug Logging

Enable detailed logs:
```json
{
  "env": {
    "DATABASE_URL": "...",
    "DEBUG": "true"
  }
}
```

Or via command line:
```bash
DEBUG=true bun run index.ts -- --transport=stdio
```

## Security Best Practices

1. **Use read-only credentials by default:** Create PostgreSQL user with SELECT-only permissions
2. **Enable writes only when necessary:** Keep `DANGEROUSLY_ALLOW_WRITE_OPS` disabled unless required
3. **Use SSL/TLS for production:** Always configure `PG_SSL_ROOT_CERT` for remote databases
4. **Limit network access:** Use firewall rules and security groups to restrict database access
5. **Audit queries:** Monitor logs when `DEBUG=true` to track executed queries

## Advanced Configuration

### Read-only PostgreSQL User

```sql
-- Create read-only user
CREATE USER mcp_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE your_database TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO mcp_readonly;
```

Use in configuration:
```json
{
  "env": {
    "DATABASE_URL": "postgresql://mcp_readonly:secure_password@localhost:5432/your_database"
  }
}
```
