---
name: affine-mcp-server-integration
description: Connect AI assistants to AFFiNE workspaces, documents, databases, and collaboration APIs using the Model Context Protocol.
triggers:
  - "connect to my AFFiNE workspace"
  - "set up AFFiNE MCP server"
  - "configure MCP for AFFiNE Cloud"
  - "create and manage AFFiNE documents with AI"
  - "integrate Claude with AFFiNE"
  - "access my AFFiNE knowledge base"
  - "use AFFiNE database with MCP"
  - "deploy AFFiNE MCP server"
---

# AFFiNE MCP Server Integration

> Skill by [ara.so](https://ara.so) — MCP Skills collection.

## Overview

AFFiNE MCP Server is a Model Context Protocol server that exposes AFFiNE workspaces, documents, databases, and collaboration features to AI assistants. It supports both AFFiNE Cloud and self-hosted deployments, offering 85+ tools across workspace management, document operations, database manipulation, and organizational features.

**Key capabilities:**
- Connect AI assistants (Claude Code, Cursor, Codex CLI, Claude Desktop) to AFFiNE
- Read, create, update, and delete documents programmatically
- Manage databases, collections, and organizational structures
- Access via stdio (local) or HTTP (remote) transports
- Semantic page composition and template instantiation
- Block-level document mutation and structured data handling

## Installation

### Global CLI Installation

```bash
# Install globally via npm
npm i -g affine-mcp-server

# Verify installation
affine-mcp --version
```

### Ad-hoc Execution

```bash
# Run without installing
npx -y -p affine-mcp-server affine-mcp -- --version
```

### Docker Deployment

```bash
# Pull the official image
docker pull ghcr.io/dawncr0w/affine-mcp-server:latest

# Run with environment variables
docker run -d \
  -p 3000:3000 \
  -e MCP_TRANSPORT=http \
  -e AFFINE_BASE_URL=https://app.affine.pro \
  -e AFFINE_API_TOKEN=${AFFINE_API_TOKEN} \
  -e AFFINE_MCP_AUTH_MODE=bearer \
  -e AFFINE_MCP_HTTP_TOKEN=${MCP_HTTP_TOKEN} \
  ghcr.io/dawncr0w/affine-mcp-server:latest
```

## Authentication Setup

### Interactive Login (Recommended for Local Use)

```bash
# Store credentials securely (~/.config/affine-mcp/config with mode 600)
affine-mcp login

# Interactive prompts will ask for:
# - AFFiNE base URL (default: https://app.affine.pro)
# - Authentication method (token, cookie, or email/password)
# - Credentials
```

### Environment Variables

```bash
# For AFFiNE Cloud (token required)
export AFFINE_BASE_URL=https://app.affine.pro
export AFFINE_API_TOKEN=ut_your_token_here

# For self-hosted with email/password
export AFFINE_BASE_URL=https://your-affine-instance.com
export AFFINE_EMAIL=user@example.com
export AFFINE_PASSWORD=${AFFINE_PASSWORD}

# For self-hosted with cookie
export AFFINE_BASE_URL=https://your-affine-instance.com
export AFFINE_COOKIE=${AFFINE_COOKIE}
```

### Getting an API Token

**AFFiNE Cloud:**
1. Sign in to https://app.affine.pro
2. Go to Settings → Integrations → MCP Server
3. Generate an API token

**Self-hosted:**
1. Sign in to your AFFiNE instance
2. Navigate to Settings → Account → Personal Access Tokens
3. Create a new token with appropriate scopes

## Client Configuration

### Claude Code

Add to your `.claude/project_config.json`:

```json
{
  "mcpServers": {
    "affine": {
      "command": "affine-mcp"
    }
  }
}
```

Or with explicit environment variables:

```json
{
  "mcpServers": {
    "affine": {
      "command": "affine-mcp",
      "env": {
        "AFFINE_BASE_URL": "https://app.affine.pro",
        "AFFINE_API_TOKEN": "${AFFINE_API_TOKEN}"
      }
    }
  }
}
```

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "affine": {
      "command": "affine-mcp"
    }
  }
}
```

### Cursor

Add to `~/.cursor/mcp_config.json`:

```json
{
  "mcpServers": {
    "affine": {
      "command": "affine-mcp"
    }
  }
}
```

### Codex CLI

```bash
# Add the server
codex mcp add affine -- affine-mcp

