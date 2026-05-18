---
name: remote-mcp-server-with-auth
description: Build and deploy authenticated remote MCP servers with GitHub OAuth on Cloudflare Workers, featuring PostgreSQL integration and dual transport protocols
triggers:
  - create a remote MCP server with authentication
  - build an authenticated MCP server on Cloudflare
  - deploy MCP server with GitHub OAuth
  - set up remote MCP with database access
  - implement role-based MCP tools
  - create MCP server with SSE and HTTP transport
  - build database MCP server template
  - configure OAuth for MCP server
---

# remote-mcp-server-with-auth

> Skill by [ara.so](https://ara.so) — MCP Skills collection

## What This Project Does

`remote-mcp-server-with-auth` is a production-ready template for building remote Model Context Protocol (MCP) servers with GitHub OAuth authentication, deployable on Cloudflare Workers. It demonstrates best practices for MCP server architecture including modular tools, role-based access control, dual transport protocols (streamable HTTP and SSE), and PostgreSQL database integration with SQL injection protection.

## Key Capabilities

- **Modular Tool Architecture**: Separate files for each tool in `src/tools/`
- **Dual Transport Support**: `/mcp` (streamable HTTP, recommended) and `/sse` (SSE, legacy)
- **GitHub OAuth Authentication**: Built-in user authentication with role-based permissions
- **Database Lifespan Management**: PostgreSQL connection pooling across tool calls
- **Security**: SQL injection protection, query validation, username-based write access
- **Monitoring**: Optional Sentry integration for production observability

## Installation

```bash
# Clone the repository
git clone <repository-url>
cd remote-mcp-server-with-auth

# Install dependencies
npm install

# Install Wrangler CLI globally
npm install -g wrangler

# Authenticate with Cloudflare
wrangler login

# Create environment variables file
cp .dev.vars.example .dev.vars
```

## Configuration

### Environment Variables (`.dev.vars`)

```bash
# GitHub OAuth
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret
COOKIE_ENCRYPTION_KEY=your_random_encryption_key

# Database Connection
DATABASE_URL=postgresql://username:password@host:5432/database_name

# Optional: Sentry monitoring
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id
NODE_ENV=development
```

### Generate Encryption Key

```bash
openssl rand -hex 32
```

### GitHub OAuth App Setup

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Create new OAuth App:
   - **Homepage URL**: `http://localhost:8792` (local) or `https://your-worker.workers.dev` (production)
   - **Callback URL**: `http://localhost:8792/callback` or `https://your-worker.workers.dev/callback`
3. Copy Client ID and Client Secret to `.dev.vars`

### Role-Based Access Control

Edit `src/index.ts` to control database write access:

```typescript
const ALLOWED_USERNAMES = new Set([
  'your-github-username',
  'teammate-username',
  'admin-username'
]);
```

## Local Development

```bash
# Run the full database MCP server
wrangler dev

# Run the simple math example
wrangler dev --config wrangler-simple.jsonc

# Server runs at http://localhost:8792
```

### Test with MCP Inspector

```bash
npx @modelcontextprotocol/inspector@latest
```

Connect to:
- `http://localhost:8792/mcp` (recommended)
- `http://localhost:8792/sse` (legacy)

## Production Deployment

### Create KV Namespace

```bash
wrangler kv namespace create "OAUTH_KV"
```

Update `wrangler.jsonc` with the KV ID:

```json
{
  "kv_namespaces": [
    {
      "binding": "OAUTH_KV",
      "id": "your-kv-id-here"
    }
  ]
}
```

### Deploy to Cloudflare

```bash
wrangler deploy
```

### Set Production Secrets

```bash
wrangler secret put GITHUB_CLIENT_ID
wrangler secret put GITHUB_CLIENT_SECRET
wrangler secret put COOKIE_ENCRYPTION_KEY
wrangler secret put DATABASE_URL
wrangler secret put SENTRY_DSN  # optional
```

## Creating Custom MCP Tools

### Tool File Structure

Create a new tool in `src/tools/your-tool.ts`:

```typescript
import { z } from 'zod';
import { McpServer } from '../types.js';

const YourToolInputSchema = z.object({
  param1: z.string().describe('Description of parameter'),
  param2: z.number().optional().describe('Optional parameter'),
});

export function registerYourTool(server: McpServer) {
  server.tool(
    'yourTool',
    'Single-purpose description of what this tool does',
    YourToolInputSchema,
    async ({ param1, param2 }, { lifespan }) => {
      // Access shared resources from lifespan
      const db = lifespan.get('db');
      
      // Implement tool logic
      const result = await db.query('SELECT * FROM table WHERE id = $1', [param1]);
      
      return {
        content: [
          {
            type: 'text',
            text: JSON.stringify(result.rows, null, 2),
          },
        ],
      };
    }
  );
}
```

### Register the Tool

Add to `src/tools/index.ts`:

```typescript
import { registerYourTool } from './your-tool.js';

export function registerAllTools(server: McpServer) {
  registerListTables(server);
  registerQueryDatabase(server);
  registerExecuteDatabase(server);
  registerYourTool(server);  // Add your tool
}
```

## Database Tool Examples

### List Tables Tool

```typescript
// Read database schema (all authenticated users)
const ListTablesInputSchema = z.object({});

export function registerListTables(server: McpServer) {
  server.tool(
    'listTables',
    'List all tables in the database with their columns and constraints',
    ListTablesInputSchema,
    async (_, { lifespan }) => {
      const db = lifespan.get('db');
      const result = await db.query(`
        SELECT 
          table_name,
          column_name,
          data_type,
          is_nullable
        FROM information_schema.columns
        WHERE table_schema = 'public'
        ORDER BY table_name, ordinal_position
      `);
      
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(result.rows, null, 2),
        }],
      };
    }
  );
}
```

### Query Database Tool (Read-Only)

```typescript
const QueryDatabaseInputSchema = z.object({
  query: z.string().describe('SQL SELECT query to execute'),
});

export function registerQueryDatabase(server: McpServer) {
  server.tool(
    'queryDatabase',
    'Execute read-only SQL queries (SELECT statements only)',
    QueryDatabaseInputSchema,
    async ({ query }, { lifespan }) => {
      const db = lifespan.get('db');
      
      // Validate read-only operation
      const isReadOnly = /^\s*(SELECT|SHOW|DESCRIBE|EXPLAIN)\b/i.test(query);
      if (!isReadOnly) {
        throw new Error('Only read-only queries are allowed');
      }
      
      const result = await db.query(query);
      
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(result.rows, null, 2),
        }],
      };
    }
  );
}
```

### Execute Database Tool (Write Operations)

```typescript
const ExecuteDatabaseInputSchema = z.object({
  query: z.string().describe('SQL query to execute (INSERT, UPDATE, DELETE, DDL)'),
});

export function registerExecuteDatabase(server: McpServer) {
  server.tool(
    'executeDatabase',
    'Execute write operations (INSERT, UPDATE, DELETE, DDL) - requires privileged access',
    ExecuteDatabaseInputSchema,
    async ({ query }, { lifespan, meta }) => {
      const db = lifespan.get('db');
      const username = meta.username;
      
      // Check write permissions
      const ALLOWED_USERNAMES = new Set(['admin-username']);
      if (!username || !ALLOWED_USERNAMES.has(username)) {
        throw new Error('Insufficient permissions for write operations');
      }
      
      const result = await db.query(query);
      
      return {
        content: [{
          type: 'text',
          text: `Query executed successfully. Rows affected: ${result.rowCount}`,
        }],
      };
    }
  );
}
```

## Transport Protocol Implementation

### Streamable HTTP Transport (Recommended)

```typescript
app.post('/mcp', async (c) => {
  const username = c.get('username');
  
  const mcp = createMcpServer(
    server,
    {
      capabilities: {
        tools: {},
      },
    },
    { username }
  );
  
  return new StreamableHTTPServerTransport('/mcp', c.req.raw, mcp);
});
```

### SSE Transport (Legacy)

```typescript
app.post('/sse', async (c) => {
  const username = c.get('username');
  
  const mcp = createMcpServer(
    server,
    {
      capabilities: {
        tools: {},
      },
    },
    { username }
  );
  
  return new SSEServerTransport('/sse', c.req.raw, mcp);
});
```

## Lifespan Management

Share resources across tool calls using lifespan:

```typescript
const lifespan = createLifespan({
  db: async () => {
    const pool = new Pool({
      connectionString: c.env.DATABASE_URL,
      ssl: { rejectUnauthorized: false },
    });
    return {
      value: pool,
      dispose: () => pool.end(),
    };
  },
});
```

Access in tools:

```typescript
async ({ param }, { lifespan }) => {
  const db = lifespan.get('db');
  const result = await db.query('SELECT * FROM table');
  return result;
}
```

## Claude Desktop Integration

Edit Claude Desktop config (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "remote-db-server": {
      "url": "https://your-worker.workers.dev/mcp",
      "transport": "streamableHttp"
    }
  }
}
```

Or for SSE transport:

```json
{
  "mcpServers": {
    "remote-db-server": {
      "url": "https://your-worker.workers.dev/sse"
    }
  }
}
```

## Simple Math Example

The project includes a minimal example (`src/simple-math.ts`) demonstrating core concepts:

```typescript
import { z } from 'zod';

