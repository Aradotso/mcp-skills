---
name: easy-notion-mcp-integration
description: Connect AI agents to Notion using markdown instead of raw JSON through MCP server with 42 tools for reading, writing, and managing pages and databases
triggers:
  - "connect to Notion through MCP"
  - "read and write Notion pages as markdown"
  - "set up Notion MCP server"
  - "add database entries to Notion"
  - "convert Notion blocks to markdown"
  - "configure Notion integration with OAuth"
  - "upload files to Notion pages"
  - "query Notion databases with filters"
---

# Easy Notion MCP Integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

`easy-notion-mcp` is a production-grade Model Context Protocol (MCP) server that bridges AI agents to Notion using GitHub-flavored markdown instead of raw Notion API JSON. It provides 42 MCP tools for reading and writing Notion content with round-trip fidelity across 24 block types including toggles, columns, callouts, tables, and file uploads.

### Key Features

- **Markdown-first I/O**: Agents work with markdown; server handles all Notion block JSON conversion
- **Database ergonomics**: Write simple property objects like `{ "Status": "Done" }` with automatic schema mapping
- **Dual transport**: Stdio (API token) or HTTP (OAuth/bearer token)
- **Optional Redis**: Shared schema cache and OAuth persistence for multi-instance deployments
- **Security defaults**: Content sanitization, URL validation, workspace-root file containment

## Installation

### Quick Start with npx (Recommended)

Configure in your MCP client (Claude Desktop, Cursor, etc.):

```json
{
  "mcpServers": {
    "notion": {
      "command": "npx",
      "args": ["-y", "easy-notion-mcp"],
      "env": {
        "NOTION_TOKEN": "YOUR_NOTION_INTEGRATION_TOKEN"
      }
    }
  }
}
```

### From Source

```bash
git clone https://github.com/Grey-Iris/easy-notion-mcp.git
cd easy-notion-mcp
npm install
npm run build
```

MCP client configuration:

```json
{
  "mcpServers": {
    "notion": {
      "command": "node",
      "args": ["/absolute/path/to/easy-notion-mcp/dist/index.js"],
      "env": {
        "NOTION_TOKEN": "YOUR_NOTION_INTEGRATION_TOKEN",
        "NOTION_ROOT_PAGE_ID": "optional_default_parent_page_id"
      }
    }
  }
}
```

### Getting a Notion Integration Token

1. Visit https://www.notion.so/my-integrations
2. Click **+ New integration**
3. Name your integration and select capabilities
4. Copy the **Internal Integration Token** (starts with `ntn_`)
5. Share target pages/databases with your integration

## Configuration

### Environment Variables

**Core Settings:**

```bash
# Required for stdio transport
NOTION_TOKEN=ntn_your_integration_token_here

# Optional: default parent for create_page
NOTION_ROOT_PAGE_ID=abc123def456

# Disable content-notice prefix on reads (default: false)
NOTION_TRUST_CONTENT=false

# Root directory for file:// uploads (default: cwd)
NOTION_MCP_WORKSPACE_ROOT=/path/to/workspace

# Logging verbosity: debug, info, warn, error (default: info)
LOG_LEVEL=info
```

**HTTP Transport (OAuth or Static Bearer):**

```bash
# Static bearer token for /mcp endpoint
NOTION_MCP_BEARER=your_random_secret_token

# OAuth configuration
NOTION_OAUTH_CLIENT_ID=your_oauth_client_id
NOTION_OAUTH_CLIENT_SECRET=your_oauth_client_secret

# HTTP server settings
PORT=3333
NOTION_MCP_BIND_HOST=127.0.0.1
```

**Redis (Optional Multi-Instance Persistence):**

```bash
# Enable Redis (auto-enabled if REDIS_URL is set)
REDIS_ENABLED=true
REDIS_URL=redis://127.0.0.1:6379/0

# Or configure individually:
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=your_redis_password
REDIS_DB=0
REDIS_KEY_PREFIX=easy-notion:
REDIS_CONNECT_TIMEOUT_MS=10000
REDIS_MAX_RETRIES=3
```

### .env File Setup

```bash
cp .env.example .env
# Edit .env with your configuration
```

## MCP Tools Reference

### Page Operations

#### `read_page`
Read a Notion page as markdown.

```typescript
// Tool call
{
  "name": "read_page",
  "arguments": {
    "page_id": "abc123def456"
  }
}

// Response includes markdown with metadata
{
  "markdown": "# Page Title\n\nContent here...",
  "warnings": []
}
```

#### `create_page`
Create a new page with markdown content.

