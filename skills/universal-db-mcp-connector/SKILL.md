---
name: universal-db-mcp-connector
description: Connect AI assistants to 17+ databases (MySQL, PostgreSQL, MongoDB, Oracle, etc.) via MCP protocol using natural language queries
triggers:
  - set up database connection with MCP
  - query database with natural language
  - configure universal db mcp for Claude
  - connect AI to MySQL/PostgreSQL database
  - use MCP protocol for database access
  - integrate database with Cursor/Claude Desktop
  - setup HTTP API mode for database queries
  - configure multi-database MCP server
---

# Universal DB MCP Connector

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

Universal DB MCP is a connector implementing the Model Context Protocol (MCP) that enables AI assistants to query and analyze databases using natural language. It supports 17 database types including MySQL, PostgreSQL, Oracle, MongoDB, Redis, and Chinese databases like Dameng, KingbaseES, and GaussDB. Works with 50+ platforms including Claude Desktop, Cursor, Windsurf, VS Code, ChatGPT, and Dify.

## Architecture Overview

The connector operates in two modes:

1. **stdio mode** - Direct MCP protocol via stdio transport (for Claude Desktop, Cursor)
2. **http mode** - HTTP server exposing MCP via SSE/Streamable HTTP + REST API (for Dify, remote access)

Both modes provide the same MCP tools for database operations.

## Installation

### Global Installation

```bash
npm install -g universal-db-mcp
```

### Project-Local Installation

```bash
npm install universal-db-mcp
```

### From Source

```bash
git clone https://github.com/Anarkh-Lee/universal-db-mcp.git
cd universal-db-mcp
npm install
npm run build
```

## Supported Databases

| Database | Type Value | Default Port |
|----------|-----------|--------------|
| MySQL | `mysql` | 3306 |
| PostgreSQL | `postgres` | 5432 |
| MongoDB | `mongodb` | 27017 |
| Redis | `redis` | 6379 |
| Oracle | `oracle` | 1521 |
| SQL Server | `sqlserver` | 1433 |
| SQLite | `sqlite` | N/A |
| Dameng | `dm` | 5236 |
| KingbaseES | `kingbase` | 54321 |
| GaussDB | `gaussdb` | 5432 |
| OceanBase | `oceanbase` | 2881 |
| TiDB | `tidb` | 4000 |
| ClickHouse | `clickhouse` | 8123 |
| PolarDB | `polardb` | 3306 |
| Vastbase | `vastbase` | 5432 |
| HighGo | `highgo` | 5866 |
| GoldenDB | `goldendb` | 3306 |

## Configuration for Claude Desktop

### macOS

Edit `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "production-mysql": {
      "command": "npx",
      "args": [
        "universal-db-mcp",
        "--type", "mysql",
        "--host", "localhost",
        "--port", "3306",
        "--user", "root",
        "--password", "${DB_PASSWORD}",
        "--database", "myapp",
        "--readonly"
      ]
    },
    "analytics-postgres": {
      "command": "npx",
      "args": [
        "universal-db-mcp",
        "--type", "postgres",
        "--host", "analytics.example.com",
        "--port", "5432",
        "--user", "analyst",
        "--password", "${ANALYTICS_DB_PASSWORD}",
        "--database", "analytics",
        "--schema", "public",
        "--cache-ttl", "600"
      ]
    },
    "local-sqlite": {
      "command": "npx",
      "args": [
        "universal-db-mcp",
        "--type", "sqlite",
        "--database", "/Users/me/data/app.db"
      ]
    }
  }
}
```

### Windows

Edit `%APPDATA%\Claude\claude_desktop_config.json` with the same structure.

### Common CLI Arguments

```bash
--type <database-type>          # Required: mysql, postgres, mongodb, etc.
--host <hostname>               # Database host (default: localhost)
--port <port>                   # Database port (uses default for type)
--user <username>               # Database user
--password <password>           # Database password (use env vars!)
--database <db-name>            # Database/schema name
--schema <schema-name>          # Schema name (PostgreSQL, SQL Server, Oracle)
--readonly                      # Enable read-only mode (recommended)
--cache-ttl <seconds>           # Schema cache TTL (default: 300)
--mask-sensitive-data           # Enable automatic data masking
--connection-timeout <ms>       # Connection timeout (default: 10000)
--pool-size <number>            # Connection pool size (default: 10)
```

## Configuration for Cursor / VS Code

Edit `.cursor/config.json` or `.vscode/settings.json`:

```json
{
  "mcp.servers": {
    "database": {
      "command": "npx",
      "args": [
        "universal-db-mcp",
        "--type", "mysql",
        "--host", "localhost",
        "--user", "root",
        "--password", "${DB_PASSWORD}",
        "--database", "myapp"
      ]
    }
  }
}
```

