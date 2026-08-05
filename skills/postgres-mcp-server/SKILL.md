---
name: postgres-mcp-server
description: MCP server for PostgreSQL databases enabling LLMs to query and analyze database schemas and data through a controlled interface
triggers:
  - "connect to a postgres database"
  - "query my postgresql database"
  - "analyze database schema"
  - "set up postgres mcp server"
  - "get table information from postgres"
  - "run sql queries through mcp"
  - "configure database mcp tools"
  - "explore postgres tables and data"
---

# Postgres MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

A Model Context Protocol (MCP) server that provides LLMs with controlled access to PostgreSQL databases. Enables querying, schema inspection, and data analysis through MCP tools and resources.

## What It Does

- **Query Execution**: Run SQL queries against PostgreSQL databases
- **Schema Inspection**: Browse tables, view structures, and sample data
- **Read-Only by Default**: Protects data with optional write operations flag
- **Dual Transport**: Supports both stdio and HTTP transports
- **SSL Support**: Optional TLS CA bundle for secure connections (e.g., AWS RDS)

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

### Global Install

```bash
npm install -g pg-mcp-server
```

Then configure:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "pg-mcp-server",
      "args": ["--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://user:password@host:5432/dbname"
      }
    }
  }
}
```

### Local Development Build

```bash
git clone https://github.com/ericzakariasson/pg-mcp-server.git
cd pg-mcp-server
bun install
bun run build:js
```

Configure with absolute path:

```json
{
  "mcpServers": {
    "postgres": {
      "command": "node",
      "args": ["/absolute/path/to/pg-mcp-server/lib/index.js", "--transport", "stdio"],
      "env": {
        "DATABASE_URL": "postgresql://postgres:postgres@localhost:5432/mydb"
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
| `DANGEROUSLY_ALLOW_WRITE_OPS` | No | `false` | Enable INSERT, UPDATE, DELETE operations |
| `DEBUG` | No | `false` | Enable debug logging |
| `PG_SSL_ROOT_CERT` | No | - | Path to TLS CA bundle (e.g., AWS RDS) |

### Connection String Format

```
postgresql://[user[:password]@][host][:port][/dbname][?param1=value1&...]
```

Examples:

```bash
# Local database
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/myapp"

# Remote with SSL
DATABASE_URL="postgresql://user:pass@db.example.com:5432/prod?sslmode=require"

# AWS RDS with CA bundle
DATABASE_URL="postgresql://admin:pass@mydb.abc123.us-east-1.rds.amazonaws.com:5432/production"
PG_SSL_ROOT_CERT="/path/to/global-bundle.pem"
```

### Transport Modes

#### stdio (Default)

For MCP clients that communicate via standard input/output:

```bash
pg-mcp-server --transport=stdio
```

#### HTTP

For clients supporting MCP Streamable HTTP:

```bash
PORT=3000 pg-mcp-server --transport=http
# Endpoint: http://localhost:3000/mcp
```

## MCP Tools

### query

Execute SQL queries against the database.

**Parameters:**
- `sql` (string, required): SQL query to execute

**Examples:**

```typescript
// Read-only query (always allowed)
{
  "sql": "SELECT * FROM users WHERE active = true LIMIT 10"
}

// Join multiple tables
{
  "sql": "SELECT o.id, o.total, u.email FROM orders o JOIN users u ON o.user_id = u.id WHERE o.status = 'pending'"
}

// Aggregate data
{
  "sql": "SELECT DATE(created_at) as date, COUNT(*) as orders FROM orders GROUP BY DATE(created_at) ORDER BY date DESC LIMIT 30"
}

// Write query (requires DANGEROUSLY_ALLOW_WRITE_OPS=true)
{
  "sql": "UPDATE users SET last_login = NOW() WHERE id = 123"
}
```

## MCP Resources

### postgres://tables

List all tables in the database.

**Returns:** Array of table objects with schema and name information.

**Example Response:**
```json
{
  "contents": [
    {
      "uri": "postgres://tables",
      "mimeType": "application/json",
      "text": "[{\"schema\":\"public\",\"name\":\"users\"},{\"schema\":\"public\",\"name\":\"orders\"}]"
    }
  ]
}
```

### postgres://table/{schema}/{table}

Get detailed schema information and sample data for a specific table.

**Parameters:**
- `schema`: Database schema (usually "public")
- `table`: Table name

**Returns:** Table structure, column types, constraints, and 5 sample rows.

**Example:**
```
postgres://table/public/users
```

**Response includes:**
- Column names and data types
- Primary keys and constraints
- Sample rows (up to 5)
- Row count

## Common Usage Patterns

### Schema Exploration

```typescript
// Prompt: "What tables are in my database?"
// Agent uses: postgres://tables resource

// Prompt: "Show me the structure of the users table"
// Agent uses: postgres://table/public/users resource

// Prompt: "What columns does the orders table have?"
// Agent uses: postgres://table/public/orders resource
```

### Data Analysis

```typescript
// Prompt: "How many active users do I have?"
// Agent uses query tool:
{
  "sql": "SELECT COUNT(*) FROM users WHERE active = true"
}

// Prompt: "Show me revenue by month for 2024"
// Agent uses query tool:
{
  "sql": "SELECT DATE_TRUNC('month', created_at) as month, SUM(total) as revenue FROM orders WHERE created_at >= '2024-01-01' GROUP BY month ORDER BY month"
}

// Prompt: "Find the top 10 customers by order value"
// Agent uses query tool:
{
  "sql": "SELECT u.email, SUM(o.total) as total_spent FROM users u JOIN orders o ON u.id = o.user_id GROUP BY u.id, u.email ORDER BY total_spent DESC LIMIT 10"
}
```

### Safe Data Modification

Enable write operations:

```json
{
  "env": {
    "DATABASE_URL": "postgresql://localhost/mydb",
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

```typescript
// Prompt: "Mark order 456 as shipped"
// Agent uses query tool:
{
  "sql": "UPDATE orders SET status = 'shipped', shipped_at = NOW() WHERE id = 456"
}

// Prompt: "Delete inactive users from last year"
// Agent uses query tool:
{
  "sql": "DELETE FROM users WHERE active = false AND last_login < '2024-01-01'"
}
```

## Development & Testing

### Local Setup with Docker

```bash
# Start PostgreSQL with sample data
bun run db:start

# Test with MCP Inspector
bun run inspector

# Stop PostgreSQL
bun run db:stop
```

Sample database includes: `users`, `products`, `orders`, `order_items`

### Running Tests

```bash
bun install
bun test
```

### Debug Mode

Enable detailed logging:

```bash
DEBUG=true pg-mcp-server --transport=stdio
```

Or via environment:

```json
{
  "env": {
    "DATABASE_URL": "postgresql://localhost/mydb",
    "DEBUG": "true"
  }
}
```

## Troubleshooting

### Connection Issues

**Problem:** "Connection refused" or timeout errors

**Solutions:**
- Verify PostgreSQL is running: `pg_isready -h localhost -p 5432`
- Check firewall rules allow port 5432
- Confirm credentials in `DATABASE_URL`
- Test connection: `psql $DATABASE_URL`

### SSL/TLS Errors

**Problem:** "SSL connection required" or certificate validation failures

**Solutions:**
```bash
# Add SSL mode to connection string
DATABASE_URL="postgresql://host/db?sslmode=require"

# For AWS RDS, download and use CA bundle
curl -o rds-ca-bundle.pem https://truststore.pki.rds.amazonaws.com/global/global-bundle.pem
PG_SSL_ROOT_CERT="/path/to/rds-ca-bundle.pem"
```

### Write Operations Blocked

**Problem:** "Write operations are disabled"

**Solution:** Enable write operations explicitly:
```json
{
  "env": {
    "DANGEROUSLY_ALLOW_WRITE_OPS": "true"
  }
}
```

### Query Performance

**Problem:** Slow queries or timeouts

**Solutions:**
- Add `LIMIT` clauses to large queries
- Create indexes on frequently queried columns
- Use `EXPLAIN` to analyze query plans:
  ```sql
  EXPLAIN ANALYZE SELECT * FROM large_table WHERE column = 'value'
  ```

### MCP Client Not Finding Server

**Problem:** Server not appearing in MCP client

**Solutions:**
- Restart the MCP client completely
- Verify JSON configuration syntax
- Check logs for startup errors
- Ensure `npx` or `node` is in PATH
- Use absolute paths for local builds

### Resource URI Format

**Problem:** Resource not found errors

**Correct formats:**
```
postgres://tables
postgres://table/public/users
postgres://table/my_schema/my_table
```

**Incorrect:**
```
postgres://table/users  # Missing schema
postgres:/tables        # Missing second slash
```

## Example Prompts

Test your setup with these prompts:

```
"Show me the first 5 users from the database"
"What tables exist in my database?"
"Analyze the schema of the orders table"
"Count how many products we have in each category"
"Show me orders placed in the last 7 days"
"Find users who haven't logged in for 90 days"
"Calculate total revenue by product"
```

## Security Best Practices

1. **Use Read-Only by Default**: Only enable writes when necessary
2. **Limit Database Permissions**: Create a dedicated database user with minimal privileges
3. **Use Environment Variables**: Never hardcode credentials
4. **Connection Pooling**: For production, consider a connection pool
5. **Query Validation**: Review queries before enabling write operations

Example restricted user:

```sql
CREATE USER mcp_readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE myapp TO mcp_readonly;
GRANT USAGE ON SCHEMA public TO mcp_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO mcp_readonly;
```