```typescript
{
  "name": "create_page",
  "arguments": {
    "title": "My New Page",
    "markdown": "# Heading\n\nSome **bold** text.",
    "parent_page_id": "parent123" // optional, uses NOTION_ROOT_PAGE_ID if not set
  }
}

// Response
{
  "id": "new_page_id",
  "url": "https://notion.so/...",
  "title": "My New Page"
}
```

#### `update_page`
Update page content with markdown.

```typescript
{
  "name": "update_page",
  "arguments": {
    "page_id": "abc123",
    "markdown": "# Updated Content\n\nNew paragraph.",
    "mode": "replace" // or "append"
  }
}
```

#### `append_to_page`
Append markdown to existing page.

```typescript
{
  "name": "append_to_page",
  "arguments": {
    "page_id": "abc123",
    "markdown": "\n## New Section\n\nAppended content."
  }
}
```

#### `delete_page`
Move page to trash.

```typescript
{
  "name": "delete_page",
  "arguments": {
    "page_id": "abc123"
  }
}
```

### Database Operations

#### `query_database`
Query database with filters and sorts.

```typescript
{
  "name": "query_database",
  "arguments": {
    "database_id": "db123",
    "filter": {
      "property": "Status",
      "select": { "equals": "In Progress" }
    },
    "sorts": [
      { "property": "Created", "direction": "descending" }
    ],
    "page_size": 50
  }
}

// Returns array of entries with markdown content
{
  "results": [
    {
      "id": "entry123",
      "properties": { "Status": "In Progress", "Name": "Task 1" },
      "markdown": "Task content..."
    }
  ],
  "has_more": false
}
```

#### `get_database`
Get database schema and properties.

```typescript
{
  "name": "get_database",
  "arguments": {
    "database_id": "db123"
  }
}

// Returns schema information
{
  "id": "db123",
  "title": "Tasks",
  "properties": {
    "Name": { "type": "title" },
    "Status": { 
      "type": "select",
      "options": ["To Do", "In Progress", "Done"]
    },
    "Due Date": { "type": "date" }
  }
}
```

#### `add_database_entry`
Create new database entry with automatic schema conversion.

```typescript
{
  "name": "add_database_entry",
  "arguments": {
    "database_id": "db123",
    "properties": {
      "Name": "New Task",
      "Status": "To Do",
      "Due Date": "2026-12-31",
      "Tags": ["urgent", "bug"]
    },
    "markdown": "## Task Details\n\nDetailed description here."
  }
}

// Response
{
  "id": "new_entry_id",
  "url": "https://notion.so/...",
  "properties": { /* converted properties */ }
}
```

#### `update_database_entry`
Update database entry properties and/or content.

```typescript
{
  "name": "update_database_entry",
  "arguments": {
    "page_id": "entry123",
    "properties": {
      "Status": "Done",
      "Completed": true
    },
    "markdown": "Updated task description."
  }
}
```

### Search Operations

#### `search_notion`
Full-text search across workspace.

```typescript
{
  "name": "search_notion",
  "arguments": {
    "query": "project planning",
    "filter": { "property": "object", "value": "page" },
    "sort": { "direction": "descending", "timestamp": "last_edited_time" },
    "page_size": 20
  }
}

// Returns matching pages/databases
{
  "results": [
    {
      "id": "page123",
      "title": "Q4 Project Planning",
      "url": "https://notion.so/...",
      "last_edited_time": "2026-07-03T10:30:00Z"
    }
  ],
  "has_more": false
}
```

### File Operations

#### `upload_file`
Upload local file to a page (stdio transport only).

```typescript
{
  "name": "upload_file",
  "arguments": {
    "page_id": "abc123",
    "file_path": "/workspace/documents/report.pdf",
    "caption": "Q3 Financial Report"
  }
}

// Security: file_path must be within NOTION_MCP_WORKSPACE_ROOT
```

#### `add_image`
Add image from URL to page.

```typescript
{
  "name": "add_image",
  "arguments": {
    "page_id": "abc123",
    "url": "https://example.com/image.png",
    "caption": "Architecture Diagram"
  }
}
```

### Block Operations

#### `create_block`
Create specific block types.

```typescript
// Callout block
{
  "name": "create_block",
  "arguments": {
    "parent_id": "page123",
    "type": "callout",
    "content": {
      "icon": "💡",
      "text": "Important note here",
      "color": "blue_background"
    }
  }
}

// Toggle block
{
  "name": "create_block",
  "arguments": {
    "parent_id": "page123",
    "type": "toggle",
    "content": {
      "text": "Click to expand",
      "children": [
        { "type": "paragraph", "text": "Hidden content" }
      ]
    }
  }
}
```

## Common Patterns

### Reading and Updating a Page

