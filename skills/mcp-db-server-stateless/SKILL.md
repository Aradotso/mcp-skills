---
name: mcp-db-server-stateless
description: Stateless MCP database server enabling AI agents to query and manage MySQL, PostgreSQL, SQLite, SQL Server, Oracle, and H2 databases without storing credentials
triggers:
  - "connect to my database with MCP"
  - "query database tables using AI"
  - "set up stateless database MCP server"
  - "execute SQL through MCP protocol"
  - "get database schema information"
  - "manage multiple databases with MCP"
  - "deploy MCP database service"
  - "configure Claude Desktop for database access"
---

# mcp-db-server Stateless Database MCP Server

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

mcp-db-server is a stateless MCP (Model Context Protocol) service built with Spring Boot that enables AI agents to interact with databases directly. Unlike traditional database tools, it **does not store any connection credentials**—the AI client passes connection parameters with every request. The server caches connection pools in memory (5-minute TTL) for performance while maintaining zero persistence.

**Supported Databases:**
- MySQL (port 3306)
- PostgreSQL (port 5432)
- H2 (embedded)
- SQLite (file-based)
- SQL Server (port 1433)
- Oracle (port 1521)

## Installation & Deployment

### Quick Start with Maven

```bash
# Clone repository
git clone https://github.com/PerfectXM/mcp-db-server.git
cd mcp-db-server

# Run with Maven wrapper (Windows)
.\mvnw.cmd spring-boot:run

# Run with Maven wrapper (Linux/macOS)
./mvnw spring-boot:run
```

Server starts on `http://localhost:8088` with SSE endpoint at `/sse`.

### Docker Deployment

```bash
# Build image
docker build -t mcp-db-server .

# Run container
docker run -d -p 8088:8088 --name mcp-db-server mcp-db-server

# Or use docker-compose
docker-compose up -d
```

### JAR Deployment

```bash
# Build JAR
./mvnw clean package -DskipTests

# Run JAR
java -jar target/mcp-db-server-1.0.0.jar

# Custom port
java -jar target/mcp-db-server-1.0.0.jar --server.port=9090
```

## Configuration

### Server Configuration (`application.yml`)

```yaml
server:
  port: 8088

spring:
  ai:
    mcp:
      server:
        sse:
          enabled: true
          path: /sse
        transport: sse
```

### AI Client Configuration

**Claude Desktop** (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "mcp-db-server": {
      "type": "sse",
      "url": "http://localhost:8088/sse"
    }
  }
}
```

**Cursor** (settings):
```json
{
  "mcpServers": {
    "mcp-db-server": {
      "transport": "sse",
      "url": "http://localhost:8088/sse"
    }
  }
}
```

## MCP Tools API

All tools require connection parameters on **every invocation**. No credentials are stored server-side.

### Common Connection Parameters

Every tool accepts these parameters:

| Parameter  | Type   | Required | Description                                    |
|------------|--------|----------|------------------------------------------------|
| `type`     | String | Yes      | Database type: mysql, postgresql, h2, sqlite, sqlserver, oracle |
| `host`     | String | Yes      | Database host (use "localhost" for local)      |
| `port`     | int    | Yes      | Port number (0 = use default for type)         |
| `database` | String | Yes      | Database/schema name                           |
| `username` | String | Yes      | Database username                              |
| `password` | String | Yes      | Database password                              |

### Tool: `listTables`

Lists all tables in the database.

**Additional Parameters:**
- `schema` (String, optional): Schema name filter

**Example AI Request:**
```
"Show me all tables in my local MySQL database 'myapp'"

→ Tool call:
  type: "mysql"
  host: "localhost"
  port: 3306
  database: "myapp"
  username: "root"
  password: "${DB_PASSWORD}"
```

**Response Format:**
```json
[
  {"tableName": "users", "tableType": "TABLE"},
  {"tableName": "orders", "tableType": "TABLE"},
  {"tableName": "products", "tableType": "TABLE"}
]
```

### Tool: `describeTable`

Gets column information for a specific table.

**Additional Parameters:**
- `table` (String, required): Table name
- `schema` (String, optional): Schema name

**Example AI Request:**
```
"What columns does the users table have?"

→ Tool call:
  type: "mysql"
  host: "localhost"
  port: 3306
  database: "myapp"
  username: "root"
  password: "${DB_PASSWORD}"
  table: "users"
```

**Response Format:**
```json
[
  {
    "columnName": "id",
    "typeName": "INT",
    "columnSize": 11,
    "nullable": "NO",
    "autoIncrement": true,
    "primaryKey": true
  },
  {
    "columnName": "email",
    "typeName": "VARCHAR",
    "columnSize": 255,
    "nullable": "NO",
    "autoIncrement": false,
    "primaryKey": false
  }
]
```

### Tool: `listIndexes`

Gets index information for a table.

**Additional Parameters:**
- `table` (String, required): Table name
- `schema` (String, optional): Schema name

**Example AI Request:**
```
"Show indexes on the orders table"