# Verify
codex mcp list
```

### HTTP Mode (Remote Deployment)

Client configuration:

```json
{
  "mcpServers": {
    "affine": {
      "type": "http",
      "url": "https://your-mcp-server.com/mcp",
      "headers": {
        "Authorization": "Bearer ${MCP_HTTP_TOKEN}"
      }
    }
  }
}
```

Server environment:

```bash
export MCP_TRANSPORT=http
export MCP_HTTP_PORT=3000
export AFFINE_MCP_AUTH_MODE=bearer
export AFFINE_MCP_HTTP_TOKEN=${MCP_HTTP_TOKEN}
```

## CLI Commands

### Configuration Management

```bash
# Show current configuration (secrets redacted)
affine-mcp show-config

# Get config file path
affine-mcp config-path

# Generate client configuration snippets
affine-mcp snippet claude
affine-mcp snippet cursor
affine-mcp snippet codex
affine-mcp snippet all

# Generate with environment variables
affine-mcp snippet claude --env

# Logout (remove stored credentials)
affine-mcp logout
```

### Health Checks

```bash
# Test effective configuration
affine-mcp status

# Machine-readable status
affine-mcp status --json

# Diagnose configuration and connectivity
affine-mcp doctor
```

### Running the Server

```bash
# Start stdio server (default)
affine-mcp

# Start HTTP server
MCP_TRANSPORT=http MCP_HTTP_PORT=3000 affine-mcp

# With custom log level
LOG_LEVEL=debug affine-mcp
```

## Tool Surface Overview

The server exposes 85 tools organized by domain:

### Workspace Tools

```typescript
// List all workspaces
list_workspaces()

// Get workspace details
get_workspace({ workspaceId: "workspace-id" })

// Create workspace
create_workspace({ 
  name: "My New Workspace",
  description: "Project workspace" 
})

// Update workspace
update_workspace({
  workspaceId: "workspace-id",
  name: "Updated Name"
})

// Delete workspace
delete_workspace({ workspaceId: "workspace-id" })
```

### Document Tools

```typescript
// Search documents
search_docs({
  workspaceId: "workspace-id",
  query: "meeting notes",
  limit: 10
})

// Get document by exact title
get_doc_by_title({
  workspaceId: "workspace-id",
  title: "Project Roadmap"
})

// Read document content
read_doc({
  workspaceId: "workspace-id",
  docId: "doc-id"
})

// Create document
create_doc({
  workspaceId: "workspace-id",
  title: "New Document",
  text: "Initial content",
  folderId: "folder-id" // Optional, new in v2.1.0
})

// Update document
update_doc({
  workspaceId: "workspace-id",
  docId: "doc-id",
  text: "Updated content"
})

// Delete document
delete_doc({
  workspaceId: "workspace-id",
  docId: "doc-id"
})

// Move document to trash
trash_doc({
  workspaceId: "workspace-id",
  docId: "doc-id"
})

// Restore from trash
restore_doc({
  workspaceId: "workspace-id",
  docId: "doc-id"
})
```

### Template Tools

```typescript
// List available templates
list_templates({ workspaceId: "workspace-id" })

// Inspect template structure
inspect_template({
  workspaceId: "workspace-id",
  templateId: "template-id"
})

// Create document from template
create_doc_from_template({
  workspaceId: "workspace-id",
  templateId: "template-id",
  title: "Q1 Report",
  folderId: "folder-id" // Optional
})
```

### Database Tools

```typescript
// Create database
create_database({
  workspaceId: "workspace-id",
  docId: "doc-id",
  blockId: "block-id", // Optional
  name: "Project Tracker"
})

// Add database column
add_database_column({
  workspaceId: "workspace-id",
  docId: "doc-id",
  databaseId: "database-id",
  name: "Status",
  type: "select",
  options: ["Todo", "In Progress", "Done"]
})

// Add database row
add_database_row({
  workspaceId: "workspace-id",
  docId: "doc-id",
  databaseId: "database-id",
  values: {
    "Task": "Implement feature",
    "Status": "In Progress"
  }
})

// Update database row
update_database_row({
  workspaceId: "workspace-id",
  docId: "doc-id",
  databaseId: "database-id",
  rowId: "row-id",
  values: {
    "Status": "Done"
  }
})

// Inspect database schema
inspect_database_schema({
  workspaceId: "workspace-id",
  docId: "doc-id",
  databaseId: "database-id"
})
```

### Collection Tools

```typescript
// List collections
list_collections({ workspaceId: "workspace-id" })