## HTTP API Mode

### Starting the Server

```bash
# Using environment variables
export MODE=http
export HTTP_PORT=3000
export API_KEYS=secret-key-1,secret-key-2
export ENABLE_CORS=true

npx universal-db-mcp
```

Or with a config file:

```typescript
// config.ts
export default {
  mode: 'http',
  http: {
    port: 3000,
    apiKeys: ['secret-key-1', 'secret-key-2'],
    cors: true,
    rateLimit: {
      windowMs: 60000,
      maxRequests: 100
    }
  }
};
```

```bash
npx universal-db-mcp --config config.ts
```

### MCP over SSE (Legacy)

Connect via Server-Sent Events:

```bash
# Establish SSE connection
curl -N "http://localhost:3000/sse?type=mysql&host=localhost&port=3306&user=root&password=mypass&database=mydb" \
  -H "Authorization: Bearer secret-key-1"

# Send MCP message
curl -X POST http://localhost:3000/sse/message \
  -H "Authorization: Bearer secret-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "params": {
      "name": "execute_query",
      "arguments": {
        "query": "SELECT * FROM users LIMIT 10"
      }
    },
    "id": 1
  }'
```

### MCP over Streamable HTTP (Recommended)

```bash
# Execute query via MCP
curl -X POST http://localhost:3000/mcp \
  -H "Authorization: Bearer secret-key-1" \
  -H "Content-Type: application/json" \
  -H "X-DB-Type: mysql" \
  -H "X-DB-Host: localhost" \
  -H "X-DB-Port: 3306" \
  -H "X-DB-User: root" \
  -H "X-DB-Password: mypass" \
  -H "X-DB-Database: mydb" \
  -d '{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "params": {
      "name": "execute_query",
      "arguments": {
        "query": "SELECT COUNT(*) as total FROM orders WHERE created_at >= DATE_SUB(NOW(), INTERVAL 7 DAY)"
      }
    },
    "id": 1
  }'
```

### REST API Endpoints

```bash
# Health check
curl http://localhost:3000/api/health

# Connect to database
curl -X POST http://localhost:3000/api/connect \
  -H "Authorization: Bearer secret-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "mysql",
    "host": "localhost",
    "port": 3306,
    "user": "root",
    "password": "mypass",
    "database": "mydb"
  }'

# Execute query
curl -X POST http://localhost:3000/api/query \
  -H "Authorization: Bearer secret-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "SELECT * FROM products WHERE price > 100 ORDER BY price DESC LIMIT 10"
  }'

# Get schema
curl -X GET http://localhost:3000/api/schema \
  -H "Authorization: Bearer secret-key-1"

# Get table info
curl -X POST http://localhost:3000/api/table \
  -H "Authorization: Bearer secret-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "tableName": "users"
  }'

# Get sample data
curl -X POST http://localhost:3000/api/sample \
  -H "Authorization: Bearer secret-key-1" \
  -H "Content-Type: application/json" \
  -d '{
    "tableName": "orders",
    "limit": 5
  }'

# Clear cache
curl -X POST http://localhost:3000/api/cache/clear \
  -H "Authorization: Bearer secret-key-1"

# Get connection status
curl -X GET http://localhost:3000/api/status \
  -H "Authorization: Bearer secret-key-1"

# Disconnect
curl -X POST http://localhost:3000/api/disconnect \
  -H "Authorization: Bearer secret-key-1"
```

## MCP Tools Available

### execute_query

Execute SQL queries (SELECT only in readonly mode).

```typescript
// Request
{
  "name": "execute_query",
  "arguments": {
    "query": "SELECT u.name, COUNT(o.id) as order_count FROM users u LEFT JOIN orders o ON u.id = o.user_id GROUP BY u.id HAVING order_count > 5"
  }
}
```

### get_schema

Get complete database schema with table relationships.

```typescript
// Request
{
  "name": "get_schema",
  "arguments": {}
}

// Response includes:
// - All tables with columns, types, constraints
// - Primary keys, foreign keys
// - Indexes, table comments
// - Inferred relationships
```

### get_table_info

Get detailed information about a specific table.

```typescript
{
  "name": "get_table_info",
  "arguments": {
    "tableName": "orders"
  }
}
```

### get_sample_data

Get sample rows from a table.

```typescript
{
  "name": "get_sample_data",
  "arguments": {
    "tableName": "products",
    "limit": 10
  }
}
```

### get_enum_values

Get all possible values for ENUM columns (MySQL).

```typescript
{
  "name": "get_enum_values",
  "arguments": {
    "tableName": "users",
    "columnName": "status"
  }
}
```

### clear_cache

Clear the schema cache to force refresh.

```typescript
{
  "name": "clear_cache",
  "arguments": {}
}
```