→ Tool call:
  type: "postgresql"
  host: "db.example.com"
  port: 5432
  database: "production"
  username: "reader"
  password: "${DB_PASSWORD}"
  table: "orders"
```

### Tool: `executeQuery`

Executes read-only SQL (SELECT, SHOW, DESCRIBE, EXPLAIN).

**Additional Parameters:**
- `sql` (String, required): SQL query
- `limit` (int, optional): Max rows to return (default: 100)

**Example AI Request:**
```
"Get the 5 most recent orders"

→ Tool call:
  type: "mysql"
  host: "localhost"
  port: 3306
  database: "myapp"
  username: "root"
  password: "${DB_PASSWORD}"
  sql: "SELECT * FROM orders ORDER BY created_at DESC"
  limit: 5
```

**Response Format:**
```json
{
  "columns": ["id", "user_id", "total", "created_at"],
  "rows": [
    [1523, 42, 129.99, "2026-06-22 10:30:00"],
    [1522, 38, 79.50, "2026-06-22 09:15:00"]
  ],
  "rowCount": 2,
  "limited": false
}
```

### Tool: `executeUpdate`

Executes write operations (INSERT, UPDATE, DELETE, DDL).

**Additional Parameters:**
- `sql` (String, required): SQL statement

**Example AI Request:**
```
"Create a new table for product reviews"

→ Tool call:
  type: "mysql"
  host: "localhost"
  port: 3306
  database: "myapp"
  username: "admin"
  password: "${DB_PASSWORD}"
  sql: "CREATE TABLE reviews (id INT PRIMARY KEY AUTO_INCREMENT, product_id INT, rating INT, comment TEXT)"
```

**Response Format:**
```json
{
  "affectedRows": 0,
  "message": "DDL executed successfully"
}
```

### Tool: `getDatabaseInfo`

Gets database metadata (version, driver, capabilities).

**Example AI Request:**
```
"What version of PostgreSQL am I connected to?"

→ Tool call:
  type: "postgresql"
  host: "localhost"
  port: 5432
  database: "myapp"
  username: "postgres"
  password: "${DB_PASSWORD}"
```

**Response Format:**
```json
{
  "databaseProductName": "PostgreSQL",
  "databaseProductVersion": "14.2",
  "driverName": "PostgreSQL JDBC Driver",
  "driverVersion": "42.3.3",
  "maxConnections": 100,
  "supportsTransactions": true
}
```

## Code Examples

### Java: Adding Custom Tool (Extending DatabaseTools)

```java
package com.mcp.db.tool;

import org.springframework.ai.mcp.server.McpServer;
import org.springframework.stereotype.Component;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

@Component
public class DatabaseTools {
    
    private final ConnectionFactory connectionFactory;
    
    public DatabaseTools(ConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }
    
    // New custom tool: count rows in a table
    @McpServer.Tool(description = "Count total rows in a table")
    public long countRows(
        String type, String host, int port, String database,
        String username, String password,
        String table
    ) throws Exception {
        
        try (Connection conn = connectionFactory.getConnection(
                type, host, port, database, username, password)) {
            
            String sql = "SELECT COUNT(*) FROM " + table;
            try (PreparedStatement stmt = conn.prepareStatement(sql);
                 ResultSet rs = stmt.executeQuery()) {
                
                if (rs.next()) {
                    return rs.getLong(1);
                }
                return 0;
            }
        }
    }
}
```

### Java: Custom Connection Pool Configuration

```java
package com.mcp.db.config;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
import org.springframework.stereotype.Component;

@Component
public class ConnectionFactory {
    
    private HikariDataSource createConnectionPool(
        String jdbcUrl, String username, String password
    ) {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(jdbcUrl);
        config.setUsername(username);
        config.setPassword(password);
        
        // Custom pool settings
        config.setMaximumPoolSize(10);
        config.setMinimumIdle(2);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(300000); // 5 minutes
        config.setMaxLifetime(600000);  // 10 minutes
        
        // Performance tuning
        config.setConnectionTestQuery("SELECT 1");
        config.setPoolName("MCP-DB-Pool");
        
        return new HikariDataSource(config);
    }
}
```

### Python: Client-Side MCP Interaction

```python
import requests
import json

# SSE endpoint
MCP_URL = "http://localhost:8088/sse"

# Database connection parameters
db_params = {
    "type": "mysql",
    "host": "localhost",
    "port": 3306,
    "database": "myapp",
    "username": "root",
    "password": os.environ["DB_PASSWORD"]
}

# Call listTables tool
def list_tables():
    payload = {
        "jsonrpc": "2.0",
        "method": "tools/call",
        "params": {
            "name": "listTables",
            "arguments": db_params
        },
        "id": 1
    }
    
    response = requests.post(MCP_URL, json=payload)
    return response.json()

# Call executeQuery tool
def run_query(sql, limit=100):
    payload = {
        "jsonrpc": "2.0",
        "method": "tools/call",
        "params": {
            "name": "executeQuery",
            "arguments": {
                **db_params,
                "sql": sql,
                "limit": limit
            }
        },
        "id": 2
    }
    
    response = requests.post(MCP_URL, json=payload)
    return response.json()