// Create collection
create_collection({
  workspaceId: "workspace-id",
  name: "Project Docs"
})

// Add document to collection
add_doc_to_collection({
  workspaceId: "workspace-id",
  collectionId: "collection-id",
  docId: "doc-id"
})

// Remove document from collection
remove_doc_from_collection({
  workspaceId: "workspace-id",
  collectionId: "collection-id",
  docId: "doc-id"
})
```

### Comment Tools

```typescript
// List comments on document
list_comments({
  workspaceId: "workspace-id",
  docId: "doc-id"
})

// Create comment
create_comment({
  workspaceId: "workspace-id",
  docId: "doc-id",
  text: "Great point!",
  quote: "original text" // Optional
})

// Update comment
update_comment({
  workspaceId: "workspace-id",
  docId: "doc-id",
  commentId: "comment-id",
  text: "Updated comment"
})

// Delete comment
delete_comment({
  workspaceId: "workspace-id",
  docId: "doc-id",
  commentId: "comment-id"
})
```

## Common Workflow Patterns

### Creating a Project Structure

```typescript
// 1. Create workspace
const workspace = await create_workspace({
  name: "Q1 Project",
  description: "Project tracking workspace"
});

// 2. Create main document
const mainDoc = await create_doc({
  workspaceId: workspace.id,
  title: "Project Overview",
  text: "# Project Overview\n\nKey objectives..."
});

// 3. Create database for task tracking
const database = await create_database({
  workspaceId: workspace.id,
  docId: mainDoc.id,
  name: "Task Tracker"
});

// 4. Add columns
await add_database_column({
  workspaceId: workspace.id,
  docId: mainDoc.id,
  databaseId: database.id,
  name: "Priority",
  type: "select",
  options: ["High", "Medium", "Low"]
});

// 5. Add initial tasks
await add_database_row({
  workspaceId: workspace.id,
  docId: mainDoc.id,
  databaseId: database.id,
  values: {
    "Task": "Define requirements",
    "Priority": "High"
  }
});
```

### Document Search and Update

```typescript
// 1. Search for documents
const results = await search_docs({
  workspaceId: "workspace-id",
  query: "meeting notes",
  limit: 5
});

// 2. Find specific document by exact title
const doc = await get_doc_by_title({
  workspaceId: "workspace-id",
  title: "Weekly Standup - 2024-01-15"
});

// 3. Read current content
const content = await read_doc({
  workspaceId: "workspace-id",
  docId: doc.id
});

// 4. Append new content
const updatedText = content.text + "\n\n## Action Items\n- Follow up on design";

await update_doc({
  workspaceId: "workspace-id",
  docId: doc.id,
  text: updatedText
});
```

### Using Templates

```typescript
// 1. List available templates
const templates = await list_templates({
  workspaceId: "workspace-id"
});

// 2. Inspect template to understand structure
const template = await inspect_template({
  workspaceId: "workspace-id",
  templateId: templates[0].id
});

// 3. Create document from template with placement
const newDoc = await create_doc_from_template({
  workspaceId: "workspace-id",
  templateId: templates[0].id,
  title: "Q1 Planning Doc",
  folderId: "planning-folder-id"
});
```

### Semantic Page Composition

```typescript
// Compose a structured document with intent
const pageIntent = {
  pageTitle: "Product Launch Plan",
  sections: [
    {
      heading: "Executive Summary",
      content: "Overview of product launch strategy..."
    },
    {
      heading: "Timeline",
      content: "Key milestones and dates",
      database: {
        name: "Launch Milestones",
        columns: [
          { name: "Milestone", type: "text" },
          { name: "Date", type: "date" },
          { name: "Owner", type: "text" }
        ]
      }
    }
  ]
};

// Use semantic composition tool
const result = await compose_semantic_page({
  workspaceId: "workspace-id",
  intent: JSON.stringify(pageIntent)
});
```

### Database Intent Composition

```typescript
// Create complex database structure from high-level intent
const dbIntent = {
  name: "Bug Tracker",
  columns: [
    { name: "Title", type: "text" },
    { name: "Severity", type: "select", options: ["Critical", "High", "Medium", "Low"] },
    { name: "Status", type: "select", options: ["Open", "In Progress", "Fixed", "Closed"] },
    { name: "Assigned To", type: "text" },
    { name: "Due Date", type: "date" }
  ],
  initialRows: [
    {
      "Title": "Login page crash",
      "Severity": "Critical",
      "Status": "Open"
    }
  ]
};