### connect_database

Dynamically connect to a different database.

```typescript
{
  "name": "connect_database",
  "arguments": {
    "type": "postgres",
    "host": "analytics.example.com",
    "port": 5432,
    "user": "analyst",
    "password": "secure-password",
    "database": "analytics"
  }
}
```

### disconnect_database

Disconnect from current database.

```typescript
{
  "name": "disconnect_database",
  "arguments": {}
}
```

### get_connection_status

Check current connection status.

```typescript
{
  "name": "get_connection_status",
  "arguments": {}
}
```

## Common Usage Patterns

### Natural Language Queries in Claude

Once configured, ask Claude questions like:

- "Show me the structure of the users table"
- "How many orders were placed in the last 7 days?"
- "What are the top 10 products by revenue this month?"
- "Find all users who haven't placed an order in 90 days"
- "Show me the relationship between users and orders tables"

Claude will automatically:
1. Use `get_schema` to understand database structure
2. Generate appropriate SQL via `execute_query`
3. Format and explain results

### Programmatic Access (TypeScript)

```typescript
import { MCPClient } from 'universal-db-mcp/client';

const client = new MCPClient({
  serverUrl: 'http://localhost:3000/mcp',
  apiKey: process.env.API_KEY
});

// Connect to database
await client.callTool('connect_database', {
  type: 'mysql',
  host: 'localhost',
  user: 'root',
  password: process.env.DB_PASSWORD,
  database: 'myapp'
});

// Get schema
const schema = await client.callTool('get_schema', {});

// Execute query
const results = await client.callTool('execute_query', {
  query: 'SELECT * FROM users WHERE created_at > DATE_SUB(NOW(), INTERVAL 30 DAY)'
});

console.log(results);
```

### Multi-Database Setup

Configure multiple databases in Claude Desktop:

```json
{
  "mcpServers": {
    "production-db": {
      "command": "npx",
      "args": [
        "universal-db-mcp",
        "--type", "mysql",
        "--host", "prod.example.com",
        "--user", "readonly",
        "--password", "${PROD_DB_PASSWORD}",
        "--database", "production",
        "--readonly"
      ]
    },
    "analytics-db": {
      "command": "npx",
      "args": [
        "universal-db-mcp",
        "--type", "postgres",
        "--host", "analytics.example.com",
        "--user", "analyst",
        "--password", "${ANALYTICS_PASSWORD}",
        "--database", "analytics"
      ]
    },
    "cache-db": {
      "command": "npx",
      "args": [
        "universal-db-mcp",
        "--type", "redis",
        "--host", "localhost",
        "--port", "6379",
        "--password", "${REDIS_PASSWORD}"
      ]
    }
  }
}
```

### Dify Integration

In Dify, configure MCP tool:

1. Go to Tools → Add Tool → MCP
2. Set endpoint: `http://localhost:3000/mcp`
3. Add headers:
   ```
   Authorization: Bearer your-api-key
   X-DB-Type: mysql
   X-DB-Host: localhost
   X-DB-Port: 3306
   X-DB-User: root
   X-DB-Password: your-password
   X-DB-Database: your-database
   ```

### Data Masking Example

```bash
# Enable automatic sensitive data masking
npx universal-db-mcp \
  --type mysql \
  --host localhost \
  --user root \
  --password "${DB_PASSWORD}" \
  --database myapp \
  --mask-sensitive-data
```

Automatically masks:
- Phone numbers → `138****5678`
- Emails → `user***@example.com`
- ID cards → `110***********1234`
- Bank cards → `6222****5678`

### Performance Optimization

```typescript
// Configure caching and connection pooling
const args = [
  'universal-db-mcp',
  '--type', 'postgres',
  '--host', 'db.example.com',
  '--user', 'app',
  '--password', process.env.DB_PASSWORD,
  '--database', 'production',
  '--cache-ttl', '600',           // Cache schema for 10 minutes
  '--pool-size', '20',            // 20 connections in pool
  '--connection-timeout', '5000'  // 5 second timeout
];
```

## Troubleshooting

### Connection Issues

**Problem**: "Connection timeout" or "Cannot connect to database"

**Solution**:
```bash
# Test connection manually
npx universal-db-mcp \
  --type mysql \
  --host localhost \
  --user root \
  --password "test" \
  --database test \
  --connection-timeout 15000

# Check firewall/network access
telnet db.example.com 3306

# Verify credentials
mysql -h localhost -u root -p
```

### Schema Not Loading

**Problem**: Schema appears empty or incomplete

**Solution**:
```bash
# Clear cache and reload
curl -X POST http://localhost:3000/api/cache/clear \
  -H "Authorization: Bearer ${API_KEY}"

# Increase cache TTL
--cache-ttl 0  # Disable caching for debugging

# Check permissions
SHOW GRANTS FOR 'username'@'host';
```