# Example usage
tables = list_tables()
print(f"Found tables: {tables}")

results = run_query("SELECT * FROM users WHERE status = 'active'", limit=50)
print(f"Query returned {results['result']['rowCount']} rows")
```

## Common Patterns

### Pattern 1: Database Schema Exploration

```
User: "I need to understand the structure of my e-commerce database"

AI workflow:
1. Call listTables with connection params
2. For each table, call describeTable
3. Call listIndexes to check optimization
4. Present schema overview to user
```

### Pattern 2: Data Analysis Query

```
User: "Show me sales trends from last month"

AI workflow:
1. Call getDatabaseInfo to verify database type
2. Craft appropriate SQL for date functions
3. Call executeQuery with generated SQL
4. Format and present results
```

### Pattern 3: Multi-Database Comparison

```
User: "Compare user counts between dev and prod databases"

AI workflow:
1. Call executeQuery on dev DB: "SELECT COUNT(*) FROM users"
2. Call executeQuery on prod DB: "SELECT COUNT(*) FROM users"
3. Compare results and report differences
```

### Pattern 4: Safe Schema Migration

```
User: "Add an email_verified column to users table"

AI workflow:
1. Call describeTable to check current schema
2. Verify column doesn't exist
3. Call executeUpdate with ALTER TABLE statement
4. Call describeTable again to confirm change
```

## Troubleshooting

### Connection Pool Issues

**Symptom:** "Connection pool exhausted" errors

**Solution:**
```yaml
# Edit application.yml or pass as Java system properties
hikari:
  maximum-pool-size: 20  # Increase max connections
  connection-timeout: 60000  # 60 seconds
```

### Database Driver Not Found

**Symptom:** `ClassNotFoundException: com.mysql.cj.jdbc.Driver`

**Solution:** Add driver dependency to `pom.xml`:
```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version>
</dependency>
```

### SSL/TLS Connection Errors (PostgreSQL)

**Symptom:** "The server does not support SSL"

**Fix:** Modify connection URL in `ConnectionFactory.java`:
```java
if (type.equals("postgresql")) {
    return String.format("jdbc:postgresql://%s:%d/%s?sslmode=disable",
        host, port, database);
}
```

### Memory Leaks from Cached Connections

**Symptom:** Server memory grows over time

**Check:** Connection pool TTL is configured (default 5 minutes):
```java
// In ConnectionFactory.java
private static final long CACHE_TTL_MILLIS = 300_000; // 5 minutes
```

### AI Can't Access Database

**Symptom:** Tools are available but calls fail with authentication errors

**Checklist:**
1. Verify database is accessible from server host: `telnet db_host db_port`
2. Check user has required permissions: `GRANT SELECT ON database.* TO 'user'@'%';`
3. Verify firewall allows connections on database port
4. Test connection manually with JDBC URL

### Oracle-Specific Connection Issues

**Symptom:** "Listener refused the connection"

**Solution:** Use correct Oracle JDBC URL format:
```java
// For SID
jdbc:oracle:thin:@hostname:1521:SID

// For Service Name
jdbc:oracle:thin:@//hostname:1521/servicename
```

## Security Best Practices

1. **Never hardcode credentials** — Always use environment variables:
   ```bash
   export DB_PASSWORD="your_password"
   ```

2. **Use read-only accounts** for query-only operations:
   ```sql
   CREATE USER 'readonly'@'%' IDENTIFIED BY 'password';
   GRANT SELECT ON database.* TO 'readonly'@'%';
   ```

3. **Network isolation** — Deploy server in same VPC as databases:
   ```yaml
   # docker-compose.yml
   services:
     mcp-db-server:
       networks:
         - internal
     mysql:
       networks:
         - internal
   ```

4. **Connection pool limits** — Prevent resource exhaustion:
   ```java
   config.setMaximumPoolSize(10);  // Per unique connection
   ```

## Development & Extension

### Adding a New Database Type

1. **Update `ConnectionFactory.java`:**
```java
private static final Map<String, Integer> DEFAULT_PORTS = Map.of(
    // ... existing entries
    "mongodb", 27017  // Add new type
);

private static final Map<String, String> DRIVER_CLASSES = Map.of(
    // ... existing entries
    "mongodb", "com.mongodb.jdbc.MongoDriver"
);

private String buildJdbcUrl(String type, String host, int port, String database) {
    return switch (type) {
        // ... existing cases
        case "mongodb" -> String.format("jdbc:mongodb://%s:%d/%s", host, port, database);
        default -> throw new IllegalArgumentException("Unsupported type: " + type);
    };
}
```

2. **Add driver dependency:**
```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-jdbc</artifactId>
    <version>2.0.0</version>
</dependency>
```

3. **Test with new database type:**
```bash
./mvnw test -Dtest=DatabaseToolsTest#testMongoDBConnection
```

---

**Project:** [PerfectXM/mcp-db-server](https://github.com/PerfectXM/mcp-db-server)  
**License:** MIT  
**Java Version:** 17+  
**Spring Boot:** 3.4.13  
**MCP Version:** 1.1.7