const result = await compose_database_from_intent({
  workspaceId: "workspace-id",
  docId: "doc-id",
  intent: JSON.stringify(dbIntent)
});
```

## Security Configuration

### Tool Profile Restriction

```bash
# Limit to read-only operations
export AFFINE_TOOL_PROFILE=read_only

# Core tools only (no experimental features)
export AFFINE_TOOL_PROFILE=core

# Authoring tools (read + write, no admin)
export AFFINE_TOOL_PROFILE=authoring

# Full access (default)
export AFFINE_TOOL_PROFILE=full
```

### Fine-Grained Tool Control

```bash
# Disable entire tool groups
export AFFINE_DISABLED_GROUPS=destructive,admin,docs.database

# Disable specific tools
export AFFINE_DISABLED_TOOLS=delete_workspace,delete_doc,trash_doc
```

### HTTP Server Security

```bash
# Enable bearer token authentication
export AFFINE_MCP_AUTH_MODE=bearer
export AFFINE_MCP_HTTP_TOKEN=${MCP_HTTP_TOKEN}

# Use HTTPS in production
export MCP_HTTP_PORT=3000
# Then reverse proxy with nginx/caddy for TLS
```

## Docker Deployment

### Basic Docker Run

```bash
docker run -d \
  --name affine-mcp \
  -p 3000:3000 \
  -e MCP_TRANSPORT=http \
  -e AFFINE_BASE_URL=${AFFINE_BASE_URL} \
  -e AFFINE_API_TOKEN=${AFFINE_API_TOKEN} \
  -e AFFINE_MCP_AUTH_MODE=bearer \
  -e AFFINE_MCP_HTTP_TOKEN=${MCP_HTTP_TOKEN} \
  ghcr.io/dawncr0w/affine-mcp-server:latest
```

### Docker Compose

```yaml
version: '3.8'

services:
  affine-mcp:
    image: ghcr.io/dawncr0w/affine-mcp-server:latest
    ports:
      - "3000:3000"
    environment:
      MCP_TRANSPORT: http
      MCP_HTTP_PORT: 3000
      AFFINE_BASE_URL: ${AFFINE_BASE_URL}
      AFFINE_API_TOKEN: ${AFFINE_API_TOKEN}
      AFFINE_MCP_AUTH_MODE: bearer
      AFFINE_MCP_HTTP_TOKEN: ${MCP_HTTP_TOKEN}
      AFFINE_TOOL_PROFILE: authoring
      LOG_LEVEL: info
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/healthz"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped
```

### Health Endpoints

```bash
# Liveness check
curl http://localhost:3000/healthz

# Readiness check
curl http://localhost:3000/readyz
```

## Environment Variables Reference

### Required

| Variable | Description | Example |
|----------|-------------|---------|
| `AFFINE_BASE_URL` | AFFiNE instance URL | `https://app.affine.pro` |
| `AFFINE_API_TOKEN` | API token (recommended) | `ut_...` |

### Authentication (Choose One)

| Variable | Description |
|----------|-------------|
| `AFFINE_API_TOKEN` | API token (recommended) |
| `AFFINE_EMAIL` + `AFFINE_PASSWORD` | Email/password (self-hosted only) |
| `AFFINE_COOKIE` | Session cookie |

### Transport

| Variable | Default | Description |
|----------|---------|-------------|
| `MCP_TRANSPORT` | `stdio` | `stdio` or `http` |
| `MCP_HTTP_PORT` | `3000` | HTTP server port |

### Security

| Variable | Default | Description |
|----------|---------|-------------|
| `AFFINE_MCP_AUTH_MODE` | `none` | `bearer`, `oauth`, or `none` |
| `AFFINE_MCP_HTTP_TOKEN` | - | Bearer token for HTTP auth |
| `AFFINE_TOOL_PROFILE` | `full` | `read_only`, `core`, `authoring`, `full` |
| `AFFINE_DISABLED_GROUPS` | - | Comma-separated groups to disable |
| `AFFINE_DISABLED_TOOLS` | - | Comma-separated tools to disable |

### Logging

| Variable | Default | Description |
|----------|---------|-------------|
| `LOG_LEVEL` | `info` | `debug`, `info`, `warn`, `error` |