```typescript
// 1. Read existing page
const readResponse = await callTool("read_page", {
  page_id: "abc123def456"
});

// 2. Process markdown
const existingMarkdown = readResponse.markdown;
const updatedMarkdown = existingMarkdown + "\n## New Section\n\nAdded content.";

// 3. Update page
await callTool("update_page", {
  page_id: "abc123def456",
  markdown: updatedMarkdown,
  mode: "replace"
});
```

### Creating a Task in Database

```typescript
// 1. Get database schema (optional but recommended for validation)
const schema = await callTool("get_database", {
  database_id: "tasks_db_id"
});

// 2. Add entry with automatic schema conversion
const newTask = await callTool("add_database_entry", {
  database_id: "tasks_db_id",
  properties: {
    "Task Name": "Implement feature X",
    "Status": "In Progress",
    "Priority": "High",
    "Assignee": "john@example.com",
    "Due Date": "2026-08-15",
    "Tags": ["backend", "api"]
  },
  markdown: `## Description

Implement the new feature X as specified in RFC-123.

### Acceptance Criteria

- [ ] API endpoints created
- [ ] Tests written
- [ ] Documentation updated`
});

console.log(`Created task: ${newTask.url}`);
```

### Querying and Processing Database Entries

```typescript
// Query active tasks
const results = await callTool("query_database", {
  database_id: "tasks_db_id",
  filter: {
    "and": [
      {
        "property": "Status",
        "select": { "does_not_equal": "Done" }
      },
      {
        "property": "Priority",
        "select": { "equals": "High" }
      }
    ]
  },
  sorts: [
    { "property": "Due Date", "direction": "ascending" }
  ]
});

// Process each task
for (const entry of results.results) {
  console.log(`Task: ${entry.properties["Task Name"]}`);
  console.log(`Due: ${entry.properties["Due Date"]}`);
  console.log(`Content: ${entry.markdown}`);
  
  // Update if needed
  if (shouldUpdate(entry)) {
    await callTool("update_database_entry", {
      page_id: entry.id,
      properties: {
        "Status": "In Review"
      }
    });
  }
}
```

### Creating a Page with Rich Content

```typescript
const richContent = `# Project Documentation

## Overview

This project implements **critical features** for Q4 delivery.

## Architecture