const CalculateInputSchema = z.object({
  operation: z.enum(['add', 'subtract', 'multiply', 'divide']),
  a: z.number(),
  b: z.number(),
});

server.tool(
  'calculate',
  'Perform basic arithmetic operations',
  CalculateInputSchema,
  async ({ operation, a, b }) => {
    let result: number;
    
    switch (operation) {
      case 'add': result = a + b; break;
      case 'subtract': result = a - b; break;
      case 'multiply': result = a * b; break;
      case 'divide':
        if (b === 0) throw new Error('Division by zero');
        result = a / b;
        break;
    }
    
    return {
      content: [{
        type: 'text',
        text: `Result: ${result}`,
      }],
    };
  }
);
```

## Security Best Practices

### SQL Injection Protection

Always use parameterized queries:

```typescript
// ✅ GOOD: Parameterized query
const result = await db.query(
  'SELECT * FROM users WHERE id = $1',
  [userId]
);

// ❌ BAD: String concatenation
const result = await db.query(
  `SELECT * FROM users WHERE id = ${userId}`
);
```

### Query Validation

Implement operation type detection:

```typescript
function isReadOnlyQuery(sql: string): boolean {
  const trimmed = sql.trim().toUpperCase();
  const readOnlyPatterns = /^(SELECT|SHOW|DESCRIBE|EXPLAIN)\b/;
  return readOnlyPatterns.test(trimmed);
}