## Troubleshooting

### Connection Issues

```bash
# Diagnose configuration
affine-mcp doctor

# Check status with verbose output
affine-mcp status --json

# Verify config file exists and is readable
affine-mcp config-path
ls -la $(affine-mcp config-path)
```

### Common Errors

**"Authentication failed"**
- Verify API token is correct and not expired
- For AFFiNE Cloud, use API token (not email/password)
- Check token has appropriate scopes

**"Workspace not found"**
- Ensure workspace exists and is server-backed (not browser-local)
- Verify user has access to workspace
- Check workspace ID is correct

**"Tool not found"**
- Check `AFFINE_TOOL_PROFILE` and `AFFINE_DISABLED_GROUPS` settings
- Verify tool name matches manifest exactly
- Update to latest version: `npm i -g affine-mcp-server@latest`

**"Connection refused" (HTTP mode)**
- Verify server is running: `docker ps` or check process
- Check firewall rules allow port access
- Verify `MCP_HTTP_PORT` matches client configuration

### Debug Mode

```bash
# Enable debug logging
LOG_LEVEL=debug affine-mcp

# Or with environment variable
export LOG_LEVEL=debug
affine-mcp
```

### Verify Tool Availability

```bash
# List all available tools
affine-mcp status --json | jq '.tools'

# Check specific tool
affine-mcp status --json | jq '.tools[] | select(.name == "create_doc")'
```

## Advanced Patterns

### Capability and Fidelity Reporting

```typescript
// Check document capabilities before operations
const capabilities = await report_doc_capabilities({
  workspaceId: "workspace-id",
  docId: "doc-id"
});

// Returns: { canEdit, canComment, canShare, exportFormats, ... }

// Check content fidelity (what blocks/features are supported)
const fidelity = await report_content_fidelity({
  workspaceId: "workspace-id",
  docId: "doc-id"
});
```

### Block-Level Mutations

```typescript
// Append block to document
await append_block({
  workspaceId: "workspace-id",
  docId: "doc-id",
  blockType: "paragraph",
  content: "New paragraph text"
});

// Insert block at specific position
await insert_block({
  workspaceId: "workspace-id",
  docId: "doc-id",
  afterBlockId: "block-id",
  blockType: "heading",
  content: "New Section",
  level: 2
});
```

### Workspace Blueprints

```typescript
// Create workspace from blueprint
const blueprint = {
  name: "Team Sprint",
  folders: ["Planning", "Design", "Development"],
  templates: ["standup", "retrospective"],
  databases: ["task-tracker"]
};

const workspace = await create_workspace_from_blueprint({
  blueprint: JSON.stringify(blueprint)
});
```

### Collection Rules and Sync

```typescript
// Create collection with rules
await create_collection({
  workspaceId: "workspace-id",
  name: "Meeting Notes",
  rules: {
    tagFilter: ["meeting"],
    autoSync: true
  }
});

// Sync collection (apply rules)
await sync_collection_rules({
  workspaceId: "workspace-id",
  collectionId: "collection-id"
});
```

## Best Practices

1. **Use API tokens** - More secure and reliable than email/password
2. **Enable health checks** - Use `/healthz` and `/readyz` for containerized deployments
3. **Restrict tool profiles** - Use `AFFINE_TOOL_PROFILE=authoring` in production to prevent accidental deletions
4. **Rotate tokens regularly** - Generate new API tokens periodically
5. **Use HTTPS** - Always use TLS for remote deployments
6. **Handle errors gracefully** - Check tool responses for error fields
7. **Batch operations** - Use semantic composition and database intent tools for complex structures
8. **Verify before destructive ops** - Use read tools to confirm before delete/trash operations

## Resources

- **Documentation**: `/docs` directory in repository
  - `getting-started.md` - First-run setup
  - `client-setup.md` - Client-specific configuration
  - `configuration-and-deployment.md` - Advanced deployment
  - `tool-reference.md` - Complete tool catalog
  - `workflow-recipes.md` - End-to-end examples
  - `edgeless-canvas-cookbook.md` - Canvas layout helpers
- **Tool Manifest**: `tool-manifest.json` - Canonical tool definitions
- **GitHub**: https://github.com/dawncr0w/affine-mcp-server
- **Issues**: https://github.com/dawncr0w/affine-mcp-server/issues
- **AFFiNE Docs**: https://docs.affine.pro