![System Diagram](https://example.com/diagram.png)

## Tasks

- [x] Initial setup
- [ ] Core implementation
- [ ] Testing

## Notes

> 💡 Remember to update the API docs after deployment.

\`\`\`typescript
// Example usage
const client = new NotionClient({ token: process.env.NOTION_TOKEN });
const page = await client.getPage("page_id");
\`\`\`

## Next Steps

1. Review code
2. Deploy to staging
3. Monitor metrics
`;

const page = await callTool("create_page", {
  title: "Q4 Project Documentation",
  markdown: richContent,
  parent_page_id: process.env.NOTION_ROOT_PAGE_ID
});
```

### Searching and Organizing Content

```typescript
// Search for all planning documents
const searchResults = await callTool("search_notion", {
  query: "planning",
  filter: { "property": "object", "value": "page" },
  sort: { 
    "direction": "descending", 
    "timestamp": "last_edited_time" 
  }
});

// Create index page with links
const indexMarkdown = searchResults.results
  .map(r => `- [${r.title}](${r.url})`)
  .join("\n");

await callTool("create_page", {
  title: "Planning Documents Index",
  markdown: `# Planning Documents\n\n${indexMarkdown}`,
  parent_page_id: process.env.NOTION_ROOT_PAGE_ID
});
```

## CLI Usage

The project includes a CLI for low-context Notion access:

```bash
# Readonly mode
npx easy-notion-mcp --profile readonly

# Readwrite mode
npx easy-notion-mcp --profile readwrite

# With custom config
NOTION_TOKEN=ntn_xxx npx easy-notion-mcp --profile readwrite
```

## Development Workflow

### Local Development

```bash
# Install dependencies
npm install

# Watch mode for development
npm run dev

# Run stdio server locally
NOTION_TOKEN=ntn_xxx npm start

# Run HTTP server
NOTION_TOKEN=ntn_xxx NOTION_MCP_BEARER=$(openssl rand -hex 32) npm run start:http
```

### Testing

```bash
# Run all tests
npm test

# Watch mode
npm run test:watch

# Type checking
npm run typecheck

# Linting
npm run lint

# Build
npm run build
```

### Integration Tests

```bash
# Requires dedicated test workspace
cp .env.example .env
# Set E2E_ROOT_PAGE_ID to a test page
npm run test:e2e
```

## HTTP Transport Setup

### Static Bearer Token

```bash
# Generate secure token
export NOTION_MCP_BEARER=$(openssl rand -hex 32)

# Start server
NOTION_TOKEN=ntn_xxx NOTION_MCP_BEARER=$NOTION_MCP_BEARER npm run start:http
```

Configure HTTP client:

```json
{
  "mcpServers": {
    "notion-http": {
      "url": "http://127.0.0.1:3333/mcp",
      "transport": "http",
      "headers": {
        "Authorization": "Bearer YOUR_BEARER_TOKEN"
      }
    }
  }
}
```

### OAuth Flow

```bash
# Configure OAuth application in Notion
NOTION_OAUTH_CLIENT_ID=your_client_id \
NOTION_OAUTH_CLIENT_SECRET=your_client_secret \
npm run start:http

# Navigate to http://localhost:3333/auth/login
# Complete OAuth flow
# Tokens stored in Redis or encrypted files
```

## Troubleshooting

### Common Issues

**`NOTION_TOKEN is required` error:**
```bash
# Ensure token is set in MCP config env block
{
  "env": {
    "NOTION_TOKEN": "ntn_your_token"
  }
}
```

**`401 invalid_token` (HTTP mode):**
```bash
# Verify bearer token matches on client and server
# Server: NOTION_MCP_BEARER=secret_token
# Client: Authorization: Bearer secret_token
```

**`outside the allowed workspace root` error:**
```bash
# Set workspace root or use absolute paths inside it
NOTION_MCP_WORKSPACE_ROOT=/path/to/workspace

# Or use URLs instead of file:// paths
```

**Redis connection warnings:**
```bash
# Start Redis or disable
redis-server

# Or explicitly disable
REDIS_ENABLED=false npm start
```

**Unknown property name errors:**
```bash
# Call get_database first to fetch current schema
# Property names are case-sensitive
# Example: "Status" not "status"
```

**Stale schema cache:**
```bash
# Clear Redis cache
redis-cli KEYS "easy-notion:schema:*" | xargs redis-cli DEL

# Or restart with fresh in-memory cache
```

### Enable Debug Logging

```bash
LOG_LEVEL=debug npm start
```

Debug logs include:
- Full Notion API request/response bodies
- Schema cache hits/misses
- Block conversion warnings
- Redis operation details

### Testing Connectivity

```typescript
// Simple page read test
const result = await callTool("read_page", {
  page_id: "known_shared_page_id"
});
console.log(result.markdown);

// Search test
const search = await callTool("search_notion", {
  query: "test",
  page_size: 1
});
console.log(search.results);
```

## Security Considerations

### Content Sanitization

By default, all read operations include a content-notice prefix warning agents about potentially untrusted content:

```bash
# Disable for trusted workspaces only
NOTION_TRUST_CONTENT=true
```

### File Upload Restrictions

- File uploads via `file://` URLs are **stdio transport only**
- All paths must be within `NOTION_MCP_WORKSPACE_ROOT`
- Symlinks are validated to prevent escape
- Use HTTPS URLs for HTTP transport

### Workspace Isolation

```bash
# Restrict file operations to specific directory
NOTION_MCP_WORKSPACE_ROOT=/workspace/notion-files

# Attempts to access /etc/passwd will fail
```

### Bearer Token Security

```bash
# Generate cryptographically secure tokens
openssl rand -hex 32

# Store securely, never commit to version control
# Use environment variables or secret management
```

## Advanced Configuration

### Redis for Multi-Instance Deployments

```bash
# Shared schema cache across instances
REDIS_ENABLED=true
REDIS_URL=redis://redis.internal:6379/0
REDIS_KEY_PREFIX=prod-notion:

# Start multiple HTTP instances
for i in {1..3}; do
  PORT=$((3333 + i)) npm run start:http &
done
```

### Custom Cache TTL

Schema cache TTL is not directly configurable but can be adjusted in code:

```typescript
// src/persistence/cache-store.ts
const DEFAULT_TTL = 3600; // 1 hour in seconds
```

### Property Type Mapping

The server automatically converts between markdown-friendly values and Notion property types:

- **Title/Rich Text**: Plain string → Notion rich text
- **Number**: Numeric string → Number
- **Date**: ISO string → Date object
- **Select/Multi-select**: String/Array → Option references
- **People**: Email string/Array → Person references
- **Checkbox**: Boolean → Boolean
- **URL**: String → URL (validated)
- **Email**: String → Email (validated)
- **Phone**: String → Phone number
- **Files**: URL string/Array → External file references

## Resources

- **Official Notion API**: https://developers.notion.com/
- **MCP Protocol Spec**: https://modelcontextprotocol.io/
- **Project Repository**: https://github.com/Grey-Iris/easy-notion-mcp
- **Issue Tracker**: https://github.com/Grey-Iris/easy-notion-mcp/issues

## License

MIT License - See project LICENSE file for details.