### Read-Only Mode Issues

**Problem**: "Query not allowed in read-only mode"

**Solution**:
```bash
# Remove --readonly flag for write access (use with caution!)
npx universal-db-mcp \
  --type mysql \
  --host localhost \
  --user admin \
  --password "${DB_PASSWORD}" \
  --database myapp
  # No --readonly flag
```

### SSL/TLS Connection

**Problem**: "SSL connection required"

**Solution**:
```bash
# For MySQL
npx universal-db-mcp \
  --type mysql \
  --host secure.example.com \
  --user root \
  --password "${DB_PASSWORD}" \
  --database myapp \
  --ssl-ca /path/to/ca.pem \
  --ssl-cert /path/to/client-cert.pem \
  --ssl-key /path/to/client-key.pem

# For PostgreSQL
npx universal-db-mcp \
  --type postgres \
  --host secure.example.com \
  --user postgres \
  --password "${DB_PASSWORD}" \
  --database myapp \
  --ssl-mode require
```

### Large Result Sets

**Problem**: Queries timing out or returning too much data

**Solution**:
```sql
-- Always use LIMIT in queries
SELECT * FROM large_table LIMIT 100;

-- Use pagination
SELECT * FROM orders 
ORDER BY created_at DESC 
LIMIT 100 OFFSET 0;

-- Aggregate instead of returning all rows
SELECT COUNT(*), AVG(price), MAX(price) 
FROM products 
WHERE category = 'electronics';
```

### Claude Desktop Not Detecting Server

**Problem**: MCP server doesn't appear in Claude Desktop

**Solution**:
1. Verify JSON syntax in config file
2. Restart Claude Desktop completely
3. Check logs:
   ```bash
   # macOS
   tail -f ~/Library/Logs/Claude/mcp*.log
   
   # Windows
   type %APPDATA%\Claude\logs\mcp*.log
   ```
4. Test command manually:
   ```bash
   npx universal-db-mcp --type mysql --host localhost --user root --password test --database test
   ```

### Environment Variables Not Working

**Problem**: Password/credentials not being read from environment

**Solution**:
```bash
# Don't use ${} in config - use actual env var expansion
# Wrong:
"--password", "${DB_PASSWORD}"

# Right - set env var before starting Claude:
export DB_PASSWORD="mypassword"
# Then in config:
"--password", "mypassword"

# Or use dotenv approach - create .env file:
echo "DB_PASSWORD=mypassword" > ~/.claude/.env
```

## Security Best Practices

1. **Always use readonly mode in production**: `--readonly`
2. **Use environment variables for credentials**: Never hardcode passwords
3. **Enable API keys in HTTP mode**: `export API_KEYS=strong-secret-key`
4. **Restrict network access**: Firewall rules, VPN, SSH tunnels
5. **Enable data masking**: `--mask-sensitive-data`
6. **Use least-privilege database users**: Grant only SELECT permissions
7. **Monitor query logs**: Track what queries are being executed
8. **Set connection limits**: `--pool-size` to prevent resource exhaustion

## Advanced Configuration

### SSH Tunnel for Remote Databases

```bash
# Setup SSH tunnel
ssh -L 3307:localhost:3306 user@remote-server.com

# Connect via tunnel
npx universal-db-mcp \
  --type mysql \
  --host localhost \
  --port 3307 \
  --user root \
  --password "${DB_PASSWORD}" \
  --database production
```

### Docker Deployment

```dockerfile
FROM node:20-alpine

WORKDIR /app

RUN npm install -g universal-db-mcp

ENV MODE=http
ENV HTTP_PORT=3000
ENV API_KEYS=changeme

EXPOSE 3000

CMD ["npx", "universal-db-mcp"]
```

```bash
docker build -t universal-db-mcp .
docker run -d \
  -p 3000:3000 \
  -e API_KEYS="${API_KEY}" \
  -e MODE=http \
  universal-db-mcp
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: universal-db-mcp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: universal-db-mcp
  template:
    metadata:
      labels:
        app: universal-db-mcp
    spec:
      containers:
      - name: mcp
        image: universal-db-mcp:latest
        ports:
        - containerPort: 3000
        env:
        - name: MODE
          value: "http"
        - name: HTTP_PORT
          value: "3000"
        - name: API_KEYS
          valueFrom:
            secretKeyRef:
              name: mcp-secrets
              key: api-keys
---
apiVersion: v1
kind: Service
metadata:
  name: universal-db-mcp
spec:
  selector:
    app: universal-db-mcp
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

This skill enables AI coding agents to help developers integrate universal-db-mcp with their databases and AI tools, supporting natural language database queries across 17+ database types and 50+ platforms.