function isWriteQuery(sql: string): boolean {
  const writePatterns = /^(INSERT|UPDATE|DELETE|CREATE|ALTER|DROP|TRUNCATE)\b/i;
  return writePatterns.test(sql.trim());
}
```

### User Context Validation

```typescript
async ({ query }, { meta }) => {
  const username = meta.username;
  
  if (!username) {
    throw new Error('Authentication required');
  }
  
  if (isWriteQuery(query) && !ALLOWED_USERNAMES.has(username)) {
    throw new Error('Insufficient permissions');
  }
}
```

## Troubleshooting

### OAuth Callback Issues

- Verify callback URL matches GitHub OAuth app configuration exactly
- Check `COOKIE_ENCRYPTION_KEY` is set and consistent
- Ensure KV namespace is created and bound correctly

### Database Connection Errors

```typescript
// Add connection timeout and retry logic
const pool = new Pool({
  connectionString: env.DATABASE_URL,
  ssl: { rejectUnauthorized: false },
  connectionTimeoutMillis: 5000,
  max: 10,
});
```

### Tool Not Appearing in Claude

- Verify tool is registered in `registerAllTools()`
- Check Zod schema validation doesn't have errors
- Restart Claude Desktop after config changes
- Test with MCP Inspector first

### Permission Denied Errors

- Confirm GitHub username is in `ALLOWED_USERNAMES` set
- Check user is properly authenticated (username available in meta)
- Verify OAuth flow completed successfully

### Deployment Issues

```bash
# Check deployment status
wrangler deployments list

# View logs
wrangler tail

# Test secrets are set
wrangler secret list
```

## Common Patterns

### Adding External API Tools

```typescript
export function registerWeatherTool(server: McpServer) {
  const schema = z.object({
    city: z.string().describe('City name'),
  });
  
  server.tool(
    'getWeather',
    'Get current weather for a city',
    schema,
    async ({ city }) => {
      const apiKey = process.env.WEATHER_API_KEY;
      const response = await fetch(
        `https://api.weather.com/v1/current?city=${city}&apiKey=${apiKey}`
      );
      const data = await response.json();
      
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(data, null, 2),
        }],
      };
    }
  );
}
```

### Tool with Complex Input Validation

```typescript
const ComplexInputSchema = z.object({
  filters: z.object({
    startDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
    endDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/),
    status: z.enum(['active', 'inactive', 'pending']).optional(),
  }),
  pagination: z.object({
    page: z.number().min(1).default(1),
    pageSize: z.number().min(1).max(100).default(20),
  }).optional(),
});
```

### Error Handling with Sentry

```typescript
import * as Sentry from '@sentry/cloudflare';

server.tool(
  'toolName',
  'Description',
  schema,
  async (input, { meta }) => {
    try {
      // Tool implementation
    } catch (error) {
      Sentry.captureException(error, {
        user: { username: meta.username },
        extra: { input },
      });
      throw error;
    }
  }
);
```
